---
layout: post
title: etcd TLS Certificates with OpenSSL
date: 2026-05-01
description: Building a private PKI for etcd by hand with openssl — CA, server, peer, and client certs, and why each role needs the EKUs and SANs it does.
tags: [tls, openssl, pki, etcd, security]
---

## Summary

etcd's mutual TLS is nothing more than a small private PKI laid over etcd's two
traffic planes — the client API on port 2379 and peer replication on 2380. One
CA signs four kinds of leaf cert, and each kind is distinguished only by its
extended key usage (EKU) and whether it carries SANs. Once you can see the
topology and the CA→leaf trust model, the EKU/SAN choices — and the `openssl`
incantations that produce them — stop being magic.

## Key concepts

### The trust model

A CA is just a keypair whose public half everyone agrees to trust. It signs leaf
certificates; a party validates a leaf by checking that its signature chains back
to a CA already trusted. Nobody trusts anybody *directly* — they trust the CA
and, transitively, whatever it signed. This is the same design the public web
runs on (browsers ship a bundle of root CAs); here you run your own root for a
closed system, so the trust set is exactly the parties you issue to.

Mutual TLS (mTLS) means both ends of a connection present a cert and each
validates the other against the CA. Ordinary HTTPS is one-sided — only the server
proves who it is. etcd runs mTLS on both planes, which is the only reason client
certs exist at all.

### etcd's two communication planes

The cert roles only make sense once you see who dials whom. etcd listens on two
ports, carrying two completely separate conversations:

```
   client plane (TCP 2379)              peer plane (TCP 2380)
   ───────────────────────              ─────────────────────
   kube-apiserver ─┐                       ┌────────┐
   etcdctl ────────┤──►  etcd-1  ◄───────► │ etcd-2 │
   backup tool ────┘                ◄────► │ etcd-3 │
                                           └────────┘
   caller presents clientAuth      each member presents serverAuth + clientAuth
   etcd   presents serverAuth      (it both dials peers and accepts from them)
```

- **Client plane — TCP 2379.** API consumers (kube-apiserver, `etcdctl`, backup
  tooling) dial etcd's key-value API. etcd presents the **server** cert; the
  caller presents a **client** cert. `--client-cert-auth` is what makes etcd
  *demand and verify* the caller's cert rather than accept anonymous TLS.
- **Peer plane — TCP 2380.** Cluster members replicate the Raft log to each
  other. Here every member both *initiates* connections to other members and
  *accepts* connections from them — so on this plane a single node is
  simultaneously a server and a client. That dual role is the entire reason the
  **peer** cert needs two EKUs. `--peer-client-cert-auth` enforces mutual
  verification between members.

Everything below follows from this picture.

### Four roles = EKU + SANs, nothing more

A leaf's *role* isn't a property of the key or the CA — it's carried in two X.509
extensions:

- **Extended Key Usage (EKU)** declares which side of a handshake the cert may
  play. `serverAuth` = "you may connect to me on my listening port."
  `clientAuth` = "I may authenticate as the party that dialed out." Those are the
  two directions of a TLS connection.
- **Subject Alternative Names (SANs)** list the addresses a cert is valid *for*
  when something reaches it by name or IP.

Map each role onto the planes above:

| Role   | Where it lives              | EKU                       | SANs     |
| ------ | --------------------------- | ------------------------- | -------- |
| CA     | signs the other three       | n/a                       | n/a      |
| Server | etcd, client plane (2379)   | `serverAuth`              | required |
| Peer   | etcd, peer plane (2380)     | `serverAuth + clientAuth` | required |
| Client | API consumers → 2379        | `clientAuth`              | optional |

The peer cert is the only non-obvious one, and now it isn't: a peer is dialed by
other peers (needs `serverAuth`) *and* dials them (needs `clientAuth`), so it
carries both.

### Why SANs matter for server/peer but not client

SAN checking is asymmetric: only the side being *dialed* gets its hostname
verified. When kube-apiserver connects to `192.168.1.10:2379`, it checks that
`192.168.1.10` appears in the server cert's SANs — that's how it knows it reached
the real etcd and not a MITM. Peers do the same to each other on 2380. So both
listeners must carry SANs covering every address they're reached at.

The client never listens; it initiates. Nothing dials *it*, so it has no hostname
to verify and needs no SANs. It's identified instead by its Subject CN, which
etcd maps to an RBAC identity. That's why a client cert's CN is load-bearing and
a server cert's mostly isn't.

### The OpenSSL extension asymmetry (the real footgun)

The most confusing thing about hand-rolling certs isn't the crypto — it's that
`openssl` reads extensions from a *different place* depending on the operation:

| Operation                    | Flag(s) needed                           | Section read            |
| ---------------------------- | ---------------------------------------- | ----------------------- |
| `req -x509` (self-signed CA) | `-config ca.ext`                         | `[req] x509_extensions` |
| `req -new` (CSR creation)    | `-config <name>.ext`                     | `[req] req_extensions`  |
| `x509 -req` (CA signs a CSR) | `-extfile <name>.ext -extensions v3_req` | `[v3_req]`              |

The sharp edge: **`x509 -req` ignores the extensions embedded in the CSR by
default.** The SANs and EKUs you put in a CSR are only a *request* — the signing
step discards them unless you re-supply them with `-extfile`/`-extensions`. This
is both a footgun (forget it and your cert silently ships with no SANs) and a
feature (the CA has the final say on what it will vouch for, not the requester).
Every leaf below re-supplies its extensions at signing time for exactly this
reason.

### The CA

`ca.ext`:

```ini
[req]
x509_extensions    = v3_ca
distinguished_name = req_distinguished_name
prompt             = no

[req_distinguished_name]
O  = homelab
CN = etcd-ca

[v3_ca]
subjectKeyIdentifier   = hash
authorityKeyIdentifier = keyid:always, issuer
basicConstraints       = critical, CA:TRUE
keyUsage               = critical, keyCertSign, cRLSign
```

> `x509_extensions` (not `req_extensions`) is the right key when `openssl req` is
> invoked with `-x509` to produce a self-signed cert. `prompt = no` makes OpenSSL
> read DN fields directly from this section instead of prompting.

```bash
# Generate private key
openssl genpkey -algorithm RSA -pkeyopt rsa_keygen_bits:4096 -out ca.key
chmod 600 ca.key

# Inspect private key
openssl pkey -in ca.key -text -noout

# Extract and view public key
openssl pkey -in ca.key -pubout -out ca.pub
openssl pkey -pubin -in ca.pub -text -noout

# Self-signed CA cert (10 years)
openssl req -new -x509 -days 3650 -sha256 \
  -key ca.key \
  -config ca.ext \
  -out ca.crt

# View cert
openssl x509 -in ca.crt -text -noout
```

### Server — client plane (2379)

`serverAuth` only. Edit `[alt_names]` to match every IP/DNS name a client uses to
reach this node.

`server.ext`:

```ini
[req]
req_extensions     = v3_req
distinguished_name = req_distinguished_name
prompt             = no

[req_distinguished_name]
O  = homelab
CN = etcd-server

[v3_req]
basicConstraints       = CA:FALSE
subjectKeyIdentifier   = hash
authorityKeyIdentifier = keyid:always
keyUsage               = critical, keyEncipherment, digitalSignature
extendedKeyUsage       = serverAuth
subjectAltName         = @alt_names

[alt_names]
DNS.1 = localhost
DNS.2 = etcd-node-1
IP.1  = 127.0.0.1
IP.2  = 192.168.1.10
```

```bash
# Generate private key
openssl genpkey -algorithm RSA -pkeyopt rsa_keygen_bits:2048 -out server.key
chmod 600 server.key

# Inspect private key
openssl pkey -in server.key -text -noout

# View public key (inline, no file)
openssl pkey -in server.key -pubout | openssl pkey -pubin -text -noout

# Generate CSR
openssl req -new \
  -key server.key \
  -config server.ext \
  -out server.csr

# Inspect CSR — verify SANs are present
openssl req -in server.csr -text -noout

# Sign with CA
openssl x509 -req -days 365 -sha256 \
  -in server.csr \
  -CA ca.crt -CAkey ca.key -CAcreateserial \
  -extensions v3_req -extfile server.ext \
  -out server.crt

# View signed cert
openssl x509 -in server.crt -text -noout
```

### Peer — peer plane (2380)

Both `serverAuth` and `clientAuth`, because a member acts as both server and
client to its peers. SANs as for the server. Only the EKU line differs from
`server.ext`.

`peer.ext`:

```ini
[req]
req_extensions     = v3_req
distinguished_name = req_distinguished_name
prompt             = no

[req_distinguished_name]
O  = homelab
CN = etcd-peer

[v3_req]
basicConstraints       = CA:FALSE
subjectKeyIdentifier   = hash
authorityKeyIdentifier = keyid:always
keyUsage               = critical, keyEncipherment, digitalSignature
extendedKeyUsage       = serverAuth, clientAuth
subjectAltName         = @alt_names

[alt_names]
DNS.1 = localhost
DNS.2 = etcd-node-1
IP.1  = 127.0.0.1
IP.2  = 192.168.1.10
```

```bash
# Generate private key
openssl genpkey -algorithm RSA -pkeyopt rsa_keygen_bits:2048 -out peer.key
chmod 600 peer.key

# Inspect private key
openssl pkey -in peer.key -text -noout

# View public key (inline, no file)
openssl pkey -in peer.key -pubout | openssl pkey -pubin -text -noout

# Generate CSR
openssl req -new \
  -key peer.key \
  -config peer.ext \
  -out peer.csr

# Inspect CSR — verify SANs and dual EKU are present
openssl req -in peer.csr -text -noout

# Sign with CA
openssl x509 -req -days 365 -sha256 \
  -in peer.csr \
  -CA ca.crt -CAkey ca.key -CAcreateserial \
  -extensions v3_req -extfile peer.ext \
  -out peer.crt

# View signed cert
openssl x509 -in peer.crt -text -noout
```

### Client — client plane caller

`clientAuth` only, no SANs. The CN identifies the caller to etcd's RBAC (e.g.
kube-apiserver → etcd).

`client.ext`:

```ini
[req]
req_extensions     = v3_req
distinguished_name = req_distinguished_name
prompt             = no

[req_distinguished_name]
O  = homelab
CN = etcd-client

[v3_req]
basicConstraints       = CA:FALSE
subjectKeyIdentifier   = hash
authorityKeyIdentifier = keyid:always
keyUsage               = critical, keyEncipherment, digitalSignature
extendedKeyUsage       = clientAuth
```

```bash
# Generate private key
openssl genpkey -algorithm RSA -pkeyopt rsa_keygen_bits:2048 -out client.key
chmod 600 client.key

# Inspect private key
openssl pkey -in client.key -text -noout

# View public key (inline, no file)
openssl pkey -in client.key -pubout | openssl pkey -pubin -text -noout

# Generate CSR
openssl req -new \
  -key client.key \
  -config client.ext \
  -out client.csr

# Inspect CSR
openssl req -in client.csr -text -noout

# Sign with CA
openssl x509 -req -days 365 -sha256 \
  -in client.csr \
  -CA ca.crt -CAkey ca.key -CAcreateserial \
  -extensions v3_req -extfile client.ext \
  -out client.crt

# View signed cert
openssl x509 -in client.crt -text -noout
```

### Verification

```bash
# Verify each cert against the CA
openssl verify -CAfile ca.crt server.crt
openssl verify -CAfile ca.crt peer.crt
openssl verify -CAfile ca.crt client.crt

# Confirm cert and key match (the two fingerprints must be identical)
openssl x509 -in server.crt -pubkey -noout | openssl dgst -sha256
openssl pkey  -in server.key -pubout       | openssl dgst -sha256

# Check SANs, EKU, and key usage on a cert
openssl x509 -in server.crt -noout -text \
  | grep -E -A3 "Subject Alternative|Extended Key Usage|Key Usage|Basic Constraints"

# Confirm CA cert is actually marked as a CA
openssl x509 -in ca.crt -noout -text \
  | grep -E "CA:TRUE|Key Usage"

# End-to-end TLS handshake against a running etcd node
openssl s_client -connect 192.168.1.10:2379 \
  -CAfile ca.crt -cert client.crt -key client.key </dev/null
```

### Wiring it into etcd

The flags mirror the two planes exactly — one set per port, each pointing at the
CA plus that plane's cert/key, with `*-client-cert-auth` turning on mutual
verification:

```
# Client plane (2379)
--cert-file=server.crt
--key-file=server.key
--trusted-ca-file=ca.crt
--client-cert-auth

# Peer plane (2380)
--peer-cert-file=peer.crt
--peer-key-file=peer.key
--peer-trusted-ca-file=ca.crt
--peer-client-cert-auth
```

### Key usage reference

The full extension matrix across roles, if you want to sanity-check a cert
against what it *should* carry:

| Field              |          CA          |   server   |    peer    |   client   |
| ------------------ | :------------------: | :--------: | :--------: | :--------: |
| `basicConstraints` | `CA:TRUE` (critical) | `CA:FALSE` | `CA:FALSE` | `CA:FALSE` |
| `keyCertSign`      |          ✓           |            |            |            |
| `cRLSign`          |          ✓           |            |            |            |
| `keyEncipherment`  |                      |     ✓      |     ✓      |     ✓      |
| `digitalSignature` |                      |     ✓      |     ✓      |     ✓      |
| `serverAuth` EKU   |                      |     ✓      |     ✓      |            |
| `clientAuth` EKU   |                      |            |     ✓      |     ✓      |
| SANs required      |                      |     ✓      |     ✓      |            |
| SKI / AKI          |       SKI only       |    both    |    both    |    both    |

### Keys: `genpkey` vs `genrsa`, and EC

`genpkey` is the modern front-end and the reason every command above uses it:

|                   | `genrsa`                         | `genpkey`                    |
| ----------------- | -------------------------------- | ---------------------------- |
| Scope             | RSA only                         | RSA, EC, Ed25519, X25519, …  |
| Output format     | PKCS#1 (`BEGIN RSA PRIVATE KEY`) | PKCS#8 (`BEGIN PRIVATE KEY`) |
| Future-proof      | No                               | Yes                          |
| `pkey` compatible | Yes (reads both)                 | Native                       |

For a new deployment, EC keys are smaller, faster, and have no downside in a
private PKI. It's a drop-in replacement for the key-generation step — the CA,
`req`, and `x509` commands are all algorithm-agnostic:

```bash
# CA (stronger curve for the root)
openssl genpkey -algorithm EC -pkeyopt ec_paramgen_curve:P-384 -out ca.key

# Leaves
openssl genpkey -algorithm EC -pkeyopt ec_paramgen_curve:P-256 -out server.key
```

Serial numbers: `-CAcreateserial` creates `ca.srl` on first use, then increments
on each signing. It's idempotent, so it's safe to leave on every command. For an
audit trail, version-control `ca.srl` alongside the issued certs.

### From homelab to production

The workflow above is complete for a homelab; production layers discipline on
top, not new tools:

- **Keep `ca.key` offline.** Sign leaves from an air-gapped or HSM-backed
  workstation, never from the cluster nodes. For higher assurance, run a two-tier
  PKI: an offline root that signs an online intermediate, which signs leaves — an
  intermediate compromise is then recoverable without touching the root.
- **Per-node peer certs, not a shared one.** Each member gets a peer cert with
  only its own SANs, and each client (kube-apiserver, an operator's `etcdctl`, a
  backup tool) gets its own client cert with a distinct CN so etcd can authorize
  per-CN. A compromised key then blasts a smaller radius and rotates
  independently.
- **Short lifetimes with rotation.** ≤90-day leaves with automated rotation is
  the modern norm; 365-day leaves are acceptable only with calendar discipline.
  CA lifetime 5–10 years, and plan the rollover *before* expiry.
- **Algorithms.** RSA 3072+ for leaves, RSA 4096 for the CA — or switch entirely
  to EC P-256/P-384. Always `-sha256`/`-sha384`; reject SHA-1 anywhere.

## Gotchas

**`x509 -req` throws away the CSR's extensions.** SANs/EKUs in the CSR are just a
request; you must re-supply them with `-extfile`/`-extensions` at signing time or
the issued cert silently lacks them. Footgun and feature — the CA gets the final
say.

**Three different places for extensions.** `req -x509` reads `[req]
x509_extensions`, `req -new` reads `[req] req_extensions`, `x509 -req` reads
whatever `-extensions` names in the `-extfile`. Put a config key in the wrong slot
and it's silently ignored.

**`prompt = no` or OpenSSL interrogates you.** Without it, `req` prompts for DN
fields interactively instead of reading them from the config — easy to miss in a
scripted flow.

**A cert and its key can silently mismatch.** Always confirm the two SHA-256
pubkey fingerprints are identical (see Verification) before deploying; nothing
else catches a swapped key until the handshake fails at runtime.

**Shared vs per-node peer certs is a real tradeoff.** A single peer cert listing
every node's IPs in `[alt_names]` is simplest to stand up; per-node certs (one
`peer-node-N.ext` each, signed individually) are the production choice because a
leaked key then only affects one node and rotation is per-node.

## Open questions

- Revocation is untouched here. When a leaf key leaks before expiry, do you lean
  on short lifetimes, a CRL, or OCSP — and does etcd even honor CRL/OCSP, or is
  re-issuing the CA the only real lever in a private PKI?
- Automated short-lived rotation (an ACME-capable internal CA, or issuing on the
  fly) is the obvious next step past hand-rolled 365-day leaves — worth its own
  entry.
- Should new clusters just default to EC P-256 leaves? The only reason not to
  seems to be interop with something ancient, which a private PKI shouldn't have.

## References

- [etcd transport security model](https://etcd.io/docs/latest/op-guide/security/)
- [OpenSSL `x509v3_config`](https://www.openssl.org/docs/man3.0/man5/x509v3_config.html) — extension syntax
- [OpenSSL `req`](https://www.openssl.org/docs/man3.0/man1/openssl-req.html) and [`x509`](https://www.openssl.org/docs/man3.0/man1/openssl-x509.html) man pages
- [RFC 5280](https://datatracker.ietf.org/doc/html/rfc5280) — X.509 certificate and CRL profile (EKU, SAN, basic constraints)
