# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Start development server with live reload at http://localhost:1313
hugo server

# Build static site to public/
hugo
```

There are no tests and no linter configured for this project.

## Architecture

This is a [Hugo](https://gohugo.io/) static site (v0.157+).

**Layouts** (`layouts/`):
- `_default/baseof.html` — base template (head, nav, container, footer, scripts); nav is hidden on homepage via `.IsHome`
- `index.html` — homepage layout (overrides `main` block with `.Content`)
- `blog/single.html` — individual blog post layout (title + date header, `full-article` body class)
- `blog/list.html` — blog listing page (`/blog/`)
- `blog/archive.html` — posts grouped by year (`/blog/archive/`)
- Partials: `partials/head.html`, `partials/nav.html`, `partials/footer.html`, `partials/scripts.html`

**Content** (`content/`):
- `_index.html` — homepage bio content
- `blog/_index.md` — blog section index
- `blog/archive.md` — archive page (uses `layout: archive`, `url: /blog/archive/`)
- `blog/YYYY-MM-DD-slug.md` — Markdown posts (migrated from `.html.md`)
- `blog/YYYY-MM-DD-slug.html` — HTML posts (migrated from `.html.erb`)

**Key config** (`hugo.toml`):
- Permalink format: `/blog/:year/:month/:day/:slug/` (matches legacy Middleman URLs)
- Date extracted from filename prefix (`YYYY-MM-DD`) when not in frontmatter
- `unsafe = true` Goldmark renderer (needed for raw HTML in `.html` posts)
- Taxonomies disabled (no tag/category pages)

**Static assets** (`static/`):
- Bootstrap 3 (`stylesheets/bootstrap.min.css`)
- Font Awesome (`stylesheets/font-awesome.min.css`)
- Prism.js syntax highlighting (`stylesheets/prism.css`, `javascripts/prism.js`)
- Site-specific: `stylesheets/nerdyc.css`
- Fonts: `fonts/` (Font Awesome + Glyphicon webfonts)

