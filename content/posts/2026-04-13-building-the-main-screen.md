---
title: "Day 7: Building the main screen — controls, playlist, and a lot of iteration"
date: 2026-04-13
draft: false
tags: ["android", "compose", "ui", "ai-assisted", "kotlin-multiplatform"]
summary: "The main screen finally looks like a real app. A long session covering the last metronome controls, the playlist list, a responsive layout, and several bugs that only surface on a real device."
---

Today was the longest coding session so far. By the end of it, the main screen has a working playlist, all the metronome controls I planned, and a layout that adapts between portrait and landscape. It also took a lot of back-and-forth to get there.

## The three missing controls

When I opened the project this morning, the main screen still had three unchecked items: the "Feel the Beat" haptic button, a volume slider, and keeping the screen awake. Not glamorous, but necessary before moving on.

Claude prepared a single Gemini prompt covering all three. The implementations:

- **Keep screen awake** — one line in `MainActivity.onCreate`: `window.addFlags(FLAG_KEEP_SCREEN_ON)`. Genuinely the simplest fix.
- **Volume slider** — reads and writes `AudioManager.STREAM_MUSIC` device volume via a `Slider` composable. No special permission needed. The slider initialises at the current device volume and updates it live.
- **"Feel the Beat"** — the most interesting one technically. The button uses `MutableInteractionSource` and `collectIsPressedAsState()` to detect press-and-hold without relying on `onClick`. While held and playing, it polls the same C++ beat timestamp used by the visual flash, so the vibration is frame-accurate. While held and not playing, it uses a software timer at the current BPM.

Gemini implemented all three with some adjustments from the generated prompt. I tested on device, everything worked.

## Adding the playlist

The next prompt was a bigger structural change: adding the song list to the main screen. Claude restructured `App.kt` from a flat column of controls into a proper `Scaffold` layout — `TopAppBar`, a `LazyColumn` for the playlist (weight 1f, takes all available space), and the controls section below it.

The list uses a sealed class `PlaylistItem` with two subtypes: `Song` (title, subtitle, BPM) and `Special` (label only, visually dimmed). Tapping a song highlights it, loads its BPM into the metronome, and scrolls the list so the selected item sits second from the top — the design I settled on during the concept phase. The scroll uses `animateScrollToItem(max(0, selectedIndex - 1))`.

For now the data is hardcoded. The real data model and persistence layer comes later.

## Iterative UI refinement — a lot of back-and-forth

What followed was about an hour of incremental adjustments. I would describe what I wanted, Claude would either edit the file directly or write a new Gemini prompt, I would test on device, and report back. The changes:

- Moved the FAB "+" button into the `TopAppBar` as an `IconButton`
- Made the volume slider **vertical**, placed to the left of the tempo circle. This required a layout modifier trick: the slider is actually a standard horizontal `Slider` composable, rotated -90° via `graphicsLayer`, with a custom `layout {}` modifier that swaps the width and height constraints so the touch area rotates with the visual.
- Replaced the four BPM buttons (-5, -1, +1, +5) with two round buttons: **+** and **−**. A single tap changes BPM by 1. Holding triggers a repeat every 400ms at ±5. This is implemented with `collectIsPressedAsState()` + a `LaunchedEffect` that watches the pressed state — no custom gesture detector needed.
- Added a **save tempo button** (disk icon) that appears below the +/− buttons when the current BPM differs from the saved song BPM. Space is always reserved for it so the layout doesn't shift. It's hidden via `Modifier.alpha(0f)` rather than removed from composition, with `enabled = false` when not needed.
- Combined Start and Stop into one toggle button, large, bottom-right.
- Made the "Feel the Beat" button the same width as Start/Stop, with a fingerprint icon.
- Added a small flash icon between the two bottom buttons to toggle the stopped-state pulse. It disappears while playing.

Most of these were direct edits by Claude to `App.kt`. A few needed Gemini prompts for larger structural changes.

## Two bugs, found on device

Two things only became apparent when testing on the actual phone.

**Double vibration on Feel the Beat while playing.** The first time I pressed the button while the metronome was running, I felt two pulses: one immediately, then the next real beat. Claude diagnosed it in one look: the polling loop initialised `lastBeatNanos = 0L`, so the first iteration always found a "new" beat — the most recent one from the C++ engine, which had been running for minutes. Fix: initialise with the current engine timestamp instead of zero.

**Pulse and Feel the Beat out of sync when stopped.** The visual pulse and the haptic feedback were both running at the correct BPM, but they started independently and drifted. The solution was the same pattern already in use for the playing state: a single software beat clock, updated by the visual loop, polled by the vibration loop. Both now read the same timestamp, so they're always in phase — regardless of when the button was pressed.

## Responsive layout

The last task was splitting the screen: playlist on top, controls on bottom in portrait; playlist left, controls right in landscape.

Claude used `LocalConfiguration.current.orientation` to detect orientation, then chose between a `Column` (portrait) and a `Row` (landscape) as the outer container. To avoid duplicating 150 lines of controls code across both branches, the controls section is stored as a `@Composable () -> Unit` lambda variable that captures all the surrounding state. The list items are stored as a `LazyListScope.() -> Unit` lambda. Both are called in whichever container is active.

In portrait, the controls column uses `Arrangement.Center` to sit vertically in its half. In landscape, it uses `verticalScroll` in case the controls overflow the screen height.

## Where things stand

The main screen is functional. It has a working playlist (hardcoded data), all the controls I designed, haptic feedback, volume control, and a responsive layout. The next milestone is connecting it to real data: the Song and Setlist models, editable entries, persistence.

---

**Time spent today:** ~1h 45min

---

_This blog documents my attempt to build and ship a music app as a solo developer, with AI assistance. The AI does a lot of the work. I try to be specific about what._
