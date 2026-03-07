# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

Gems are vendored in `vendor/`. Run Middleman via Bundler:

```bash
# Start development server with live reload at http://localhost:4567
bundle exec middleman server

# Build static site to build/
bundle exec middleman build
```

There are no tests and no linter configured for this project.

## Architecture

This is a [Middleman 4](https://middlemanapp.com/) static site with the `middleman-blog` extension.

**Layouts** (`source/layouts/`):
- `layout.erb` — default layout (includes nav bar); used by blog posts and most pages
- `homepage.erb` — layout for `index.html` only (no nav bar, just container + footer)
- `article.erb` — layout for individual blog posts (wraps in `layout.erb` via nesting)
- Partials: `_nav.erb`, `_article_list.erb`, `_article_preview.erb`, `_stylesheet_tags.erb`, `_script_tags.erb`

**Blog** (`source/blog/`):
- Posts use filename convention `YYYY-MM-DD-slug.html.md` (Markdown) or `.html.erb` (ERB)
- Blog prefix is `blog/`, accessible at `/blog/`
- `index.html.erb` — recent posts listing; `archive.html.erb` — full archive
- `config.rb` sets `blog.summary_length = nil` (no truncation) and disables day/month/tag pages

**Key config** (`config.rb`):
- Markdown engine: Kramdown with GFM input
- `activate :directory_indexes` — generates `foo/index.html` for clean URLs
- Homepage (`/index.html`) uses the `homepage` layout explicitly
- Livereload active in development only

**Stylesheets** (`source/stylesheets/`):
- Bootstrap 3 (`bootstrap.min.css`)
- Font Awesome (`font-awesome.min.css`)
- Prism.js syntax highlighting (`prism.css`)
- Site-specific: `nerdyc.css` (plain CSS, not compiled)
- `_normalize.scss` is present but the main stylesheet pipeline uses plain CSS, not Sass compilation
