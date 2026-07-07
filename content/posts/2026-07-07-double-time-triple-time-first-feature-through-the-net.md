---
title: "Double-time, triple-time: the first feature through the new safety net"
date: 2026-07-07
draft: false
tags:
  [
    "features",
    "audio",
    "metronome",
    "kmp",
    "ai-assisted",
    "claude",
    "oboe",
    "avaudioengine",
    "testing",
  ]
summary: "SessionClick's first post-infrastructure feature: per-song ×2/×3 subdivisions for slow ballads and 6/8 grooves — click and flash at double or triple the beat without faking the tempo. The safety net earned its keep, the schema survived its first change, and a real-device test rewrote the BPM limits in ways no emulator could have."
---

The infrastructure phase ended with a promise: the next post would be about an actual
feature. Here it is — the first spec pulled from the curated backlog, and the first
change to go through the complete safety net built over the last week. Both the feature
and the net have stories to tell.

## The feature: subdivisions without faking the tempo

Slow ballads are the hardest songs to play to a click. At 60 BPM there's a full second
of silence between beats — plenty of room to drift. The usual workaround is doubling
the tempo in the metronome and remembering that "120" means 60, which works until you
glance at the display mid-song and doubt yourself.

SessionClick now does this properly: a ×1/×2/×3 selector next to the tempo controls.
With ×2 or ×3 active, the click and the flash fire at double or triple frequency while
the displayed BPM stays the truth. Subdivision clicks are softer (60% volume) and
subdivision flashes dimmer (half intensity), so the main beat stays unmistakable — you
get the subdivision _feel_, not a faster metronome. The setting is per song: set it on
the main screen and the same save button that stores tempo edits picks it up, or set it
in the song editor. Load "Slow Swing" from the setlist and the triplet feel comes with it.

## Why this had to go into the audio engines

The tempting implementation is one line: send `bpm × multiplier` to the engine. It's
also wrong twice. The engine contract caps at 300 BPM — ×3 at 119 BPM would be 357 and
either crash or need an ugly exception. And the beat _grid_ would lie: everything
downstream (the flash timestamps, the visual sync) would believe the song is three
times faster.

So the multiplier went into the render callbacks themselves — the Oboe C++ callback on
Android, the `AVAudioSourceNode` render block on iOS. Both engines schedule clicks by
counting audio frames; now they tick every `framesPerBeat / multiplier` frames, tag
each tick as main beat or subdivision, and apply any tempo or multiplier change only at
a **main-beat boundary**, so the pattern never shifts mid-beat. On iOS there's one
extra care point: the flash runs on a fixed wall-clock grid and the audio deliberately
aligns itself to it (an invariant from the very first audio work). Changing the
multiplier mid-play re-runs that same grid-alignment math, so the new pattern lands
exactly on the next flash instead of drifting beside it.

## Schema v3, or: the sacred file gets its first field

Persisting the multiplier meant touching the one thing the project treats as sacred:
`session.json`, the file holding every user's songs. This was the first schema change
since the big v2 migration, and the first under the new test regime. The v3 "migration"
is deliberately boring — a default value. Old files simply load with ×1 everywhere. The
interesting part is the sanitizing on load: whatever a file claims (a hand-edited
multiplier of 7, a ×3 on a 160-BPM song), the decoder collapses it to something legal
before it ever reaches the app. Tests pin all of it: v2 files load unchanged, round
trips preserve the field, garbage collapses to ×1. The shared test count went from 43
to 55, and the old forward-compatibility test — "an old app can read a newer file" —
now describes reality instead of a hypothesis: a 1.0.x app reads a v3 file fine and
just ignores the new field.

## The net catches things

Some of the week-old infrastructure earned its keep within hours. The agent drove the
emulator, tapped ×2, hit save, and read the resulting `session.json` straight off the
device to prove the whole chain — UI to view model to shared state to disk — wrote
`"timeMultiplier": 2` under `"schemaVersion": 3`. The smoke tests caught nothing,
which is also information: launch, play/stop, add-song and the freemium gate all
survived a change that rewired the audio engines.

And the one thing no test can check found its problems immediately.

## What the real device knew that the emulator didn't

The plan said the multiplier should be available below 120 BPM, for both ×2 and ×3.
On paper, fine. On a real phone with real speakers: ×3 at 119 BPM is 357 clicks per
minute — a buzz, not a subdivision. And the flash was worse: the screen flickering six
times a second reads as a malfunction, not a beat.

One evening with the actual device produced four corrections:

- **×3 now caps at 100 BPM** (×2 keeps 120). The selector only offers what's usable —
  between 100 and 119 it shrinks to ×1/×2.
- **The flash got its own, lower limits**: subdivision flashes above 110 BPM (×2) or
  80 BPM (×3) fall back to main-beat-only — while the _audio_ keeps subdividing to its
  limits. Ears resolve faster rhythms than eyes comfortably watch. One deliberate
  subtlety: ×3 audio in the 80–99 zone flashes plain quarter notes, not ×2 — a
  two-flash pattern against triplet clicks would be visual noise.
- **Landscape mode had lost its play button** — the new selector row pushed it
  off-screen into a scroll. The flash circle now shrinks in landscape; portrait keeps
  its size.
- **The Android standby pulse ignored the multiplier** — the circle breathes the beat
  while stopped, but at ×1 regardless of the setting. iOS got this for free from its
  wall-clock flash; Android's heartbeat needed teaching. Now both platforms preview
  the subdivision feel before you press play.

None of these is guesswork the emulator could have settled. "How fast can a flash
flicker before it's annoying" is a question for a musician holding a phone, and the
answer rewrote the spec. The limits now live as named constants in one shared rule
object — with a test pinning each boundary, including the flash-fallback behavior.

## The bounded-feature loop, complete

This feature ran the whole loop designed during the infrastructure phase for the first
time: spec curated in the backlog with design questions answered up front (the four
remaining calls — thresholds, sub-click volume, editor placement, control style — were
settled in a two-minute Q&A before any code), implementation in one bounded session,
every gate green before handoff, a real-device pass, a refinement session, docs and
decision log updated, shipped section of the backlog gets its first entry.

The loop held. The feature that musicians actually asked for is in, the sacred file
gained a field without breaking anyone, and the two remaining backlog specs — pedal
controls for next-song and auto-stop after N beats — have a proven path to follow.

---

**Time spent:** ~3h across two days

---
