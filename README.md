# yujia21.github.io

Personal portfolio and blog built with [Hugo](https://gohugo.io/) using the [hello-friend-ng](https://github.com/rhazdon/hugo-theme-hello-friend-ng) theme.

## Prerequisites

- [Hugo](https://gohugo.io/installation/) (extended version recommended)
- Git

## Getting Started

Clone the repo with submodules:

```bash
git clone --recurse-submodules https://github.com/yujia21/yujia21.github.io.git
cd yujia21.github.io
```

If you already cloned without submodules:

```bash
git submodule update --init --recursive
```

## Development

Start the local development server:

```bash
hugo server
```

The site will be available at `http://localhost:1313`.

## Build

Generate the static site into the `public/` directory:

```bash
hugo
```

## Project Structure

```
content/     # Markdown content (posts, portfolio, about)
layouts/     # Custom layout overrides
static/      # Static assets (images, files)
themes/      # Hugo themes (git submodule)
hugo.toml    # Site configuration
```
