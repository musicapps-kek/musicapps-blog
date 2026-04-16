---
title: "Day 8: Docs and swipe and drag-and-drop"
date: 2026-04-15
draft: false
tags:
  [
    "android",
    "documentation",
    "oboe",
    "architecture",
    "knowledge-base",
    "compose",
    "ui",
    "ai-assisted",
  ]
summary: "A split session: two new knowledge base articles in the morning, then a long evening of wiring up swipe gestures and drag-to-reorder — including a detour through a custom implementation that never fully worked, ending with the right library call."
---

No new features today, but the kind of session that tends to pay off later: writing proper documentation while the code is still fresh.

## First Part: Two new knowledge base articles

The [MusicApps knowledge base](https://kb.musicapps.eu) got two new articles today, both covering things that took real thought to get right during the implementation phase.

**[Oboe Audio Framework](https://kb.musicapps.eu/android/oboe-audio-framework/)** — what Oboe is, why it matters for low-latency audio on Android, and how SessionClick uses it for sample-accurate metronome timing.

**[SessionClick App Architecture](https://kb.musicapps.eu/android/sessionclick-architecture/)** — the four-layer stack (Compose → ViewModel → Foreground Service → Native audio), with a focus on what survives screen rotation and why timing stays constant.

## Diagrams

Both articles have Mermaid diagrams throughout — flowcharts, sequence diagrams, a Gantt chart comparing timer-based vs frame-counting timing. Claude added these when writing the articles; I've since made it a standing preference for all future KB content.

## Second Part: Swipe gestures and the drag-to-reorder saga

After the documentation work, I switched back to the app. Three playlist gestures were still unchecked: swipe left to delete, swipe right to open the song editor, and long-press drag to reorder.

### Swipe gestures

Claude prepared a Gemini prompt. Gemini implemented the horizontal swipe using `AnchoredDraggableState` — each list item has a background layer with edit (blue) and delete (red) action buttons, and the foreground content slides horizontally to reveal them. The item also snaps back to center if the swipe doesn't complete. Swipe left past the threshold deletes the item immediately; swipe right is wired up as a placeholder for the Song Editor screen, which doesn't exist yet.

Before moving on, Claude reviewed what Gemini had written and caught two regressions: the `BpmButton` composable had been refactored to positional parameters and the `enabled` argument was silently dropped, meaning the + button no longer disabled at BPM 300 and − no longer disabled at BPM 20. The speaker icon I had previously removed had also crept back in. Both fixed in two lines.

### The drag-to-reorder attempt

The drag-to-reorder was where the evening went long.

Gemini implemented it without a library, using `detectDragGesturesAfterLongPress` and manual offset tracking. The approach: each item tracks its own `dragOffset`; dragging past 60% of the next/previous item's height triggers a live `moveItem` call; a placeholder gap and floating shadow card give visual feedback. For the most part it worked — items in the middle of the list could be reordered smoothly.

The problem was a specific case: dragging an item to position 0 (the top of the list) always cancelled the gesture the moment the item arrived there. You could drag freely from index 5 to index 2 to index 1 without issue, but the last step to index 0 would end the drag — as if the finger had lifted.

Claude diagnosed the root cause: when `moveItem(1, 0)` is called, the first item in the `LazyColumn` changes identity. `LazyColumn` tracks its scroll position relative to the first visible item's key; when that key changes, it performs an internal scroll offset correction that competes with the ongoing pointer input gesture and cancels it. This doesn't happen for any other swap because none of them change the first item.

The attempted fixes, in order:

1. Guard `LaunchedEffect(selectedIndex)` against running during drag — this was calling `animateScrollToItem` on every swap of the selected item, adding to the interference. Fixed, but drag still cancelled.
2. Suppress `animateItem()` on all items during drag — the animated layout shifts of neighbours were a secondary source of disruption. Fixed, but drag still cancelled.
3. Don't call `moveItem(X, 0)` during drag at all — defer the move to index 0 until `onDragEnd`. The card floats freely past the first item; the move is committed when the finger lifts. This fixed the cancellation.
4. But now the insert marker (the blue placeholder gap) stayed at index 1 and never jumped to indicate the top position. Added a `dragTargetIndex` state variable that tracks where the marker should appear independently of the item's actual list position, updated on every drag event.
5. With the marker at index 0, the first item didn't shift down to make room. Expanded the first item's slot height to `64.dp + itemHeight`, restricted the placeholder to the top 64dp, and offset the item content downward.

Each step fixed one thing and revealed the next. By this point the code had accumulated several hundred lines of gesture state, offset math, boundary clamping, and special-case logic for index 0.

### The right answer

I stepped back and asked whether drag-to-reorder in a scrollable Compose list is a solved problem. It is. `sh.calvin.reorderable` is the standard library for this — it handles gesture conflicts, auto-scroll, and item identity across recompositions correctly, because it was built specifically for this.

Claude prepared a Gemini prompt to replace the custom implementation. The result: all the custom drag state variables, the offset tracking, the boundary clamping, the `LaunchedEffect` guards, and the `dragTargetIndex` logic — gone. Replaced with:

```kotlin
val reorderState = rememberReorderableLazyListState(listState, onMove = { from, to ->
    moveItem(from.index, to.index)
})
```

And `ReorderableItem` wrapping each list item, with `.longPressDraggableHandle()` on the content. The horizontal swipe coexists cleanly because it runs on a different axis.

Everything works, including drag to position 0.

The lesson is obvious in hindsight: drag-and-drop in a scrollable list is genuinely hard to get right from scratch. The edge cases around the first and last items, auto-scroll, gesture cancellation on recomposition — these are well-known problems with known solutions. The custom approach was worth attempting once to understand why it's hard. Using the library was the right call.

---

**Time spent today:** ~2h 30min

---

---

_This blog documents my attempt to build and ship a music app as a solo developer, with AI assistance. The AI does a lot of the work. I try to be specific about what._
