# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Personal Hugo blog (`Andi Skrgat blog`) deployed to GitHub Pages at `https://as51340.github.io/blog/`. The site has no JS/Node build of its own — it is pure Hugo content + a themed layout.

## Common commands

```bash
# Local preview (includes drafts)
hugo server -D

# Production build (matches the GitHub Actions workflow)
hugo --minify --baseURL "https://as51340.github.io/blog/"

# New article (replace section/slug)
hugo new content/engineering/my-post.md
```

The CI pins **Hugo extended 0.108.0** (`.github/workflows/hugo.yml`). Use that locally if reproducing build output exactly matters; newer Hugo versions usually work but may shift behavior.

## Theme is a git submodule

`themes/charlolamode` is a submodule pointing at `git@github.com:as51340/hugo-theme-charlolamode.git` (SSH). After a fresh clone the directory is empty and `hugo` will fail with "module not found". Run:

```bash
git submodule update --init --recursive
```

CI passes `submodules: recursive` to `actions/checkout`, so deploys don't need manual init.

## Content layout

Articles live under `content/<section>/` where section ∈ `engineering/`, `football/`, `about/`. Each section has an `_index.md` that controls the section landing page; individual posts are siblings.

`config.yml` declares `params.mainSections: [articles]`, but the actual on-disk sections are `engineering` and `football`. The homepage uses `profileMode` (config.yml), so the `mainSections` mismatch isn't currently visible — keep this in mind if switching the homepage to a post list.

Menu entries (`menu.main` in `config.yml`) are wired to `/about/`, `/engineering/`, `/football/` — adding a new top-level section means updating both the directory **and** the menu.

## Front matter convention

Existing posts use a minimal header (see `content/engineering/ha_2026.md`):

```yaml
---
title: ...
type: page
description: ...
topic: ...
---
```

`archetypes/default.md` sets `draft: true` by default, so new posts won't appear in a non-`-D` build until that flag is removed.

## Asset & link conventions

- PDFs live in `static/pdfs/` and are linked from posts as `/blog/pdfs/<file>.pdf` (note the `/blog/` prefix — it matches `baseURL`'s path component, not Hugo's automatic basepath handling). Don't drop the `/blog/` when adding new links or local previews will look fine but production links will 404.
- Images go in `static/images/` (favicons referenced from `config.yml` `params.assets`) or top-level `images/`.
- Custom CSS/JS overrides go in `assets/css/` and `assets/js/`; layout overrides in `layouts/` shadow the theme.

## Deployment

`main` pushes trigger `.github/workflows/hugo.yml`, which builds with Hugo extended + Dart Sass Embedded and publishes `./public` to GitHub Pages. There is no preview environment; staging happens via local `hugo server`.
