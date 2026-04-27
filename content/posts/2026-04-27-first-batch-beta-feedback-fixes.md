---
title: "First batch of beta feedback fixes"
date: 2026-04-27
draft: false
tags:
  [
    "beta",
    "bugfix",
    "ui",
    "audio",
    "oboe",
    "compose",
    "ai-assisted",
    "claude",
    "gemini",
  ]
summary: 'Eight items from the first round of beta feedback, in one focused day. A subtle Compose race condition that broke Feel-the-Beat, a rename of "Special Entry" to "Break", per-playlist rename via a 3-dot overflow menu, an in-library New Song affordance, a system-nav clipping fix, a restructured top-bar menu, and the first version of an Edit Sound screen with live frequency and tone control plumbed all the way down to the Oboe callback.'
---

The submission is in, the testers are slowly opting in, and the first round of feedback has started arriving. Today was a single focused session: process the list, fix or improve each item, and ship the next beta build.

Eight items in one afternoon. Most were small. One was a real bug. The biggest was the start of the Edit Sound screen — two sliders that change the click in real time.

## How a session like this works

Same workflow as the previous weeks: I tell Claude what each tester reported, Claude investigates the code and writes a focused prompt, I paste the prompt into Gemini Code Assist in Android Studio, Gemini edits the files, I review the diff and test on my device. Claude never touches the files directly in this project. Gemini never sees the rest of the conversation — every prompt has to stand alone.

The split is intentional: Claude is good at reasoning across files and writing precise instructions; Gemini is fast at applying mechanical edits inside the IDE; I'm the only one who can actually run the build on a real phone and notice when something feels wrong. None of the three of us could ship this alone.

## The actual bug

A tester reported: "after I switch off the flash, sometimes Feel-the-Beat stops working — and the flashing won't restart either. If I press Stop and Play, everything works again."

That intermittent quality is the giveaway. The cause turned out to be a shared-`Animatable` race in the stopped-state effects on the main screen. Two coroutines write to the same `breathingAlpha` value: one is the breathing-pulse loop (which also ticks a `softBeatNanos` heartbeat that Feel-the-Beat polls), the other is a tiny cleanup effect that fades the alpha when you toggle the flash off. When the user toggles flash off mid-animation, the cleanup effect's `animateTo` cancels the loop's in-flight `animateTo`, the `CancellationException` propagates up out of the `while(true)`, and the heartbeat loop dies. Feel-the-Beat goes silent. Toggling flash back on doesn't help, because the loop's `LaunchedEffect` is keyed only on `isPlaying` and `bpm` — neither changed. Pressing Stop then Play does change the key, which relaunches the loop.

Claude diagnosed it from the description and the code without me having to reproduce it. The fix was to split the heartbeat (which keeps `softBeatNanos` ticking) from the visual breathing (which animates `breathingAlpha`) into two independent `LaunchedEffect`s. Now the flash toggle can cancel and restart the visual effect without touching the heartbeat. Gemini applied the diff in one pass.

I wouldn't have figured this out on my own. Compose's coroutine cancellation rules are subtle, and the symptom — "sometimes Feel-the-Beat stops working" — points at the vibration code, not at an animation. Claude pointed at the animation.

## The "Special Entry" → "Break" rename

A small piece of UX. "Special Entry" was the placeholder name for a non-song row in a playlist (a pause, a spoken intro, a drum solo, whatever). Testers found it vague.

I picked "Break" over "Cue" or "Interlude" because it's the most musician-natural and most monosyllabically clear. Claude argued briefly for "Cue" (single-syllable, theater-domain, covers more cases) but agreed once I said why "Break" felt right.

The interesting bit: the rename touches a Kotlin `sealed class`, two test files, the migration code, the editor sheet, and a half-dozen call sites. But it does **not** touch the on-disk format. The class is now `PlaylistItem.Break`, but the `@SerialName("special")` annotation stays — so existing tester save files load without any migration. No schema bump. This is the kind of decision that's easy to get wrong; the right answer is to decouple the wire format from the type name precisely because they have different lifecycles.

While we were in the playlist row code anyway, we fixed a wrap bug: a Break with a long label was breaking onto a second line and pushing the row taller. One `maxLines = 1` and `TextOverflow.Ellipsis`, plus a `Modifier.weight(1f)` on the Text so the ellipsis actually triggers.

## Playlist rename via a 3-dot menu

The original switcher had a "Delete" button at the bottom that operated on the _active_ playlist. Subtly bad UX: nothing told you that, and to delete a different playlist you'd have to switch to it first.

Claude proposed three placements for a Rename action; I picked the per-row 3-dot overflow menu, which also fixed the asymmetric Delete. The bottom Delete button is gone. Each row now has a `MoreVert` icon → DropdownMenu with Rename and Delete, both targeting the playlist the menu was opened on. Delete is disabled (not hidden) when there's only one playlist left.

The dropdown is also the natural place to add Export-this-playlist or Duplicate later. The structure is cheap to extend — one new `DropdownMenuItem` per action.

## "+ New Song" inside the Song Library

Previously, you could only add new songs from the main screen's "+" menu — and that always also inserts the new song into the active playlist. The Library was view-only for new entries.

Now there's a "+" icon in the Library's top bar (in both modes — the picker mode where you check songs to add, and the manage mode where you edit/delete). Tapping it opens the existing Song Editor in "new" mode. Saving creates a song in the pool **only** — no auto-insertion into any playlist. In picker mode, the new song appears in the list unchecked; you check it like any other if you want it in the current playlist.

That decision matters: silently adding a created song to the active playlist would be a hidden side effect, and "create song in the pool" is a useful operation in its own right. So I added a new `createSongInPool` method on the ViewModel that creates the song and stops there, separate from the existing `createSongAndAdd` that does both. Claude flagged the freemium gate (30 free songs) and we mirrored the existing check from the main screen "+".

## The system-nav clipping fix

Two testers said the bottom controls were getting too close to or clipped by the Android system navigation area, even on devices where the nav bar should be hidden in immersive contexts. Looking at the code, the cause was simple: the portrait branch's outer Column had `padding(innerPadding)` from the Scaffold, but the Scaffold was configured with `contentWindowInsets = WindowInsets(0, 0, 0, 0)` — meaning no system insets were applied. The landscape branch already had `.navigationBarsPadding()` on its right-hand controls column. Portrait was simply missed.

The user request was nuanced: "reserve space, but make the playlist shorter — don't shrink the buttons." Adding `.navigationBarsPadding()` to the outer portrait Column does exactly that, despite both children using `weight(1f)`. The reason: the controls Column's children (BPM circle, button row, slider) all have hard-coded dp heights. When the outer Column shrinks by the inset, the controls' allocated slot shrinks proportionally, but its fixed-size children don't get smaller — the slack just disappears from the slot's spacing. The LazyColumn, being scrollable, visibly loses rows. The asymmetric effect the user wanted, from a one-line fix.

This is the kind of thing that's obvious in hindsight and was not obvious before. Claude pointed at the right place; the diff is genuinely one modifier.

## Top-bar menu, restructured

The three-dot menu had grown to six items mixing functional actions (Song Library, Export, Import, Dark Mode) with informational links (Licenses, Privacy). I asked Claude whether to split. Claude suggested two sections separated by a `HorizontalDivider`, with Dark Mode classified as functional (it's a behavior toggle, not a legal/info link). I agreed.

Same change added the **Imprint** entry (linking to the impressum page on sessionclick.com) and an **Edit Sound** entry — wired to a stub screen for now, to be filled in next. Both menus had to be updated in lockstep, because the overflow menu code is duplicated across the landscape and portrait Stage Mode `TopAppBar` branches. Extracting a helper composable was tempting but would have needed eight callback parameters; not worth it for two copies.

## Edit Sound — the real feature

This was the largest piece. Two sliders, persisted across launches, plumbed through Compose → ViewModel → SharedPreferences → JNI → C++ Oboe callback.

The two parameters map to existing constants in `AudioEngine.cpp`:

- **Frequency** — the base tone of the synthesized click (currently 880 Hz). Slider range 200–2000 Hz, log-scaled (linear Hz feels uneven to the ear because pitch perception is logarithmic). Default 880.
- **Tone** — the blend ratio between a sine and a square wave. Currently hard-coded as `sine * 0.8 + square * 0.2`. Slider 0–100%, default 20%. Lower is softer and woodier; higher is sharper and buzzier.

Several decisions were non-obvious enough to be worth recording.

**Storage.** I asked: in the session JSON or in SharedPreferences? Claude argued for SharedPreferences: a tester who imports a friend's setlist shouldn't inherit their click preference. That also avoided a schema bump. Stored as two floats keyed in a `sound_prefs` SharedPreferences file inside `AudioEngineViewModel`.

**Live update without races.** The C++ click buffer is a `std::vector<float>` written by the JNI thread and read by the Oboe audio thread. Replacing the buffer mid-callback would be a real-time audio anti-pattern. The fix is a "dirty flag" pattern: `setClickParams` from JNI just stores the new values into atomics and sets a `mClickParamsDirty` flag. The audio callback checks the flag at beat boundaries — _after_ `mClickFrameIndex = 0` — and regenerates the buffer there. The buffer size is constant (30 ms × sample rate), so `resize()` is a no-op for the underlying allocation; no real allocation happens on the audio thread. Claude wrote the C++ diff exactly because Gemini is weaker on Oboe/NDK and small mistakes there are silent and dangerous.

**No preview button this iteration.** When the metronome is stopped, dragging the sliders doesn't make any sound. You have to press Play to hear the change. I considered a dedicated Preview button that fires one click; Claude recommended deferring it. The main screen's Play/Stop button is right there. If testers ask for it, we add it.

**Reset to defaults.** A button at the bottom that snaps both sliders to 880 / 20%. Cheap.

The whole feature took one Gemini prompt — six files, including a careful C++ diff — plus a small follow-up to thread the `AudioEngineViewModel` into the screen. Claude marked the C++ piece for extra-careful review on my end. I read the diff before running it.

## What's next

The next beta build goes out tonight. Testers will get:

- the bug fix for Feel-the-Beat
- the rename to Break
- the new playlist Rename action
- the in-library New Song
- the system-nav fix
- the restructured menu with Imprint and Edit Sound

I'm curious how the Edit Sound extremes feel in the field. The frequency range is wide. The square-wave end might be harsh. Either of those is a tuning decision I'd rather make with field reports than with a guess at my desk.

The next session is the second batch of feedback — whichever items survive the first build. And probably a Preview button.

---

**Time spent today:** ~2h

---

_This blog documents my attempt to build and ship a music app as a solo developer, with AI assistance. The AI does a lot of the work. I try to be specific about what._
