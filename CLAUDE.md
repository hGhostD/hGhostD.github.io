# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a Hexo static blog site deployed to GitHub Pages. The repo doubles as an Obsidian vault (`.obsidian/` config is tracked). Blog source files are written in Markdown with YAML front matter.

**Live site**: https://hghostd.top

## Commands

All commands run from the `blog/` directory:

```bash
cd blog

# Install dependencies
yarn install

# Start dev server (http://localhost:4000)
npx hexo server

# Create a new post
npx hexo new post "Post Title"

# Create a new draft
npx hexo new draft "Draft Title"

# Generate static files (output to public/)
npx hexo generate

# Deploy to GitHub Pages (generates + pushes to master branch)
npx hexo deploy

# Generate + deploy in one step
npx hexo generate --deploy

# Clean generated files
npx hexo clean
```

## Architecture

### Two-branch deployment

- **`hexo` branch** — source code (Markdown posts, config, themes). This is the working branch.
- **`master` branch** — generated static site (output of `hexo generate`). Deployed automatically via `hexo-deployer-git`.

### Source directory structure (`blog/source/`)

- **_posts/** — Blog posts, organized into subdirectories by category:
  - `Swift/`, `Objective-C/` — Technical posts, each has an `index.md` category page
  - `读书/` — Book notes (use `layout: 读书` in front matter)
  - `日记/` — Diary entries
  - `待续/` — Draft/in-progress posts
- **categories/index.md** — Categories listing page
- **tags/index.md** — Tags listing page
- **img/** — Static images (favicon, etc.)
- **css/custom.css** — Custom CSS overrides for the theme

### Post front matter format

```yaml
---
title: Post Title
date: YYYY-MM-DD HH:mm:ss
tags:
- Tag1
- Tag2
categories:
- CategoryName
---
```

Custom layouts (like `layout: 读书`) can be specified for special post types.

### Theme

Uses **Butterfly** theme (v3.8.4) with configuration in `_config.butterfly.yml`. Key features enabled:
- Code highlighting (mac theme, copy button, language label, word wrap)
- JSON content generation (`hexo-generator-json-content`) for posts metadata
- Word count (`hexo-wordcount`)

### Key plugins

- `hexo-deployer-git` — Deploys generated site to `master` branch
- `hexo-generator-json-content` — Generates `content.json` with post metadata (titles, dates, tags, categories)
- `hexo-wordcount` — Post word count estimates
- `hexo-renderer-marked` — Markdown rendering
- `hexo-renderer-stylus` — CSS preprocessing
- `hexo-renderer-pug` — Pug template support

### Scaffolds (`blog/scaffolds/`)

Template files used by `hexo new`:
- `post.md` — Default post template (title, date, tags)
- `draft.md` — Draft template (title, tags, no date)
- `page.md` — Page template (title, date)

### Obsidian integration

The `.obsidian/` directory at repo root contains Obsidian vault configuration. The Markdown files in `blog/source/` can be edited directly in Obsidian as a vault.
