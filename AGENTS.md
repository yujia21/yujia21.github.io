# AGENTS.md

Personal portfolio/blog for Yu Jia Cheong, built with [Hugo](https://gohugo.io/) and the [hello-friend-ng](https://github.com/rhazdon/hugo-theme-hello-friend-ng) theme.

## Essential Commands

```bash
# Local dev server (live-reload at http://localhost:1313)
hugo server

# Production build (outputs to public/)
hugo --minify
```

No build scripts, Makefiles, or package managers — Hugo is the only build tool. Deploy happens automatically via GitHub Actions on push to `main`.

## Architecture

```
content/          # Markdown source (front matter + body)
  about.md
  writing.md
  portfolio/      # Dated portfolio posts (YYYY-MM-DD-slug.md)
layouts/
  shortcodes/
    include.html  # Custom shortcode: embeds pre-built HTML from static/
static/           # Served verbatim; charts live here as self-contained HTML
  assets/img/     # Images referenced in post front matter
  usep/, weather/, row/, resale/  # Pre-built interactive chart HTML
themes/
  hello-friend-ng/  # Git submodule — must be initialized before building
hugo.toml         # Primary site config (canonical)
```

## Content Authoring

### Portfolio post front matter

```yaml
---
author: Yu Jia Cheong
categories: ["energy"]
date: '2019-08-31T00:00:00Z'
image: city-1.jpg          # Filename only, must exist in static/assets/img/
tags:
- singapore
- exploratory data analysis
title: What do Singapore's electricity prices look like?
---
```

- `categories` maps to the `blog` taxonomy (configured in `hugo.toml`)
- `image` is filename-only (not a path) — images live in `static/assets/img/`
- `date` must be ISO 8601 with `Z` suffix as shown

### Embedding interactive charts

Charts are pre-built standalone HTML files (Plotly, Bokeh, etc.) placed in a subdirectory of `static/`. The custom `include` shortcode injects them inline:

```
{{< include "usep/usep_2018.html" >}}
```

The path is relative to `static/`. The shortcode implementation is `layouts/shortcodes/include.html`:
```
{{ readFile (printf "static/%s" (.Get 0)) | safeHTML }}
```

## Gotchas

- **Theme is a git submodule**: After cloning, run `git submodule update --init --recursive`. CI does this automatically (`submodules: recursive`). Without it, the build fails with a missing theme error.

- **Hugo extended required**: The CI workflow uses `extended: true` for the Hugo setup step (SCSS processing). Use the extended version locally too.

- **`archetypes/` is empty**: No project-level archetypes are defined. Write front matter manually following the pattern above or refer to `themes/hello-friend-ng/archetypes/`.

## Deployment

Push to `main` triggers `.github/workflows/deploy.yml`, which builds with `hugo --minify --baseURL <pages-url>` and deploys to GitHub Pages. No manual deploy step is needed.
