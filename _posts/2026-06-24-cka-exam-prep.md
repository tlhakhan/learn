---
title: "CKA Exam Prep — 7-Day Plan & Reference"
description: "Tailored 7-day study schedule plus exam reference for the Certified Kubernetes Administrator exam."
author: "Tenzin"
created: 2026-06-24
exam: "Certified Kubernetes Administrator (CKA)"
exam_date: 2026-07-09
k8s_version: "v1.35"
pass_mark: "66%"
exam_duration: "120 minutes"
curriculum_version: "CNCF CKA v1.35"
domains:
  - { name: "Troubleshooting", weight: "30%" }
  - {
      name: "Cluster Architecture, Installation & Configuration",
      weight: "25%",
    }
  - { name: "Services & Networking", weight: "20%" }
  - { name: "Workloads & Scheduling", weight: "15%" }
  - { name: "Storage", weight: "10%" }
allowed_resources:
  - https://kubernetes.io/docs
  - https://kubernetes.io/blog/
  - https://helm.sh/docs
  - https://gateway-api.sigs.k8s.io
tags: [kubernetes, cka, certification, study-plan, devops]
status: draft
last_updated: 2026-06-24
---

# CKA Exam Prep — 7-Day Plan & Reference

**Target exam date:** July 9, 2026
**Exam version:** Kubernetes **v1.35** (re-verify at the [LF candidate FAQ](https://docs.linuxfoundation.org/tc-docs/certification/faq-cka-ckad-cks) a few days before — it realigns 4–8 weeks after a K8s release)
**Format:** 2 hours, ~15–25 performance-based tasks in a live terminal, open-book (kubernetes.io/docs, kubernetes.io/blog, helm.sh/docs in one tab)
**Pass mark:** 66%, partial credit awarded, auto-graded, results within ~24h

> This plan assumes you already know Kubernetes internals well (kubeadm, etcd/PKI, CNI, networking). It is tuned for **exam speed, imperative fluency, and the highest-weight domains**, not for learning concepts from scratch. Drop the basics, drill the muscle memory.

---

## Exam setup at a glance

| Item            | Detail                                                                          |
| --------------- | ------------------------------------------------------------------------------- |
| K8s version     | v1.35                                                                           |
| Time            | 120 minutes                                                                     |
| Tasks           | ~15–25, each weighted (do high-weight first)                                    |
| Pass            | 66% (partial credit)                                                            |
| Clusters        | ~6 clusters, each with its own nodes; switch with `kubectl config use-context`  |
| Node access     | SSH to control-plane/worker nodes, with sudo                                    |
| Cluster tooling | kubeadm, etcd, CoreDNS, a CNI plugin; Helm + Kustomize expected                 |
| Allowed docs    | kubernetes.io/docs, kubernetes.io/blog, helm.sh/docs (one browser tab)          |
| Simulator       | 2x Killer.sh sessions (CKA-A, CKA-B), 17 questions each, 36h access per session |

**Critical exam habit:** every task starts with a context-switch command. Run it. A correct answer in the wrong cluster scores zero.

---

## Allowed resources DURING the exam (official list)

Open-book, but only these sites, only through the browser inside the exam VM. No Google, Stack Overflow, GitHub, or personal notes. The docs search bar is fine, **but you must not open external search results.**

- **Kubernetes Docs** — https://kubernetes.io/docs
- **Kubernetes Blog** — https://kubernetes.io/blog/
- **Helm Docs** — https://helm.sh/docs
- **Gateway API Docs** — https://gateway-api.sigs.k8s.io (CKA-only)
- **Quick Reference** links shown in a given task's info box

> Every per-domain link below resolves to one of these four allowed domains, so they're all usable live. Two links elsewhere in this file — the LF candidate FAQ and the CNCF curriculum on GitHub — are **pre-exam reference only** and will NOT open during the exam.

---

## Grading breakdown (CNCF curriculum, v1.35) + doc links

> All links in this section are on allowed domains (kubernetes.io/docs, helm.sh/docs, gateway-api.sigs.k8s.io) and can be opened during the exam.

Five domains. **Troubleshooting (30%) and Cluster Architecture (25%) are 55% of the exam** — spend your time accordingly.

### 1. Cluster Architecture, Installation & Configuration — 25%

RBAC, kubeadm cluster create/upgrade, HA control plane, Helm, Kustomize, extension interfaces (CNI/CSI/CRI), CRDs, Operators, etcd backup/restore.

- RBAC — https://kubernetes.io/docs/reference/access-authn-authz/rbac/
- kubeadm create cluster — https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/
- kubeadm upgrade — https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/
- HA topology — https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/high-availability/
- etcd backup/restore — https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/
- Kustomize — https://kubernetes.io/docs/tasks/manage-kubernetes-objects/kustomization/
- Helm — https://helm.sh/docs/
- CRDs — https://kubernetes.io/docs/tasks/extend-kubernetes/custom-resources/custom-resource-definitions/
- Operators — https://kubernetes.io/docs/concepts/extend-kubernetes/operator/

### 2. Workloads & Scheduling — 15%

Deployments & rollouts, ConfigMaps/Secrets, HPA, scheduling (affinity, taints/tolerations, nodeSelector).

- Deployments — https://kubernetes.io/docs/concepts/workloads/controllers/deployment/
- ConfigMaps — https://kubernetes.io/docs/concepts/configuration/configmap/
- Secrets — https://kubernetes.io/docs/concepts/configuration/secret/
- HPA — https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/
- Assigning Pods to nodes — https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/
- Taints & tolerations — https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/

### 3. Storage — 10%

StorageClasses, dynamic provisioning, PV/PVC, access modes, reclaim policies.

- Storage classes — https://kubernetes.io/docs/concepts/storage/storage-classes/
- Persistent volumes — https://kubernetes.io/docs/concepts/storage/persistent-volumes/
- Dynamic provisioning — https://kubernetes.io/docs/concepts/storage/dynamic-provisioning/

### 4. Services & Networking — 20%

Pod connectivity, NetworkPolicies, Service types & endpoints, Gateway API, Ingress, CoreDNS.

- Services — https://kubernetes.io/docs/concepts/services-networking/service/
- Network policies — https://kubernetes.io/docs/concepts/services-networking/network-policies/
- Ingress — https://kubernetes.io/docs/concepts/services-networking/ingress/
- Gateway API (concepts) — https://kubernetes.io/docs/concepts/services-networking/gateway/
- Gateway API (full reference, CKA-allowed) — https://gateway-api.sigs.k8s.io
- DNS for services/pods (CoreDNS) — https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/

### 5. Troubleshooting — 30% ← highest weight

Cluster/node failures, control-plane components, kubelet, networking, app failures, resource monitoring, container logs.

- Debug a cluster — https://kubernetes.io/docs/tasks/debug/debug-cluster/
- Debug running Pods — https://kubernetes.io/docs/tasks/debug/debug-application/debug-running-pod/
- Debug services — https://kubernetes.io/docs/tasks/debug/debug-application/debug-service/
- kubelet troubleshooting — https://kubernetes.io/docs/tasks/debug/debug-cluster/kubelet/
- Resource usage monitoring — https://kubernetes.io/docs/tasks/debug/debug-cluster/resource-usage-monitoring/

Full official curriculum (pre-exam reference only — not accessible during the exam): https://github.com/cncf/curriculum

---

## Day 0 (do this before Day 1)

- Run both **Killer.sh** sessions? No — save **one for the final dry run**. Activate the first now if you haven't, since each session is only 36h.
- Set up a local 2-node kubeadm sandbox (you already have sparkle-1 / nvidia-1 + libvirt; spin up 3 throwaway Ubuntu 24.04 VMs: 1 CP + 2 workers). You want a cluster you can **break and rebuild fast**.
- Lock in your exam shell setup so it's automatic on Day 1 of the real exam:
  ```bash
  alias k=kubectl
  export do="--dry-run=client -o yaml"
  export now="--force --grace-period=0"
  source <(kubectl completion bash)
  complete -o default -F __start_kubectl k
  # vim: set ts=2 sw=2 et, and 'set paste' awareness for YAML
  ```

---

## The 7-day plan

### Day 1 — Troubleshooting I (the 30% domain, part 1)

Highest weight, so it goes first and gets two days.

- Broken kubelet: stop it, misconfigure `/var/lib/kubelet/config.yaml` and `--kubeconfig`, fix it. `systemctl status kubelet`, `journalctl -u kubelet`.
- Static pod failures: break a control-plane manifest in `/etc/kubernetes/manifests/`, watch the API server / scheduler / controller-manager go down, recover.
- `kubectl get nodes` NotReady scenarios: CNI down, kubelet down, cert expiry, disk pressure.
- Drill: `crictl ps`, `crictl logs`, `crictl inspect` on a node where kubectl can't reach a pod.
- **Reps target:** rebuild a NotReady node to Ready 5+ times until it's reflexive.

### Day 2 — Troubleshooting II + etcd

- App-level debugging: CrashLoopBackOff, ImagePullBackOff, wrong probes, OOMKills, pending pods (taints, resources, affinity). Read events first: `kubectl get events --sort-by=.metadata.creationTimestamp`.
- Service/networking debug: endpoints empty, wrong selector, DNS resolution (`kubectl run tmp --rm -it --image=busybox -- nslookup svc`).
- **etcd snapshot + restore** end-to-end (you know etcd/TLS cold, but drill the exam mechanics):
  ```bash
  ETCDCTL_API=3 etcdctl snapshot save /tmp/snap.db \
    --cacert=... --cert=... --key=... --endpoints=https://127.0.0.1:2379
  ETCDCTL_API=3 etcdctl snapshot restore /tmp/snap.db --data-dir=/var/lib/etcd-restore
  # then point the static pod manifest at the new data-dir and restart
  ```
- **Reps target:** full backup→destroy→restore→verify cycle 3 times under 10 min each.

### Day 3 — Cluster Architecture I (the 25% domain)

- `kubeadm init` from scratch + join a worker. Time yourself.
- **kubeadm upgrade** of a CP node and a worker (`kubeadm upgrade plan/apply`, drain, `apt` the kubelet/kubectl, uncordon). This is a near-guaranteed exam task — make it boring.
- RBAC: create a Role/ClusterRole + binding + ServiceAccount, then **verify** with `kubectl auth can-i --as=system:serviceaccount:ns:sa`.
- **Reps target:** one clean upgrade + one RBAC grant verified by `auth can-i`.

### Day 4 — Cluster Architecture II: Helm, Kustomize, CRDs

These are the newer/under-practiced bits even for strong admins.

- Helm: `repo add`, `search`, `install`, `upgrade`, `--set`, `template`, `uninstall`, list releases. Know `helm install --dry-run`.
- Kustomize: base + overlay, `kubectl apply -k`, patches, `images:` and `replicas:` transforms.
- CRDs/Operators: apply a CRD, create a CR instance, install a simple operator. Understand the apply order (CRD before CR).
- **Reps target:** install one chart via Helm and one app via Kustomize overlay, both from docs only.

### Day 5 — Services & Networking (20%) + Storage (10%)

- Services: ClusterIP/NodePort/LoadBalancer, multi-port, `kubectl expose`, endpoints inspection.
- **Gateway API** (GA, on the exam): GatewayClass → Gateway → HTTPRoute. Practice from the docs since the YAML is less reflexive than Ingress.
- Ingress + ingress controller routing rules.
- NetworkPolicy: default-deny, then allow-by-label/namespace/port — both ingress and egress.
- CoreDNS: inspect the ConfigMap, understand cluster DNS resolution paths.
- Storage: StorageClass + dynamic PVC, manual PV/PVC bind, access modes, reclaim policy, resize.
- **Reps target:** a default-deny NetworkPolicy + one allow rule, and a dynamically-provisioned PVC bound to a pod.

### Day 6 — Workloads & Scheduling (15%) + speed pass

- Deployments: rollout, `set image`, `rollout undo`, `rollout status`, scaling, revision history.
- ConfigMaps/Secrets: create imperatively, mount as env and as volume.
- Scheduling: nodeSelector, affinity/anti-affinity, taints/tolerations, manual scheduling via `nodeName`, static pods.
- HPA: create against a Deployment with metrics-server present.
- **Speed pass:** do 10–12 mixed imperative tasks against the clock. Everything via `k ... $do > x.yaml && vim x.yaml && k apply -f x.yaml`. No hand-writing YAML from a blank file.

### Day 7 — Full dry run + cleanup

- **Use your second Killer.sh session as a timed 2-hour mock.** Treat it as the real thing: context-switch every task, flag-and-skip hard ones, high-weight first.
- Review only the questions you missed; re-drill those exact mechanics.
- Re-read the [LF FAQ](https://docs.linuxfoundation.org/tc-docs/certification/faq-cka-ckad-cks) and confirm the K8s version hasn't shifted.
- Light day otherwise. Don't cram new material the night before.

---

## Exam-day tactics

- First 60 seconds: set aliases, `$do`/`$now`, completion, vim config.
- Read each task's **weight**. Do the heavy ones first; flag and skip anything that stalls you >~8 min.
- **Always run the provided `kubectl config use-context` line first.** Confirm with `k config current-context`.
- Bookmark the doc pages above; navigate by docs search, don't browse.
- Imperative-first: generate YAML with `$do`, edit, apply. Hand-write only what you must (PV, NetworkPolicy, Gateway).
- Partial credit is real — get the easy 70% of a task done rather than perfecting one.
- Verify your work: `k get`, `k describe`, `auth can-i`, `nslookup`, `curl` from a temp pod.

## Quick command refresh to keep warm

```bash
k run nginx --image=nginx $do
k create deploy web --image=nginx --replicas=3 $do
k expose deploy web --port=80 --target-port=8080 --type=NodePort $do
k create cm app --from-literal=KEY=val $do
k create secret generic db --from-literal=pw=s3cr3t $do
k create role r --verb=get,list --resource=pods $do
k create rolebinding rb --role=r --serviceaccount=ns:sa $do
k get events --sort-by=.metadata.creationTimestamp
k get pods -A -o wide --field-selector status.phase!=Running
```
