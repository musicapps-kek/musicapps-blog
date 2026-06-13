# MusicApps Dev Blog

An engineer's diary on building music apps for Android and iOS — with heavy AI assistance, a tight budget, and a healthy dose of scepticism.

Published at: <https://blog.musicapps.eu>

## Purpose

This blog documents the full development process of solo-built music apps, covering:

- Technical decisions and architecture choices
- AI-assisted development experiments
- Failures and lessons learned
- Progress updates from concept to shipped product

## Technology

| Component             | Details                                                    |
| --------------------- | ---------------------------------------------------------- |
| Static site generator | [Hugo](https://gohugo.io/) v0.160.0 (extended)             |
| Theme                 | [PaperMod](https://github.com/adityatelange/hugo-PaperMod) |
| Hosting               | Custom domain via CNAME (`blog.musicapps.eu`)              |
| Content format        | Markdown                                                   |

The theme is included as a **git submodule** under `themes/PaperMod/` — a fresh clone must initialize it (see Setup), or Hugo serves "Page Not Found".

## Setup

Install the **extended** edition of Hugo (the standard build can't compile the theme's SCSS):

```bash
# macOS (Homebrew)
brew install hugo

# Verify — output must say "extended"
hugo version
# e.g. hugo v0.160.0+extended darwin/arm64 ...
```

Other platforms and install methods: <https://gohugo.io/installation/>.

Then pull in the PaperMod theme (tracked as a git submodule):

```bash
git submodule update --init --recursive
```

Without this, `themes/PaperMod/` is empty and `hugo server` returns "Page Not Found".

## Local Preview

```bash
# Clone the repository and navigate into it
cd musicapps-blog

# Start the local development server with live reload
hugo server

# The site is then available at:
# http://localhost:1313
```

To include draft posts in the preview:

```bash
hugo server -D
```

## Build

To generate the static site into the `public/` directory:

```bash
hugo
```

## Project Structure

```
content/        # Markdown content (posts, about, concept, resources)
layouts/        # Custom layout overrides
static/         # Static assets (images, CNAME)
themes/         # PaperMod theme
hugo.toml       # Site configuration
public/         # Generated output (do not edit manually)
```

## Author

Karl-Ernst Kiel — <karl@musicapps.eu>  
GitHub: <https://github.com/musicapps-kek>

---
