---
title: "Day 3: Setting up a knowledge base and organising the project"
date: 2026-04-09
draft: false
tags: ["infrastructure", "zensical", "mkdocs", "knowledge-base", "organisation"]
summary: "No app code written today, but the project infrastructure got a lot more solid — a knowledge base is live, four technical articles are written, and the project notes are finally tidy."
---

Another infrastructure day. Still no app code, but the kind of work that pays off later.

## Project inventory and organisation

Before writing anything new, I took stock of everything that exists: GitHub repos, local paths, live URLs, accounts, domains. It turns out spreading a project across Android Studio, an external drive, iCloud, and GitHub means things get hard to find quickly. I now have a proper inventory document in my Obsidian vault that lists everything in one place.

I also reorganised the existing project notes folder in Obsidian. What had been a flat pile of markdown files — original concept drafts, Perplexity research sessions, account credentials — got sorted into subfolders:

- `Research/` — AI-assisted market analysis and feasibility studies from the early project phase
- `History/` — the original concept prompt and Claude's first response, kept for reference

Nothing deleted, just easier to navigate.

## The knowledge base decision

I've been thinking for a while about where to put technical reference content. The blog is a development diary — wrong format for reference material that I'll want to look up repeatedly. I needed something more like a manual.

The options I considered: extend the blog, use Obsidian Publish, or set up a separate static site. After some research I landed on a **separate site with MkDocs Material** — or so I thought.

While setting it up, I came across a post by the MkDocs Material author explaining that MkDocs is effectively unmaintained and heading towards an incompatible 2.0 rewrite. His team has built a replacement called [Zensical](https://zensical.org) — a new static site generator that's fully compatible with MkDocs content and configuration, but built on a modern foundation with significantly faster build times.

Since I was starting fresh, switching to Zensical from the start was the obvious call. Configuration is nearly identical — changing the theme name from `material` to `zensical` was most of the migration.

The KB is now live at **[kb.musicapps.eu](https://kb.musicapps.eu)**, deployed via GitHub Actions on every push to `main`.

One small hurdle: Python's `pip` isn't directly usable on macOS anymore due to the "externally managed environment" restriction. The solution is `pipx`, which installs Python CLI tools in isolated environments. `brew install pipx && pipx install zensical` — done.

## First KB articles

With the infrastructure in place, I asked Claude to generate four articles to seed the knowledge base. I described the target reader — someone who knows Android development but is new to KMP and iOS — and Claude wrote them. I reviewed and approved each one.

- **What is Kotlin Multiplatform?** — the core concept, what can be shared, what can't, and how it maps to the SessionClick project
- **Gradle in a KMP project** — what Gradle actually does in a multiplatform setup, the module structure, version catalogs, and useful tasks
- **How Kotlin and iOS work together** — the Kotlin/Native compiler, how shared code becomes a framework that Swift imports, type mapping, and the coroutines/async bridging problem
- **Android vs iOS — concepts and terminology** — a side-by-side comparison of screens, navigation, state, ViewModels, lifecycle, storage, and in-app purchases for someone coming from Android

The goal is to have a solid reference for myself as I work through the project, and to create content that could be useful to other developers in the same situation. I didn't write the articles — Claude did — but I chose the topics, defined the audience, and fact-checked the output.

## What's next

The project structure documentation is still on the list — the simplified diagram in my notes doesn't match the real folder layout that Android Studio generated. That needs fixing before I start writing real code, so I don't confuse myself later.

After that: audio engine prototype. Everything else is just scaffolding.

---

**Time spent today:** 1h 12min

---

_This blog documents my attempt to build and ship a music app as a solo developer, with AI assistance. The AI does a lot of the work. I try to be specific about what._
