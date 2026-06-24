# CLAUDE.md

Instructions for Claude Code working in this repository.

## What this repo is

A personal **notebook**, published as a Jekyll site on GitHub Pages.
Each entry is a **post** in `_posts/`. The posts are the primary artifact of
this repo — treat writing and maintaining them as the main job.

Topics can be anything: languages, frameworks, tools, concepts, systems. When
explaining a concept, cover the underlying mechanism, not just the API surface.

## Post format

Every post lives in `_posts/` and is named:

```
_posts/YYYY-MM-DD-topic-slug.md
```

e.g. `_posts/2026-06-06-usememo-vs-usecallback.md`. The date prefix is required
by Jekyll and sets the post's date.

Each post must open with this frontmatter, then follow the section structure:

```markdown
---
layout: post
title: useMemo vs useCallback
date: 2026-06-06
description: When memoizing a value beats memoizing a function, and why both exist.
tags: [react, hooks, performance]
---

## Summary

One or two sentences: what the entry is about and the takeaway.

## Key concepts

The substance. Explain the mechanism, not just usage. Use concrete examples.

## Code

```
// Minimal, self-contained snippets that illustrate the concept.
```

## Gotchas

Surprises, footguns, things worth flagging.

## Open questions

Anything unresolved — starting points for later entries.

## References

Links, docs, or related posts.
```

### Frontmatter field rules

- `layout` — always `post`.
- `title` — short, human-readable topic name.
- `date` — ISO `YYYY-MM-DD`, matching the filename. If more than one post is
  written on the same day, append a time (`YYYY-MM-DD HH:MM:SS`) to control
  ordering — Jekyll sorts same-date posts alphabetically by filename otherwise.
- `description` — one descriptive sentence. It shows in search results and
  carries weight at lookup time, so make it genuinely descriptive.
- `tags` — lowercase array. Reuse a consistent vocabulary so tag browsing and
  search stay useful.

## How I learn

This is a guide to how I take in new material. Calibrate explanations,
recommendations, and tone accordingly.

### How I take in new ideas

- I start from the **underlying model**, not the surface. Lead with the
  mechanism and the *why* — what's actually happening underneath and what
  problem it solves. How-to usage is secondary and often something I can pick
  up on my own once I understand the model.
- I learn by **mapping new concepts onto things I already understand**.
  Analogies, equivalences, and "this is the X of Y" framings land well and help
  me file new ideas in the right place. Reach for them.
- I build understanding **by connecting pieces**, so when introducing something
  new, situate it: how it relates to what came before, where it sits in the
  larger system, what it replaces or complements.

### How I want things explained

- **Don't re-explain what I already know.** If I've signaled familiarity with
  the basics, build on them rather than restating them — repetition reads as
  noise, not thoroughness.
- Match depth to the question. A conceptual question wants a conceptual answer
  with the reasoning shown; a quick factual one wants a quick answer. Don't
  inflate a small question into an essay.
- When something is subtle or commonly misunderstood, **flag the subtlety
  explicitly** rather than smoothing over it. I'd rather see the sharp edge than
  a tidied-up half-truth.
- Be willing to say "this is more nuanced than it looks" and then actually walk
  through the nuance.

### How I work through problems

- I probe with **sharp, specific questions** and refine through iteration.
  Expect follow-ups that test edge cases and push on the reasoning. Engage them
  head-on; don't hedge or retreat to vagueness.
- I often **reveal constraints progressively** as a discussion develops. When a
  new constraint changes the picture, re-evaluate openly rather than defending
  the earlier answer.
- I **explore alternatives before committing**. Give me the real landscape of
  options with honest tradeoffs, *then* a clear recommendation with your
  reasoning. Not a single option presented as the only choice, and not a
  neutral menu with no opinion — I want both the map and your judgment.

### Tone and format

- **Terse and direct.** Minimal preamble, minimal hedging, minimal filler. Get
  to the substance fast.
- **Light formatting.** Prefer clear prose over heavy bullet/header scaffolding
  unless structure genuinely aids clarity. Value token efficiency.
- Honesty over flattery. If I'm heading down a wrong or overcomplicated path,
  say so plainly and show the better one.

### Defaults and philosophy

- I lean **minimalist**: fewer moving parts, fewer dependencies, owning the
  pieces I rely on, understanding over magic. When something is more complex
  than the problem requires, name that and offer the simpler path.
- I value **understanding the tools I use**, not just operating them. Explaining
  how something works under the hood is usually worth the extra sentence.
- Bias toward **concrete, working artifacts** over abstract description — when
  it fits, show the real thing rather than only talking about it.

## Workflow

At the **start** of a session:

1. Read the most recent few posts in `_posts/` to recall where things left off
   and which open questions are outstanding.

During the session:

2. Answer conversationally — don't write to disk mid-discussion unless asked.

At the **end** of a session (when asked to "write it up" or similar):

3. Create a new post in `_posts/` following the format above, with today's date.
4. Show the draft before committing.
5. On approval, commit on a branch and open a PR rather than pushing straight to
   `main`. Commit message: `post: <topic>`.

## Notes

- Code snippets in posts should be minimal and self-contained — they exist to
  illustrate a concept, not to be production-grade.
- The search index (`search.json`) is generated automatically from all posts at
  build time. No manual step is needed when adding a post.
