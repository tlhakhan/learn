# Notebook

A personal notebook published as a Jekyll site on GitHub Pages with
client-side search.

## Adding a post

Create a Markdown file in `_posts/` named `YYYY-MM-DD-topic-slug.md` with the
frontmatter and sections described in `CLAUDE.md`. Commit and push. The search
index rebuilds automatically.

## Local preview (optional)

On the Mac mini, with Ruby installed:

```bash
bundle install
bundle exec jekyll serve
```

Then open http://localhost:4000. Note: search reads `search.json`, which only
exists after a build — it works on `jekyll serve` and on the deployed site, not
on a raw file open.

## How search works

`search.json` is a Liquid template that emits every post's title, url, date,
description, tags, and stripped body text as JSON at build time. `search.html`
loads that file and filters it in the browser — no plugins, no external service,
nothing to maintain. Result listings show the title and description, so a clear
`description` makes results easier to scan.
