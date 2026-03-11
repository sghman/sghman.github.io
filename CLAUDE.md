# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Jekyll-based personal tech blog & portfolio (GitHub Pages) using the **Chirpy theme v6.3.1**. Korean-language engineering blog with an automated content pipeline that collects, evaluates (via Claude AI), and publishes technical articles daily. Dark mode only (`theme_mode: dark`).

## Commands

```bash
# Local development
bundle exec jekyll serve          # → http://localhost:4000/

# Build
bundle exec jekyll build

# SCSS linting
npm run test                      # stylelint check
npm run fixlint                   # auto-fix

# JavaScript bundling (Rollup + Babel)
npm run build                     # production (minified)
npm run watch                     # dev watch mode

# HTML validation
bundle exec htmlproofer _site --disable-external=true
```

**Deploy:** Push to `main` → GitHub Actions auto-builds and deploys.

## Architecture

### Content Flow
RSS feeds (22 sources) → Naver/Tavily APIs → Claude AI evaluation → Markdown posts → GitHub Actions deploy

### Key Directories

| Path | Purpose |
|------|---------|
| `_data/projects.yml` | Project list (shared by About + Projects pages) |
| `_data/activity.yml` | Recent activity timeline (keep 5-10 items) |
| `_data/dashboard.json` | **Auto-generated — DO NOT EDIT** |
| `_tabs/*.md` | Top nav pages (about, posts, projects, dashboard) |
| `_layouts/landing.html` | Full-screen dark landing page (used by index.html) |
| `_sass/variables-hook.scss` | Design token overrides (Geist font, sidebar width) |
| `_sass/colors/typography-dark.scss` | Dark mode color definitions |
| `assets/css/jekyll-theme-chirpy.scss` | Custom CSS (~9KB of design refinements) |
| `_posts/YYYY-MM-DD-title.md` | Blog posts (Markdown + YAML front matter) |
| `_javascript/` | Source JS modules (compiled by Rollup to `assets/js/dist/`) |

### Design System

- **Fonts:** Geist + Inter (Google Fonts)
- **Colors:** Black background (#000), dark grays (#0a0a0a–#1a1a1a), accent blue (#8ab4f8), green (#81c995), yellow (#fdd663), red (#f28b82), text (#ededed/#999/#555)
- **Layout:** 240px fixed sidebar + 880px content = 1120px centered block
- **Status badges:** `live` (green) / `dev` (blue) / `plan` (yellow) / `idea` (gray)
- **Framework:** Bootstrap 5 grid, Chart.js for dashboard

### Templating

Liquid templates. Layouts inherit: `compress.html` → `page.html` → specific layouts. Custom includes in `_includes/`. Tab pages are a Jekyll collection (`_tabs/`) sorted by `order` front matter.

## Conventions

- **Commit messages:** Conventional Commits — `fix:`, `feat:`, `chore:`, `perf:`
- **Language:** UI/content in Korean (ko-KR), code comments in English
- **CSS:** Use `/* */` comments in SCSS/inline styles, never `//` in HTML script blocks
- **Prettier:** trailing comma `false`
- **SCSS:** Follows stylelint-config-standard-scss (see `package.json` for rule overrides)

## Data Files

Project entries in `_data/projects.yml`:
```yaml
- name: Project Name
  desc: One-line description
  status: dev          # live | dev | plan | idea
  category: personal   # automation | personal | learning
  tech: [Python, Docker]
```

Activity entries in `_data/activity.yml`:
```yaml
- date: "2026-04-01"
  project: Project Name
  text: Brief description
  color: green         # green | accent | yellow
```
