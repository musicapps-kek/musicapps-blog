---
title: "Selectable click sounds and a haptic tempo circle"
date: 2026-05-01
draft: false
tags:
  [
    "feature",
    "audio",
    "oboe",
    "haptics",
    "compose",
    "ai-assisted",
    "claude",
    "gemini",
  ]
summary: "Two new features in one session: three pre-recorded WAV click sounds you can pick alongside the existing synth click, and press-and-hold haptic feedback on the main tempo circle. Plus a long debugging detour that turned out to be a lazy-init bug in the native engine."
---

Two features today, plus a small detour into native debugging.

## Selectable click sounds

The Edit Sound screen already had two sliders for the synthesized click. I dropped three WAV files into `assets/sounds/` (metro-1, 2, 3) and asked for a selector at the top of the screen: pick the synth or one of the three samples. When a sample is selected, the synth sliders grey out. Tapping any option plays a one-shot preview. Selection persists across launches.

I had a few decisions to make before Claude could write the prompt:

- **No accent on the WAVs.** The synth click distinguishes downbeats; the WAVs play the exact same sample at the exact same volume on every beat. I only supplied three files, not six, and I didn't want to fake an accent by pitch-shifting. Flat is fine for these.
- **Disabled, not hidden** for the synth sliders when a WAV is active. Cheaper to scan visually, and it tells the reader that the synth state still exists in the background.
- **Oboe, not SoundPool.** Same engine the synth click already uses, so timing stays sample-accurate.

Claude wrote the Gemini prompt; Gemini implemented it across the C++ engine, a new `WavDecoder.kt` using `MediaCodec`, the ViewModel, the service, and the Compose screen.

## The detour

First build: the selector worked, the chips highlighted, but no matter what I picked, the synth click was what played. Gemini's first fix attempt didn't help. Second attempt didn't help.

Claude asked me to run the app with Logcat filtered to the package and paste the lines. The trace was decisive in a way I couldn't have read myself: the WAV decoder logs showed all three files decoded successfully (680, 511, 737 samples), but **zero log lines from the native side** — even though Gemini had explicitly added `LOGE` calls in `loadWav`, `setClickType`, and `updateClickBuffer`. Claude pointed out that the native methods were never being called at all.

The actual cause turned out to be a lazy-init bug in the C++ engine: the global `AudioEngine *engine` pointer was only created when `nativeStart()` fired. But the ViewModel decodes WAVs and pushes them down at service-bind time, well before the user presses Play. Every JNI call before Play hit `if (engine != nullptr)` and was silently dropped. The decoded PCM literally never reached native code.

Gemini's third pass moved engine creation into a `getEngine()` helper that lazily allocates on the first JNI call from any direction. After that the next Logcat run showed the full chain — `nativeLoadWav` → `loadWav: loading type 1, 680 samples` → `successfully loaded WAV buffer 0` — and the click sounds finally changed when I tapped the chips.

Two things only I could do in this loop: run the build on the phone, and paste the Logcat back. Two things only Claude could do: spot that the missing native logs were the diagnostic, and reason about JNI initialization order. Gemini did the actual edits in the IDE. The split worked.

## Haptic feedback on the tempo circle

The "Feel the Beat" button vibrates in time with the configured BPM while held — even when the metronome isn't playing. I wanted the same on the main flashing tempo circle. Press and hold the circle: phone vibrates on every beat. Release: stops. No effect on play/stop state.

Claude told Gemini to find the existing button's haptic code, extract it into a shared `Modifier.beatHapticPress` plus a small `BeatHapticHandler`, and have both the button and the circle feed a single ref-counted "active" state — so holding both at once doesn't double-vibrate. Same high-resolution timing source the visual flash uses, so the haptic ticks land on the audible clicks.

One Gemini pass, two files touched, no follow-ups needed.

## The small stuff

Two more, both Gemini-only with no involvement from Claude or me:

- **Version info menu entry.** Just a string and a dialog.
- **Audio/flash sync.** A small tightening of the timing source the visual flash reads from. I didn't dig into the diff.

## Today's changelog

- New: three selectable WAV click sounds alongside the synth
- New: haptic feedback on the tempo circle (press and hold)
- New: version info menu entry
- Improved: tighter audio/flash sync

---

**Time spent today:** ~2h

---

_This blog documents my attempt to build and ship a music app as a solo developer, with AI assistance. The AI does a lot of the work. I try to be specific about what._
