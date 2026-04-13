---
title: "Day 5: UI design, a foreground service, and a hard sync problem"
date: 2026-04-11
draft: false
tags:
  [
    "android",
    "audio",
    "compose",
    "kotlin-multiplatform",
    "jni",
    "animation",
    "ai-assisted",
  ]
summary: "Today had three acts: sketching the full UI concept, building a foreground service, and solving an audio/visual sync problem that turned out to require a C++ fix."
---

A longer session today, and a more varied one. We covered UI design, Android service architecture, Compose animation, and a tricky sync problem that went through several failed attempts before landing on the right solution. Here's what actually happened.

## Act 1: UI concept work

Before writing any code, I wanted to think through the full app design — not just the main screen I'd already sketched, but all the flows that were marked "to be added" in the concept document. (The original mockup and feature spec are published separately as the [SessionClick Concept](/concept/).)

Claude read my mockup and the written concept, then asked a series of clarifying questions:

- Should "special entries" (Pause, Wait for singer, etc.) look visually different from songs? _(Yes — dimmed text, different background, not added to the song pool)_
- Volume control: in-app or device volume? Slider or tap-to-mute? _(Device volume, slider only — tap-to-mute is too dangerous on stage)_
- "Feel the beat" button: works when the screen is off? _(No, app must be visible — I just meant when the metronome isn't playing)_
- Tempo ring when stopped: slow breathe or BPM-synced pulse? _(Subtle pulse)_

From those answers, Claude sketched out the missing screens in text — navigation architecture, a Song Editor, a Song Library (for adding existing songs to a playlist), and a Playlist Switcher as a bottom sheet. Two decisions came up during the sketch that I hadn't thought through: should the FAB (add song button) be visible while the metronome is playing? And when creating a new playlist, where do you land?

Claude recommended hiding the FAB while playing (avoid accidental taps mid-gig) and opening the Song Library immediately after naming a new playlist. Both felt right. Those decisions are now in the concept document alongside the others.

No code was written in this act. Just decisions committed to text.

## Act 2: The foreground service

Claude had already flagged this at the end of yesterday's post: a ViewModel survives rotation, but the audio engine can still be killed under memory pressure. For a stage app, that's not acceptable. The fix is an Android foreground Service — a component that keeps running with a persistent notification, protected from the OS task killer.

Claude explained the architecture before writing anything: the Service owns the audio engine, the ViewModel binds to the Service, the UI observes the ViewModel. Same public interface, different internal wiring.

Claude prepared the Gemini prompt. Gemini wrote three files: `MetronomeService.kt` (the Service with a Binder pattern), an updated `AudioEngineViewModel.kt` (now an `AndroidViewModel` that binds via `ServiceConnection`), and an updated `AndroidManifest.xml` (permissions, service declaration, `foregroundServiceType="mediaPlayback"` for API 34+).

I pasted the generated files back to Claude for review before running anything. Claude caught one real bug: the ViewModel called `startService()` in `init` but never called `stopService()` in `onCleared()`. A started service keeps running until explicitly stopped — so when the app closed, the MetronomeService would linger silently in the background forever. A one-line fix in `onCleared()`.

The other issues were cosmetic: two version checks that were always true at minSdk 28 (dead code, harmless). After fixing the leak and simplifying those, I ran it on the device. Works.

## Act 3: The sync problem

With the service in place, the next task was building the proper UI: the tempo flashlight (a large pulsing circle with the BPM in the centre), separate Start/Stop buttons, and ±1/±5 BPM controls.

Gemini implemented the layout from Claude's prompt without trouble. But the flashlight animation immediately revealed a problem: the visual flash was supposed to coincide with each audio click, but it drifted — slowly at first, then obviously. Stopping and restarting made it worse. Rotating the device reset the visual tempo back to 120 BPM while the audio kept playing.

We went through several attempts to fix this with timer-based animation:

1. **Drift-correcting Kotlin timer**: track `startNanos` and calculate each beat's absolute expected time. Fixes steady-state drift, but `startNanos` was recorded one recomposition frame after the audio started — so there was always a fixed offset, and stop/restart made it accumulate.

2. **Permanent loop, intensity switching**: keep the animation loop running at all times, just change brightness on Start/Stop. Avoids the restart problem but doesn't fix the offset. Rotation still restarted the loop with a fresh `startNanos`.

3. **ViewModel-stored timestamp**: record `startNanos` in the ViewModel (survives rotation) at the exact moment `start()` is called. Closer — but the LaunchedEffect restart still had the beat counter starting from 0, spinning through hundreds of negative wait times before catching up.

At this point I asked Claude directly: is it even possible to sync audio and visual when audio runs in its own stable native thread?

Claude's answer: yes, but not this way. The core problem is that **two independent clocks can never stay aligned by timer math alone**. The Oboe audio callback counts audio frames — rock-solid, immune to OS jitter. The Compose animation loop runs on the main thread, driven by display vsync. No amount of correction on the Kotlin side can keep them in sync, because the source of truth is on the C++ side.

The correct approach: let the audio tell the UI when each beat fires.

## The fix: C++ atomic timestamp

The solution required a small addition to the native audio engine. On every beat, the C++ callback writes the current monotonic timestamp into an `std::atomic<int64_t>`. A new JNI method exposes this value to Kotlin. The Compose animation doesn't run its own timer at all — it polls the C++ timestamp every 8ms, and when it changes, a new click just fired.

Claude prepared two focused Gemini prompts for this. The first covered the C++ and JNI layer only:

- Add `std::atomic<int64_t> mLastBeatNanos{0}` to the `AudioEngine` class
- In `onAudioReady()`, write `clock_gettime(CLOCK_MONOTONIC, ...)` to the atomic at every beat boundary
- Add a `getLastBeatNanos()` getter and the corresponding JNI function
- Thread the value up through `AndroidAudioEngine.kt`, `MetronomeService.kt`, and `AudioEngineViewModel.kt`

Once that built and ran without crashes, Claude prepared the second prompt — the Compose animation:

```kotlin
LaunchedEffect(isPlaying) {
    if (!isPlaying) return@LaunchedEffect
    coroutineScope {
        var lastBeatNanos = 0L
        while (true) {
            val currentBeatNanos = getLastBeatNanos()
            if (currentBeatNanos != lastBeatNanos && currentBeatNanos > 0L) {
                lastBeatNanos = currentBeatNanos
                pulseAlpha.snapTo(1f)
                launch {
                    pulseAlpha.animateTo(0.15f, tween(fadeDuration, FastOutSlowInEasing))
                }
            }
            delay(8L)
        }
    }
}
```

No timer math. No `startNanos`. No drift possible — the flash fires because the audio engine says it fired, not because the Kotlin side guessed when it should.

I tested it: started, stopped, started again, rotated the device. Audio and visual stayed in sync throughout. That's the first time that's been true in this session.

One remaining bug: rotating the device reset the BPM display to 120 while the audio kept playing at whatever BPM I'd set. `bpm` was still in `remember`, which doesn't survive rotation. Moving it to the ViewModel caused a JVM signature clash. Claude fixed this directly.

## What I actually did today

Decided what the app should look like and how it should behave. Tested on a real device repeatedly and noticed when things were wrong. Asked the question ("is sync even possible?") that changed the technical direction. Chose which bugs to fix and which to defer.

What Claude did: all the architecture reasoning, the design questions, catching the service leak, diagnosing the two-clock problem, writing every Gemini prompt, reviewing every generated file, and fixing the JVM clash directly.

What Gemini did: implemented five files from Claude's prompts — the Service, the ViewModel, the manifest, the C++ timestamp addition, and the App.kt animation.

---

**Time spent today:** 4h 21min

---

_This blog documents my attempt to build and ship a music app as a solo developer, with AI assistance. The AI does a lot of the work. I try to be specific about what._
