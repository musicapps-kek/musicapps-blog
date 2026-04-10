---
title: "Day 2: Picking a name, choosing an architecture, and making the first commit"
date: 2026-04-08
draft: false
tags: ["planning", "kotlin-multiplatform", "architecture", "naming"]
summary: "SessionClick has a name, a GitHub repo, and a KMP project skeleton. Here's how the decisions fell."
---

A lot of groundwork happened today — no code written yet, but several decisions that will shape the whole project.

## The name

I had four candidates: TockTock, Tickbox, VTronome, and SessionClick.

TockTock was the most memorable — onomatopoeic, fun to say. But it felt too playful for something meant to be a serious stage tool. Tickbox sounded like a to-do app. VTronome had an unexplained "V" prefix that I couldn't build a story around.

SessionClick won. "Click" is how musicians actually talk about a metronome — as in "can I get more click in my monitor?" — and "Session" signals band rehearsals and live gigs rather than solo practice. It describes the product without over-explaining it.

I registered sessionclick.com and sessionclick.de straight away. The umbrella brand stays musicapps.eu; SessionClick is the first product under it.

## Store accounts and identity

A decision I hadn't thought through until today: which email and name to use for the Google Play and Apple developer accounts.

The answer turned out to be straightforward. I'll use **karl@musicapps.eu** for both store accounts, **info@musicapps.eu** as the public support address, and **MusicApps** as the developer/publisher name in both stores. That way future apps appear under the same publisher without any awkward rebranding.

The Apple side is slightly more involved. To use "MusicApps" as the seller name rather than my personal name, I need an Organisation account instead of an Individual one. That requires a D-U-N-S number from Dun & Bradstreet — which is free through Apple's enrollment flow — but first I need a Gewerbeanmeldung. I've parked this as a task for next week; it doesn't block Android development.

## Architecture: Kotlin Multiplatform

Originally I planned separate native codebases — Android first, iOS later. Today I decided to use **Kotlin Multiplatform (KMP)** instead, with a single repository.

I put the question to Claude: given my constraints (solo developer, 30-60 min/day, Android first), does KMP actually help or just add complexity? The reasoning it laid out was convincing: a lot of what this app does is not platform-specific. Song and setlist models, BPM logic, JSON export/import, freemium rules — all of that can live in a `shared/` module and be used by both platforms without duplication. The iOS port becomes "implement the audio engine and build the SwiftUI UI" rather than "rewrite everything". I agreed and made the call.

The audio engine itself stays platform-specific. Timing precision is the whole point of this app, and that means native audio APIs on both sides — Oboe/AAudio on Android, AVAudioEngine or AudioUnit on iOS. Claude flagged the audio engine as the highest-risk component and recommended prototyping it early — before building any UI on top of it.

Project structure looks like this:

```
sessionclick/
├── shared/           ← models, BPM logic, JSON, freemium rules
│   └── src/
│       ├── commonMain/
│       ├── androidMain/
│       └── iosMain/
├── androidApp/       ← Jetpack Compose + Oboe
└── iosApp/           ← SwiftUI + AVAudioEngine
```

## Project setup and first commit

I created the KMP project in Android Studio, pushed it to GitHub under the musicapps-kek account, and watched Gradle download half the internet. I also verified that Xcode can build the iOS target without errors — it needed a few Gradle config tweaks, which Claude diagnosed and applied.

## What's next

- Explore layout ideas with Google Stitch before writing any real UI code
- Audio engine prototype — this is where I find out whether KMP + Oboe is as straightforward as it looks on paper
- Let Claude explain some of the KMP concepts to me in more detail, like how to share code between the shared module and the platform-specific ones, and how to handle dependencies in a KMP project.

---

_This blog documents my attempt to build and ship a music app as a solo developer, with AI assistance. The AI does a lot of the work. I try to be specific about what._
