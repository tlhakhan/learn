---
layout: post
title: Claude Code slash commands (VS Code extension)
date: 2026-07-01
description: Reference for the slash commands the Claude Code VS Code extension exposes — what each does, when to use it, and how the extension's subset differs from the terminal CLI.
tags: [claude-code, tools, workflow, reference]
---

## Summary

Slash commands control the Claude Code *session* rather than talking to the
model. Typed at the start of a prompt, they switch models, manage the context
window, run a workflow, check usage, and so on. Open the menu by typing `/` in
the prompt box.

There are three kinds, and the distinction matters:

- **Built-in** — behavior coded into Claude Code itself (`/clear`, `/config`).
- **Skill** — a bundled *prompt* handed to Claude, which Claude can also invoke
  on its own when relevant. A skill isn't hardwired logic; it's an instruction
  file the model follows (you can write your own — see [skills](https://code.claude.com/docs/en/skills)).
- **Workflow** — a bundled task that fans work out across many background
  subagents. (None of the commands below are workflows, but you'll see the label
  in the docs.)

**This documents the VS Code extension, which is important.** Claude Code exists
as both a terminal CLI and a VS Code extension, and the extension deliberately
exposes only a **subset** of commands — the docs' own comparison table reads
*Commands and skills — CLI: All; VS Code extension: Subset*. The 29 commands
here are what my extension's `/` menu shows. Some of them open a **native VS Code
dialog** rather than a text flow (`/usage`, `/config`, `/context`), and items
marked with a terminal icon shell out to the integrated terminal. If you need a
command that isn't in the menu, run `claude` in the integrated terminal to get
the full CLI. The `/` menu itself is the source of truth for what's available to
you — availability shifts with plan, platform, and version.

## Key concepts

The 29 commands, grouped by what you'd reach for them for. `(Skill)` marks
bundled skills; everything else is built-in.

### Session & context

Every command here is about the **context window** — the model's working memory,
i.e. the running transcript it re-reads on every turn. It's finite, so managing
what's in it is real work.

- **`/clear [name]`** — Starts a fresh conversation with empty context. The old
  one stays retrievable in `/resume`; pass a name to label it. *When:* you're
  switching to an unrelated task and don't want the model dragging along
  irrelevant history. *Contrast:* `/compact` keeps the conversation but shrinks
  it; `/clear` abandons it. *Example:* `/clear` or `/clear auth-refactor`.
  Aliases `/reset`, `/new`.
- **`/compact [instructions]`** — Summarizes the conversation so far into a
  smaller form, freeing context while keeping the thread going. You can steer it:
  `/compact focus on the API changes`. In the extension this also happens
  automatically when the window fills. *When:* mid-task and running low on room.
- **`/context [all]`** — Shows what's currently *using* your context window as a
  colored grid — which tools, memory files, and history are taking space, with
  trim suggestions. *When:* diagnosing why context is filling up. *Example:*
  `/context all` for the full breakdown.

### Setup & config

- **`/init`** — Generates a starter `CLAUDE.md` for the repo. *Concept:*
  `CLAUDE.md` is a file Claude Code reads at the start of every session in that
  project — your standing instructions and context (this notebook has one).
  `/init` bootstraps it by scanning the codebase. *When:* first session in a new
  repo. *Example:* `/init`.
- **`/config [key=value …]`** — Opens the Settings UI (theme, model, output
  style). Recent versions let you set a value directly: `/config theme=dark`,
  `/config model=sonnet`. In the extension, `/` → General Config reaches the same
  place. *Example:* `/config`. Alias `/settings`.
- **`/update-config`** (Skill) — Edits your `settings.json` from a plain-English
  request. *Concept:* `settings.json` (in `~/.claude/` for you, `.claude/` for
  the project, shared between extension and CLI) is where permissions, env vars,
  and [hooks](https://code.claude.com/docs/en/hooks) are configured. A **hook**
  is a shell command Claude Code runs automatically at a fixed moment — before/
  after a tool runs, when a session starts or stops — an event listener for the
  agent (e.g. run the linter after every edit). *When:* "allow all git read
  commands", "run prettier after each edit". *Example:*
  `/update-config allow npm test`.
- **`/fewer-permission-prompts`** (Skill) — Scans your recent sessions for safe,
  read-only commands you keep approving and adds them to the project's permission
  allowlist so they stop prompting. *Concept:* Claude Code asks before running
  tools unless a [permission](https://code.claude.com/docs/en/permissions) rule
  pre-approves them. *When:* tired of confirming `git status`-type calls.

### Review & ship

These all read a **diff** — the set of changes since the last commit — so run
them before you move on.

- **`/code-review [low|medium|high|xhigh|max|ultra] [--fix] [--comment] [target]`**
  (Skill) — Reviews your uncommitted diff for correctness bugs *plus* reuse/
  simplification/efficiency cleanups. Effort scales `low` (few, high-confidence)
  → `max`; `ultra` runs a deep multi-agent review in the cloud
  ([ultrareview](https://code.claude.com/docs/en/ultrareview) — billed, 3 free on
  Pro/Max). `--fix` applies findings to your working tree; `--comment` posts them
  as inline GitHub PR comments. *When:* before committing. *Example:*
  `/code-review high`.
- **`/simplify [target]`** (Skill) — Cleans up your changed code (reuse,
  simplification, efficiency, right level of abstraction) and applies the fixes.
  *Concept:* it runs four review [subagents](https://code.claude.com/docs/en/sub-agents)
  — separate Claude instances working in parallel, results merged back. Quality
  only; it does **not** look for bugs (use `/code-review` for that). *When:*
  tidying a change before shipping.
- **`/security-review`** — Reviews the branch's pending changes specifically for
  security issues — injection, auth mistakes, data exposure — not general bugs.
  *When:* touching auth, input handling, secrets, or anything internet-facing.
- **`/review [PR]`** — Same engine as `/code-review`, but against a GitHub pull
  request instead of your local diff. *When:* reviewing a PR (yours or a
  teammate's); for local uncommitted work use `/code-review`. *Example:*
  `/review 128`, or `/review` to pick from open PRs.

### Run & verify

The difference from tests: these launch the **real app** and observe it.

- **`/run`** (Skill) — Launches your project's app and drives it so you can see a
  change working in the running program, not just in green tests. *When:*
  confirming a UI/behavior change end-to-end. First use may need
  `/run-skill-generator`.
- **`/verify`** (Skill) — Builds, runs, and observes the app to confirm a
  specific change does what it should — verification by watching real behavior.
  *When:* validating a fix before calling it done. *Example:*
  `/verify the logout button clears the session`.
- **`/run-skill-generator`** (Skill) — Writes a per-project
  [skill](https://code.claude.com/docs/en/skills) teaching `/run` and `/verify`
  how to build, launch, and drive *your* app from a clean checkout. *Concept:* a
  skill is a reusable instruction file stored in the repo — here, the recipe for
  starting your app. *When:* the first time `/run` doesn't know how to launch
  your project.

### Automation & parallel work

- **`/loop [interval] [prompt]`** (Skill) — Re-runs a prompt on a schedule while
  the session stays open. *When:* polling for something to change — "is the
  deploy done?", "any new CI failures?" With an interval it waits between runs;
  without one, Claude paces itself. *Example:* `/loop 5m check if the deploy
  finished`. Alias `/proactive`.
- **`/schedule [description]`** — Creates and manages
  [routines](https://code.claude.com/docs/en/routines): tasks that run on a
  cron-like schedule on Anthropic's **cloud**, not your machine. *Concept:* like
  a cron job, but the worker is a Claude Code agent in the cloud, so it runs even
  with your editor closed. *When:* recurring automation ("every morning
  summarize new issues"). Billed/cloud. Alias `/routines`.
- **`/batch <instruction>`** (Skill) — For a big change across many files,
  decomposes it into 5–30 independent units and runs them in parallel. *Concepts:*
  each unit runs as a background [subagent](https://code.claude.com/docs/en/sub-agents)
  (a separate Claude instance) in its own [git worktree](https://code.claude.com/docs/en/worktrees)
  — a throwaway second checkout of your repo on its own branch, so the parallel
  agents don't overwrite each other's files. Each unit implements, tests, and
  opens its own PR. Requires git; cloud/heavy. *Example:*
  `/batch migrate src/ from Solid to React`.
- **`/goal [condition|clear]`** — Sets a persistent objective Claude keeps
  pursuing across turns until it's met, instead of stopping after each reply.
  *When:* a multi-step end state you want driven to completion. *Example:*
  `/goal all lint and type errors are gone`; `/goal clear` to cancel. See
  [goal](https://code.claude.com/docs/en/goal).

### Session control & skills

- **`/remote-control`** — Exposes this local session so you can drive it from
  claude.ai on another device. *Concept:* the session keeps running on your
  machine, but a web/mobile client can send prompts and see output — a remote
  control for the agent. *When:* stepping away but wanting to keep steering a run
  from your phone. Alias `/rc`. See
  [remote control](https://code.claude.com/docs/en/remote-control).
- **`/reload-skills`** — Re-scans your skill/command directories so skills you
  added or edited on disk appear without restarting. *Concept:* skills are files
  (in `~/.claude/skills` or the project) loaded at startup, so edits mid-session
  normally don't take effect. *When:* authoring or tweaking a skill and testing
  it now.

### Usage & diagnostics

- **`/usage`** — Opens the Account & usage dialog: your plan, how much of your
  session/weekly limits you've used, and reset times. On paid plans it breaks
  usage down by skill, subagent, plugin, and MCP server, and flags what's driving
  it (cache misses, long context). *When:* checking how close you are to a limit.
  Aliases `/cost`, `/stats`.
- **`/usage-credits`** (formerly **`/extra-usage`**) — Configures usage credits:
  pay-as-you-go overflow so you can keep working after hitting your plan's limit
  instead of being blocked. Both names show in the menu — same command. *When:*
  you routinely hit limits mid-task.
- **`/debug [description]`** (Skill) — Turns on debug logging from this point and
  helps diagnose problems by reading that log. *Concept:* Claude Code doesn't
  write a verbose log by default (unless launched with `claude --debug`), so
  `/debug` starts capturing one now. *When:* something's misbehaving and you want
  the internals. *Example:* `/debug the file watcher isn't picking up changes`.
- **`/heapdump`** — Writes a memory snapshot of the Claude Code process to your
  Desktop for diagnosing high memory use. *Concept:* Claude Code runs on Node.js;
  a heap snapshot dumps everything using memory at that instant so support can
  find leaks. *When:* the app is eating RAM and you're filing a bug.
- **`/insights`** — Generates a report on how you've been using Claude Code —
  what you work on, interaction patterns, and where you hit friction. *When:*
  reflecting on your own workflow or deciding what to automate.

### Reference, design & onboarding

- **`/claude-api [migrate|managed-agents-onboard]`** (Skill) — Loads accurate
  Claude API / SDK reference for your project's language (model IDs, tool use,
  streaming, batches, pitfalls) so Claude writes correct API code instead of
  guessing. Also activates automatically when your code imports `anthropic` /
  `@anthropic-ai/sdk`. `/claude-api migrate` upgrades existing API code to a newer
  model. *When:* building anything against the Claude API.
- **`/design-sync [hint]`** (Skill) — Converts your repo's React design system
  and uploads it to [Claude Design](https://claude.ai/design) so AI-generated
  designs use your real components. *Concept:* a design system is your library of
  reusable UI components (buttons, cards, etc.); this mirrors it to the cloud
  tool. Needs claude.ai auth (via `/design-login`); the first sync can take hours
  on a large repo. *When:* you maintain a React component library and want
  Claude's designs to match it.
- **`/team-onboarding`** — Generates a markdown onboarding guide from your last
  30 days of Claude Code usage — the commands, MCP servers, and patterns you rely
  on — that a teammate can paste as a first message to get set up like you.
  *When:* bringing someone onto a project or sharing your setup. Pro/Max/Team/
  Enterprise also get a share link.

## Gotchas

- **The extension is a subset of the CLI.** Type `/` to see what's actually
  available; for a command that isn't there, run `claude` in the integrated
  terminal (`` Cmd+` `` / `` Ctrl+` ``) to get the full set. Run `/ide` inside a
  terminal session to reconnect it to VS Code.
- **CLI-only conveniences are missing** in the extension: the `!` bash shortcut
  and tab completion. [MCP](https://code.claude.com/docs/en/mcp) config is partial
  — add servers with `claude mcp add` in the terminal, then manage them with
  `/mcp`.
- **Some commands open native UI, not a text flow** — `/usage`, `/config`, and
  `/context` open dialogs; don't expect a terminal-style response.
- **Aliases hide duplicates:** `/cost` = `/stats` = `/usage`; `/extra-usage` =
  `/usage-credits`; `/clear` = `/reset` = `/new`; `/remote-control` = `/rc`;
  `/schedule` = `/routines`; `/loop` = `/proactive`.
- **Cloud/billed commands:** `/code-review ultra`, `/schedule`, `/batch` (and
  `/design-sync`) run work on Anthropic infrastructure and can consume usage.
- **Skills can fire on their own.** A `(Skill)` command is a prompt Claude may
  invoke automatically when relevant (e.g. `/claude-api` on an SDK import);
  built-ins only run when you type them.

## Open questions

- Which of these are worth wiring into a [hook](https://code.claude.com/docs/en/hooks)
  or a `/loop` so they run without me remembering to?
- Which prompts I type repeatedly should become my own skills under
  `~/.claude/skills` (and show up as `/`-commands)?
- Will the extension's subset grow toward CLI parity, or stay curated?

## References

- [Commands](https://code.claude.com/docs/en/commands) — full CLI command
  reference (~80 commands)
- [Use Claude Code in VS Code](https://code.claude.com/docs/en/vs-code) — the
  extension, its command menu, and CLI-vs-extension differences
- [Interactive mode](https://code.claude.com/docs/en/interactive-mode) —
  keyboard shortcuts and input modes
- Concept deep-dives: [hooks](https://code.claude.com/docs/en/hooks),
  [MCP](https://code.claude.com/docs/en/mcp),
  [git worktrees](https://code.claude.com/docs/en/worktrees),
  [subagents](https://code.claude.com/docs/en/sub-agents),
  [skills](https://code.claude.com/docs/en/skills),
  [checkpointing](https://code.claude.com/docs/en/checkpointing),
  [routines](https://code.claude.com/docs/en/routines),
  [workflows](https://code.claude.com/docs/en/workflows),
  [remote control](https://code.claude.com/docs/en/remote-control),
  [memory](https://code.claude.com/docs/en/memory),
  [permissions](https://code.claude.com/docs/en/permissions),
  [ultrareview](https://code.claude.com/docs/en/ultrareview)
