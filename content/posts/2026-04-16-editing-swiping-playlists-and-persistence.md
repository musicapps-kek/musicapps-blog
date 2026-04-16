---
title: "Day 9: Editing, swiping, playlists, and a very satisfying number"
date: 2026-04-16
draft: false
tags: ["android", "compose", "ui", "ai-assisted", "architecture", "audio", "kotlin-multiplatform"]
summary: "The longest session so far. Song and special entry editing, a complete redesign of the swipe gestures, multiple playlist support, JSON persistence, and a jitter measurement that confirms the audio engine is production-ready."
---

Today was the longest session since the project started. By the end of it, the app has editable song entries, a gesture system that finally feels right, multiple playlists with persistence, and a confirmed audio engine. A lot to write up.

## Editing songs and special entries

The playlist list had swipe-to-edit wired up since the previous session, but there was nowhere to go — no editor screen existed yet. The first task was building it.

Claude designed a `SongEditorSheet` composable as a `ModalBottomSheet`: three fields (title, subtitle, BPM), pre-populated when editing an existing song, empty for new entries. Save is disabled while the title is blank or BPM is out of range. A Gemini prompt implemented it and wired the swipe-right action to open it.

Then I asked about special entries — the "Pause" and "Speaker introduction" items in the list. These only have a single label field, so they needed their own simpler sheet. Claude prepared a second prompt: `SpecialEntryEditorSheet`, one text field, same pattern. At the same time, the "+" button in the top bar became a dropdown menu with two options: "New Song" and "New Special Entry".

Both sheets worked after one round of Gemini output. I made some adjustments — inserting new items below the currently selected entry rather than at the end, and a few layout tweaks. The edit and add flows are now fully functional.

## Redesigning the swipe gestures

This was the most interesting part of the day, and required the most back-and-forth.

The previous implementation used `AnchoredDraggableState`: swipe to reveal an action button, then tap the button. I noticed this felt clunky — two gestures to do one thing. I described the behaviour I wanted to Claude: *"like the Microsoft To-Do app, or the phone app on my Samsung — you drag the item all the way across and it triggers automatically when you lift your finger past one third. If you drag back, nothing happens."*

That's a fundamentally different interaction model. Claude explained the architecture change needed: replace `AnchoredDraggableState` entirely with an `Animatable<Float>` and `detectHorizontalDragGestures`. Free drag with no hard stop, action on release if past the threshold, spring-back if not.

The visual detail I wanted was also specific: below the threshold, show the icon in the action colour with no background. The moment the drag crosses one third of the item width, a coloured circle appears behind the icon with a quick spring animation. Icon turns white. This signals "if you let go now, the action fires."

Claude wrote the complete replacement implementation as a Gemini prompt. The key pieces:

- `offsetX: Animatable<Float>` tracks the horizontal drag position
- `itemWidth` measured via `onSizeChanged` — threshold is `itemWidth / 3`
- `isPastThreshold` is a derived Boolean; a `LaunchedEffect(isPastThreshold)` animates the circle scale in and out
- `onDragEnd` checks the final offset: past threshold → trigger action; before threshold → `spring()` back to zero
- For delete: item slides off screen (180ms tween), then `snackbarHostState.showSnackbar()` with an UNDO action. If UNDO is tapped, the item and its original position are restored via `SessionViewModel.restoreItem()`
- For edit: `snapTo(0f)` immediately, then open the editor sheet — no delay, the sheet opening is the visual feedback

One subtlety Gemini caught during implementation: the coroutine scope for the snackbar needed to live at the `AppContent` level, not inside the `ReorderableItem` block. When the item is deleted and leaves composition, its local scope is cancelled — which would cancel the snackbar before the user can tap UNDO.

The gesture coexists cleanly with the existing long-press drag-to-reorder: different axes, different trigger conditions.

## UX refinements on the editor

After the first version, the edit sheet opening sequence felt jerky: the item snapped back, then the sheet animated up, then the keyboard appeared and pushed the sheet further. Three separate animations fighting for space.

The fixes, in order:
1. Remove the 150ms snap-back animation for edit — use `snapTo(0f)` instead so the sheet opening is the only animation the eye follows.
2. Add `Modifier.imePadding()` to the sheet content so the keyboard expands the sheet's inner padding rather than repositioning the whole sheet.
3. No auto-focus on any text field. Claude's first suggestion was a 300ms delayed focus to let the sheet settle first. After testing, I decided I prefer no auto-focus at all: the sheet opens, I can see all fields, and I tap the one I want to edit. The keyboard appears when I need it, not before.
4. The "+" button is now disabled while the metronome is playing — a one-liner `enabled = !isPlaying` that prevents accidentally opening a dialog mid-performance.

## SessionViewModel — extracting the data model

Up to this point, all playlist state lived directly in `App.kt`: a `mutableStateListOf` with hardcoded items and all the mutation logic as local functions. Claude flagged this as the right moment to move it out.

The new `SessionViewModel` owns the entire session state: the item list, the selected index, and all mutation operations (`addItem`, `removeItem`, `restoreItem`, `updateItem`, `moveItem`). Crucially, `removeItem` returns both the removed item and its original index, so the UNDO path in the UI has everything it needs without holding on to extra state.

`App.kt` now receives the ViewModel as a parameter and calls its methods. The local mutation functions are gone. The hardcoded data moved into the ViewModel's initialiser as a temporary default — it will be replaced by loaded data once persistence is in place (which came later the same session).

A small model change in `shared/commonMain`: `PlaylistItem.Song` gained a `lastEdited: Long` field. It's not displayed anywhere yet, but it's needed for future sorting of the song pool.

## Playlist Switcher

With the ViewModel in place, adding multiple playlists was straightforward.

A `Playlist` data class in `shared/commonMain` (id, name, list of items). The ViewModel now holds a list of playlists internally, with a live `items` SnapshotStateList for the active one. A `sync()` call at the end of every mutation writes the current items back into the playlist store, keeping counts accurate.

The UI is a `ModalBottomSheet` opened by the playlist name button in the top app bar. It lists all playlists with the active one highlighted and a play marker. Two buttons at the bottom: "New Playlist" (opens an inline `AlertDialog` with a name field) and "Delete" (confirmation dialog, disabled when only one playlist exists). Switching is immediate — the sheet closes and the list updates.

Claude prepared a single Gemini prompt covering the data model update, the ViewModel additions, and the new composable. It worked in one pass.

## JSON persistence

Everything built today would be lost on app kill without persistence. Claude chose `org.json` — built into Android, no Gradle changes needed, and straightforward for a small nested structure.

`SessionViewModel` was changed from `ViewModel` to `AndroidViewModel` to access `filesDir`. A `sync()` call now also triggers an async write to `session.json` on the IO dispatcher via `viewModelScope`. On init, `loadIfExists()` reads the file if it's there, restores all playlists and the active playlist ID, and falls back to the hardcoded defaults on first launch or parse error.

The JSON format is simple: a top-level object with `activePlaylistId` and a `playlists` array. Each playlist has `id`, `name`, and `items`. Each item has a `type` field (`"song"` or `"special"`) followed by its properties.

Only `SessionViewModel.kt` changed. App.kt was untouched.

## Jitter measurement

The last task of the day addressed the one open item from Phase 1: measuring timing consistency.

Claude explained the distinction between start latency (how long until the first click after pressing play — not important) and jitter (variation between successive clicks — the number that actually matters for a metronome). Sub-1ms jitter is professional grade.

Since the C++ engine already writes a beat timestamp to `mLastBeatNanos` on every beat boundary, all the measurement logic could live in Kotlin. A rolling `ArrayDeque<Long>` of the last 32 beat timestamps, updated inside the existing beat-polling loop. For each consecutive pair of timestamps, compute `|actual_interval - target_interval|`. Report average and max deviation as `Float` state that drives a small text display below the BPM circle, visible only while playing.

Results on my device at 120 BPM after one minute of playback:

**ø 0.15 ms — max 0.26 ms**

This is better than most hardware metronomes. The Oboe callback model does what it's designed to do: clicks are placed at exact sample positions in the audio stream, and the hardware clock plays them out at a stable rate. The `clock_gettime` timestamps in the callback reflect the OS scheduling the audio thread, not the actual output timing — which is why the numbers are this good.

Phase 1 is closed.

---

**Time spent today:** ~3h

---

_This blog documents my attempt to build and ship a music app as a solo developer, with AI assistance. The AI does a lot of the work. I try to be specific about what._
