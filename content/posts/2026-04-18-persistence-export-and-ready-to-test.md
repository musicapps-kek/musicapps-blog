---
title: "Day 11: Persistence refactor, export/import, UI polish, and splitting App.kt"
date: 2026-04-18
draft: false
tags:
  [
    "kotlin-multiplatform",
    "architecture",
    "serialization",
    "android",
    "ai-assisted",
    "refactoring",
  ]
summary: "The last Android-only code moved into shared KMP. JSON export and import landed. The app got a proper color schema with a dark/light toggle. Then App.kt — grown to 1400 lines — got split into focused files without touching the layout, after two earlier failed attempts taught us exactly which composables are safe to extract."
---

This morning's session finished the architecture work that yesterday's session started, added user-facing backup features, and ended with the app going into my pocket for a real rehearsal.

## Finishing the iOS readiness refactor

Yesterday's session extracted the domain logic into `shared/commonMain`. One piece was still missing: the JSON persistence layer still used `org.json` — an Android-only library — and lived inside the Android ViewModel.

Claude audited the code, then prepared a Gemini prompt to replace `org.json` with `kotlinx.serialization` (the KMP-native library), introduce a `FileStorage` interface with platform-specific implementations, and move a new `SessionRepository` class into `shared/commonMain`. Gemini implemented it; all existing tests passed.

One small bug emerged in the process: kotlinx.serialization silently omits fields that match their default value. The `schemaVersion` field in the JSON file — the key that tells the loader whether to run the v1→v2 migration — was being dropped from newly written files because its default is `2` and the file format specifies `2`. Gemini caught this and fixed it by setting the value explicitly. The kind of serialization edge case that would have caused a confusing bug later.

## The audit that concluded "don't move it"

The step 3 todo also included an open question: should the audio engine's playback state (`isPlaying`, BPM) move into shared code too?

I asked Claude to audit `AudioEngineViewModel` properly before writing a single line of code. The conclusion was no — and the reasoning was worth capturing. The `AudioEngine` interface already lives in `shared/commonMain`; that's the right abstraction boundary. The ViewModel layer exists to connect that interface to platform lifecycle management, which is fundamentally different on each platform: Android uses a foreground Service with a Binder; iOS will use something closer to Combine or ObservableObject. Sharing the ViewModel class would save a few lines while making the code actively harder to reason about per platform.

The only real finding was a minor one: the BPM range constant `20..300` is duplicated across two files. That goes on the polish list.

Sometimes the right outcome of an architecture audit is "the architecture is fine."

## Export and import in one session block

With `SessionRepository` now cleanly separated from file I/O, adding user-facing export and import was straightforward. Claude designed the approach and prepared the Gemini prompt:

- **Export:** Serialize the current session to a JSON string, write it to the cache directory, share via Android's system share sheet using `FileProvider`. The user picks the destination — Files, Google Drive, email — without the app needing any storage permissions.
- **Import:** `ActivityResultContracts.OpenDocument` lets the user pick a `.json` file. The app parses it using the same `SessionRepository.decode()` function that already handles loading on startup (including the v1 migration path, for free). A confirmation dialog shows the song and playlist count before overwriting anything.

The separation of `encode()` / `decode()` from `save()` / `load()` — done in the morning's refactor — meant the import and export features needed almost no new logic. The infrastructure was already there.

## Visual polish: color schema, dark mode, and sound

Before pocketing the app, I worked through visual polish with Gemini. The app now has a proper green color schema — both a dark and a light variant — with a toggle in the overflow menu to switch between them. The user's preference persists across restarts alongside the playlist data. Font sizes and the header bar were also adjusted, and immersive mode keeps the system UI out of the way on stage.

The click sound was also made louder at this point. The audio engine was producing a correct, frame-accurate click, but the amplitude was conservative. For a stage environment — monitors, drums, ambient noise — it needed to be more assertive.

The deliberate decision: stop visual polish here and test first. For a stage app, what looks good at a desk and what works under dim venue lighting with a crowd nearby are different things. I'd rather spend an hour at a real rehearsal and come back with a specific list than spend an hour now guessing.

## Splitting App.kt before it got worse

With the app UI-stable, there was one piece of technical housekeeping that needed to happen before adding the monetization layer: `App.kt` had grown to roughly 1400 lines. Everything was in one file — the main screen logic, both editor sheets, the playlist switcher, the song library screen, small list item helpers. Readable in isolation, but the kind of file that gets harder to work with every time you add something.

Gemini had already attempted this refactor twice at an earlier point and broken the layout both times. The root cause was that it tried to extract `AppContent` — the composable that owns the landscape/portrait layout split and uses `Modifier.weight()` inside its `Row`/`Column` structure. Moving code out of that context changes how weight constraints are resolved, and the layout collapses.

The second attempt worked, with a different strategy: extract only the composables that have no layout context dependency. A `ModalBottomSheet` or a full-screen `Scaffold` composable doesn't care where it's called from — it fills the window regardless. Those are safe to lift. The inner composables that live inside a weighted layout are not.

Two Gemini prompts, one per batch:

- **Batch 1:** `SongEditorSheet`, `SpecialEntryEditorSheet`, `PlaylistSwitcherSheet`, `SongLibraryScreen` — each into its own file. These are all complete sheet or screen composables. Zero risk of layout change.
- **Batch 2:** `SongListItem`, `SpecialListItem`, `BpmButton` — small helpers into `PlaylistComponents.kt`.

Each prompt was explicit about what not to touch: no signature changes, no modifier changes, no padding changes, nothing in `AppContent`. The project compiled clean on both passes and the UI was identical.

`App.kt` is now ~700 lines — still not small, but it's the genuinely complex core: beat tracking, animations, color schemes, the TopAppBar, the main layout split. That's the code that deserves to be read together. Everything else is now in the file that describes it.

## What's left before beta

The app is functionally complete enough to test. The remaining Phase 2 item is monetization: a freemium gate (song/playlist limits in the free tier) and a one-time Google Play in-app purchase. After that, the polish pass that real-world testing will inform, store assets, and a beta release.

---

_This blog documents my attempt to build and ship a music app as a solo developer, with AI assistance. The AI does a lot of the work. I try to be specific about what._
