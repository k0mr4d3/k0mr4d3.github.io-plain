# Post Tags & Tag Archives — Design

## Goal

Add WordPress-style tags to the blog:

- Posts can declare tags.
- Each tag has an archive page at `/tag/<tag>/` listing its posts.
- A tag index at `/tag/` lists all tags.
- Posts display their tags, linking to the archives.

Existing post URLs (`/<post-slug>/`) are unchanged.

## Constraints

- The site is built by **GitHub Pages' native builder**, which only allows
  whitelisted plugins (currently `jekyll-sitemap`). Tag-archive plugins such as
  `jekyll-tagging` are **not** available, and no CI/Actions build is used.
- The blog's ethos is minimal, plugin-free, and works without CSS/JS. The design
  must not introduce client-side JavaScript or a new build runtime.
- Because the native builder cannot generate a page per tag on its own, the
  per-tag archive files must exist as real files in the repo.

## Data model

Tags are written as standard Jekyll YAML-array front matter, which is what
populates `site.tags`:

```yaml
---
layout: post
title: "Welcome to plain-html-blog"
date: 2016-04-01 09:00:00 +0200
updated: 2018-11-23 21:40:58 +0100
category: posts
tags: [jekyll, plain-html]
---
```

Rules:

- Tag names are **lowercase-hyphenated** (e.g. `smoke-sim`) so the display name
  and the URL slug are identical. This avoids any slugify-vs-`site.tags`-key
  mismatch.
- Posts with no `tags:` key simply have no tags.

## URLs & pages

| URL             | Content                                                        |
|-----------------|---------------------------------------------------------------|
| `/tag/`         | Index of all tags, alphabetical, each with a post count + link |
| `/tag/<tag>/`   | Archive listing every post with `<tag>`, reverse-chronological |
| `/<post-slug>/` | Individual post (unchanged)                                    |

Components:

- **`_layouts/tag.html`** — shared layout holding the archive markup. Loops
  `site.tags[page.tag]` to list posts.
- **`tag/<tag>/index.html`** — a ~4-line generated stub per tag:
  ```
  ---
  layout: tag
  tag: jekyll
  permalink: /tag/jekyll/
  ---
  ```
- **`tag/index.html`** — hand-written Liquid page (committed once). Jekyll renders
  it live from `site.tags`, so it never needs regeneration.

## Generation

- The **tag index** (`/tag/`) is static Liquid — no generation needed.
- The **per-tag stubs** are produced by a generator script `scripts/gen-tags`:
  - Scans the `tags:` front matter of every post across **all** `_posts`
    directories (`posts/_posts`, `tutorials/_posts`, and any future section),
    matching how `site.tags` aggregates tags globally.
  - Writes one `tag/<tag>/index.html` stub per unique tag.
  - Deletes stubs for tags that are no longer used by any post (no orphans).
  - Written in **`sh` + `awk`**, both already present in the user's Git Bash —
    the same toolchain as the existing `pre-commit` hook. No new runtime
    dependency (no Ruby/Python/Node).
- The generator is **wired into the `pre-commit` hook** so stubs regenerate and
  are `git add`-ed automatically on every commit. It can also be run manually.

### Interaction with the existing pre-commit hook

The repo already has a `pre-commit` hook (installed at `.git/hooks/pre-commit`,
sourced from `scripts/pre-commit`) that stamps the `updated:` field of modified
`.md` files. The tag generator runs as an additional step in that hook. Order:
regenerate tag stubs, stage them, then run the `updated:` stamping.

## Display

The `post` layout gains a "Tags:" line in the footer area, rendering each of the
post's tags as a link to `/tag/<tag>/`. Posts with no tags render nothing.

## Out of scope

- Tags embedded in post permalinks (e.g. `/<tag>/<post-slug>/`).
- Tag pages for the `tutorials` section beyond what `site.tags` naturally
  aggregates (tags are global across all posts; the same tag used in a tutorial
  and an essay shares one archive).
- Client-side filtering / search.
- Migrating existing posts to have tags (can be done incrementally by editing
  front matter).
