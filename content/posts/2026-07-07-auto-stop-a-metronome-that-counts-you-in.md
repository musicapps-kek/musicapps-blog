---
title: "Auto-stop: a metronome that counts you in, then bows out"
date: 2026-07-07
draft: false
tags: ["features", "metronome", "kmp", "ai-assisted", "claude", "ux", "testing"]
summary: "SessionClick's second backlog feature: a per-song beat limit that turns the click into an intro count-in — 32 beats, then silence, and the band plays on. Why the countdown lives in the UI instead of the audio engine, how the last beat still rings out cleanly, and two touches that came straight from playing with it on a real phone."
---

The double-/triple-time feature needed open-heart surgery on both audio engines. This
one — the next spec off the backlog — barely touched them at all. Sometimes the right
place for a feature isn't where you'd first reach for it, and auto-stop is a good example
of a change that stayed simple by staying out of the engine.

## The feature: a click with a last beat

Plenty of songs start with a counted intro — four bars of click, then the band comes in
and the metronome would only get in the way. The usual move is to start the click, count
in your head, and stab the stop button at the right moment. It works, but it's one more
thing to get wrong on stage.

SessionClick now does the counting for you. Each song gets an **auto-stop** value: the
number of beats the click should play before it stops itself. Set it to 32 on an intro
tune (that's eight bars in 4/4), press play, and after the thirty-second beat the click
falls silent on its own. Zero means "never stop" — the default, so nothing changes for
songs you don't set it on. The value lives in the song editor, and while the click runs a
small countdown sits right next to the flash circle, ticking down so you can see how many
beats are left.

Press play again and it starts over from the full count. No auto-advance to the next song,
no cleverness — it just gives you a fixed count-in and then gets out of the way.

## Where the countdown lives — and where it doesn't

The obvious place to count beats is inside the audio engine, right where the beats are
made. That was the right call for double-time, where the _sound itself_ had to change. But
auto-stop doesn't change a single click — it just needs to know when the Nth beat has
sounded and stop. And both platforms already publish exactly that.

On Android the audio engine hands the UI a timestamp for every beat, accurate to when the
sound actually hits the speaker; the screen polls it to keep the flash in sync. On iOS the
flash runs on a wall-clock grid that the audio aligns itself to. So the countdown just
listens to the clock that was already ticking: Android decrements on each beat the poller
reports, iOS on each tick of the grid. The native engines — the C++ and the Swift render
code — didn't change at all. A feature that could have meant careful edits to two audio
callbacks instead became a small amount of UI logic on each side, with the engines none
the wiser.

The one subtlety worth getting right is the _last_ beat. You want beat 32 to ring out
fully, but you never want to hear the start of beat 33. So the stop fires a beat-interval
after the final beat, minus a small margin that accounts for the engines rendering a little
audio ahead of the speaker. The final click plays in full — including its softer
subdivision ticks if double-time is on — and the next one never escapes.

Which brings up a nice interaction: the limit counts **main beats only**. Turn on triple
time and each beat becomes three clicks, but 32 still means 32 beats — eight bars — not 32
of the little subdivision ticks. The two features compose the way a musician would expect
them to, without either one having to know about the other.

## Schema v4: the sacred file's second field

Saving a per-song value means touching `session.json` again — the file that holds every
user's songs, and the one part of the project treated as sacred. This is the second field
added since the pool migration, and by now the recipe is dull in the best way: a new field
with a sensible default, a decoder that sanitises anything out of range before it reaches
the app, and tests pinning that old files load untouched, round-trips preserve the value,
and garbage collapses to "off." Old app versions reading a v4 file just ignore the field.
A schema change that used to be a source of dread is now a checklist.

## Two things playing with it taught me

The spec was answered and the feature worked on the first pass. Then I actually used it,
and two rough edges showed up that no amount of desk-thinking would have.

The first: an intro count-in is often the moment you _most_ want to keep the beat going a
little longer — the singer isn't ready, the count needs one more bar. Stopping and
restarting to get there is exactly the fumble the feature was meant to remove. So the
countdown got an escape hatch: **double-tap it and auto-stop suspends** — the number turns
to an infinity sign and the click keeps running for as long as you like. Double-tap again,
or just reselect the song, and it re-arms. (Getting that double-tap to register reliably on
iOS took a small platform-specific detour, but that's a story for the commit log.)

The second was quieter and, to me, the nicer one. When the click auto-stops mid-count, the
flash circle keeps breathing the beat — same tempo, no pause — so you can _keep following
the pulse with your eyes_ after the sound is gone. That's the whole point of a stage
metronome: the count-in doesn't really end at beat 32, it just goes silent, and the visual
carries you into the first bar. On iOS the flash already runs on its own clock, so it never
skipped. On Android the standby pulse had to be taught to pick up exactly where the last
audible beat left off, instead of restarting its own rhythm. A small fix, but it's the
difference between a metronome that stops and one that hands you off.

## Two down

That's the second of the curated backlog specs shipped, and the pattern from last week held
again: spec with its questions answered up front, one bounded implementation session, every
gate green, then a real-device pass that earned its keep. The engines stayed untouched, the
sacred file gained a field without anyone noticing, and the click learned when to stop
talking — which, for a metronome, is its own kind of feature.

---

**Time spent:** ~2.5h across a day

---
