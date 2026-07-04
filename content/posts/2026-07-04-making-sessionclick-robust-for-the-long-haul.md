---
title: "Shipping was the easy part — making SessionClick robust for the long haul"
date: 2026-07-04
draft: false
tags:
  [
    "process",
    "ai-assisted",
    "claude",
    "documentation",
    "testing",
    "ci",
    "detekt",
    "kmp",
    "agents-md",
  ]
summary: "SessionClick is live on both stores, and I have a feature list itching to be built. Before touching it, I'm spending a few sessions on something less glamorous: documentation that survives months-long pauses, tests that pin the invariants, CI, a lint gate — an agentic development loop that lets me (and my AI tools) pick the project up cold."
---

SessionClick is live on Google Play and the App Store, and I have a feature list waiting — double-time subdivisions, auto-stop after N beats, a "next song" pedal action. The temptation to start building is real.

I'm not starting. Here's why.

## The actual risk isn't bugs — it's the pause

This is a side project on 30–60 minutes a day, and the honest pattern is: bursts of development separated by pauses of weeks, sometimes months. Life, gigs, day job. Every time I come back, I pay a re-entry tax: _Where was I? How do I release? Which decisions were deliberate and which were accidents?_

And there's a second reader with the same problem. I build this with AI agents — Claude Code for planning and multi-file work, Gemini in Android Studio for in-IDE help. An agent starting a session has even less memory than I do after three months. Whatever gets me back into the project fast is exactly what makes an agent productive and safe.

The kicker: when I audited the repo after launch, the context files were actively lying. `CLAUDE.md` still declared the project to be in _"Phase 1: Audio Prototype — audio engine decision not yet finalized"_ — while the finished app sat on two stores. The README was the untouched KMP-wizard boilerplate, cheerfully explaining a desktop target that doesn't exist. And the docs said the free tier was 30 songs / 3 playlists when the code had said 50 / 5 for a month. Nobody updated the paper; the paper didn't complain.

So: one infrastructure phase, before any new feature. The goal is a loop where every session — mine or an agent's — starts with full context and ends with the project provably green.

## A plan that is itself the re-entry document

Everything is driven by one file in the repo: `docs/DEVELOPMENT_PLAN.md`. It contains the strategic decisions (marked _"don't re-litigate"_), the work broken into 30–60 minute tasks with checkboxes, a **Current status** block refreshed every session, and a session log — one line per session, newest at the bottom.

The re-entry path after a break is now three hops: repo README → Current status → next unchecked task. Any task can be handed to an agent verbatim: _"Do task 3.2 from docs/DEVELOPMENT_PLAN.md."_ That sentence is a complete, bounded work order.

A few decisions got locked up front so that no future session (or agent) reopens them:

- **Release automation = local fastlane lanes**, not CI deploys. Signing stays on my Mac.
- **Usage reporting = store APIs only.** No analytics SDK in the app, no privacy-label changes.
- **CI runs Linux only.** The repo is private, and GitHub bills macOS runners at 10× the free-tier minutes. iOS checks run locally.
- **One `AGENTS.md`** as the single agent context file.

## One context file to rule them all

I had been maintaining `CLAUDE.md` and `GEMINI.md` in parallel — same content, two files, guaranteed drift. It turns out Gemini in Android Studio now natively reads the tool-agnostic [`AGENTS.md`](https://developer.android.com/studio/gemini/agent-files) standard. Claude Code doesn't yet, but its import syntax makes the stub trivial:

```markdown
# SessionClick — Claude Code Context

All agent context and rules live in the tool-agnostic master file:
@AGENTS.md
```

`GEMINI.md` is deleted. `AGENTS.md` now holds the project context plus two sections that do the heavy lifting:

**Rules of engagement** — adapted from a guardrails checklist I'd researched earlier: one bounded task per session; editing more files than expected is a stop-and-ask signal; no new dependencies or patterns without asking; hands off signing, versions, permissions, and billing config. And one invariant in bold: **on-disk JSON compatibility is sacred** — existing user data must always load.

**Definition of Done** — a task is done when tests and lint pass, new logic has tests, docs were touched if behavior changed, the status block is refreshed, and anything skipped or untested is reported honestly. Hardware-sensitive areas (audio timing, Bluetooth pedals, billing) always get a real-device check by a human before release.

## Writing down what only the code knew

Two new docs, both condensed from the code rather than from memory:

`docs/ARCHITECTURE.md` — module map, data flow, persistence, and a detailed audio-engine section. Writing it surfaced knowledge that existed _only_ in code comments — for instance, that Android and iOS deliberately sync audio and visuals in **opposite directions** (Android's flash follows DAC-accurate audio timestamps; iOS audio aligns itself to the visual wall-clock grid so a tempo change doesn't reset the running pulse). If a future refactor "simplifies" that away, audio and flash drift apart permanently. Now it's documented with a preserve-this warning.

`docs/DECISIONS.md` — a decision log, one dated paragraph each, append-only. It backfills fourteen decisions, including the trap-shaped ones: the playlist item type renamed `Special` → `Break` in code but kept as `"special"` on disk so tester data survives; the App Store background-audio rejection that was a false flag and how it was defended. The log exists so nobody — including future me — "cleans up" something that was load-bearing.

## Tests that pin the invariants

The shared business logic had 27 tests, mostly around playlist state arithmetic. A coverage inventory turned the gaps into 16 named tests, all now implemented — 43 total, running on both the JVM and the iOS simulator target.

The theme is pinning, not coverage percentage. Examples:

- **Corrupt `session.json` → app falls back to the seed instead of crashing.** That fallback path was the app's crash-proofing story, and it had zero tests.
- **`Break` serializes as `"special"` — pinned in both directions** at raw-JSON level. If anyone touches that `@SerialName`, two tests now fail with a clear message instead of users losing their setlists.
- **`schemaVersion` is always written explicitly** — a regression guard for a real bug from April, where kotlinx.serialization silently omitted the field because it matched its default value.
- A new seed-integrity test: every seeded song reference resolves, IDs are unique, and **the seed stays within the free-tier limits**. That last assertion exists because the seed once shipped exactly as many playlists as the free tier allowed — a fresh install couldn't create a playlist without deleting one. (That's why the limits went from 30/3 to 50/5 in June.)

The Android side then got its own unit tests, and the approach — extract the pure logic, test the extraction — earned its keep immediately. The billing unlock condition (_purchase contains `premium_unlock` and is in state PURCHASED_) existed twice in `BillingManager`; it's now one function with seven tests, including the commercially important one: **a pending purchase must never unlock**. And pulling the WAV decoder's PCM math into a testable function surfaced a latent crash — the stereo-downmix loop indexed one sample past the end of odd-length buffers. The fix came free with the extraction, and a test pins it.

Just as valuable is what deliberately did _not_ get tested. The plan confidently listed "BPM clamping" tests for the audio ViewModel; the actual class contains no clamping and no logic at all — it's service-binding glue. Testing it would have meant adding Robolectric for zero real assertions, so the plan got corrected instead. Plans written before reading the code are hypotheses, not specs.

## CI and a lint gate that doesn't nag

A single GitHub Actions workflow now runs on every push: detekt, the shared tests, and a full Android debug build including the C++ Oboe engine. Linux only, per the cost decision above; caching via `setup-gradle`; failed runs upload their test reports as artifacts.

The detekt setup follows one philosophy: **catch real defects, don't fight the IDE.** `MagicNumber` is off (UI code is legitimate literals all the way down), wildcard imports are allowed (Android Studio auto-creates them), and `@Composable` functions are exempt from naming and length rules. The initial run found 103 issues. Twelve were fixed properly, one was genuinely dead code, and 42 pre-existing findings went into a **frozen baseline** — new code must pass clean, and the agent rules explicitly forbid "fixing" a finding by adding it to the baseline.

The baseline is also a confession in XML form: it records that one Compose function has a cyclomatic complexity of **85** against a threshold of 15. That's the main-screen composable, and its re-split is on the backlog — but now it's _measured_ debt instead of a vague bad feeling.

The same checks run locally as a **pre-push hook** — a versioned `.githooks/` directory activated with `git config core.hooksPath .githooks`, so it survives fresh clones. Detekt plus both JVM test suites take a few seconds warm; a push that CI would reject mostly can't leave the machine in the first place.

## Words, and a funnel for ideas

Two small documents rounded out the phase. A sixteen-row **glossary**, because the project's vocabulary has traps that will bite any newcomer, human or agent: the thing called `Break` in code is stored as `"special"` on disk for backward compatibility, and "Flashlight" means the visual beat pulse — not the camera LED.

And a **feature backlog** with a deliberate funnel. Raw ideas get captured in Obsidian, which syncs to my phone — where ideas actually happen, usually at a rehearsal. Only when an idea graduates to "I want to build this" does it move into the repo's `BACKLOG.md` as a mini-spec: description, constraints, and explicitly listed _open design questions_. Agents implement only from curated specs whose questions are answered. The feature list I'm itching to build now lives there, with its open questions staring back at me.

## What's still ahead

The remaining workstreams, in order: fastlane lanes so a release is one command instead of a click-odyssey through two consoles; a small script that pulls installs, purchases, crashes, and ratings from the Play and App Store Connect APIs into one monthly report; and a handful of UI smoke tests on both platforms.

Then — and only then — the feature list.

## The lesson so far

Working through this phase with Claude Code has sharpened one belief: an AI agent is a fast junior developer with weak judgment at the edges. The answer isn't less AI — it's the same things that would make a human team safe. Written context that's actually true. Decisions with their _why_ attached. Tests that pin what must never change. A gate that keeps main green. And a status file that answers "where was I?" in one screen.

The docs lied to me within a month of shipping. The code never did. The whole point of this phase is to close that gap — and keep it closed automatically.
