---
title: "Hands-free start/stop with a Bluetooth foot pedal"
date: 2026-05-07
draft: false
tags:
  [
    "feature",
    "input",
    "bluetooth",
    "accessibility",
    "compose",
    "ai-assisted",
    "claude",
    "gemini",
  ]
summary: "SessionClick now responds to Bluetooth foot pedals. Pedals marketed as page turners present themselves as keyboards, so the app listens for the keys those pedals send and toggles play/stop. One small feature, one short Gemini prompt, one real bug Claude caught in review."
---

If you play live, taking your hands off the instrument to tap a phone screen is annoying. Most musicians who use a tablet for sheet music already own a small Bluetooth foot pedal — the kind sold as a "page turner". I wanted SessionClick to start and stop with one of those.

## What pedals actually send

I asked Perplexity for a developer-level overview of the common pedals (iRig BlueTurn, PageFlip Firefly, AirTurn BT-200, Donner, …). The pattern across all of them: the pedal pairs with the phone as a **Bluetooth HID keyboard** and emits ordinary keystrokes. Which keystrokes depends on the model and the mode it's in — Page Up / Page Down on one, arrow keys on another, Space and Enter on a third.

So the app doesn't need to know anything about Bluetooth at all. It just needs to listen for the keys those pedals are likely to send. For this first version I went with the simplest possible behavior: **either pedal, any of the common keys, toggles play/stop.** Splitting the two pedals into different actions (next song, previous song) is on the backlog for later.

## The collaboration this time

I sent Claude the Perplexity research and said "prepare a Gemini prompt for this". Claude came back with six design questions before writing anything:

- Which keys exactly? (Decided: arrow keys, PageUp/Down, Space, Enter, Numpad Enter — the union of what the common pedals send.)
- Trigger on key down or key up? (Down — that's what feels natural when you stomp.)
- What happens when a text field is focused? (Don't intercept — otherwise typing "Page Down" into a song title would stop the metronome.)
- Capture at the activity level or in Compose? (Activity, more reliable for HID input.)
- Settings toggle to enable/disable? (No — always on, unless a tester complains.)
- Want me to add the "second pedal = next song" idea to the backlog? (Yes please.)

Once I'd answered those, Claude wrote a focused Gemini prompt scoped to two files. Gemini implemented it. It worked on the first build with a real Bluetooth keyboard.

## What only Claude could catch

After it worked, I asked Claude to review the changes. It found one real issue I would have shipped without noticing: the guard that's supposed to skip key handling while the user is typing only checked for the old Android `EditText` widget. Compose text fields don't show up as `EditText`s — Compose manages text-field focus internally. So if you opened the song editor and typed Space into the title, the metronome would have started and the space wouldn't have made it into the field.

The fix was a one-liner: use `onCheckIsTextEditor()` instead of `is EditText`. That's the Android API the system keyboard itself uses to decide whether a focused view accepts text input — and Compose answers it correctly.

Claude also spotted a lint warning about a restricted-API call and explained why it was a false positive (the warning fires on any `dispatchKeyEvent` override regardless of how it's called) plus the standard one-line suppression for it.

Two things only I could do: write the prompt requirements based on actual stage use, and physically pair a Bluetooth keyboard with the phone to confirm it works. Two things only Claude could do: spot the Compose-vs-EditText subtlety in code review, and explain the lint warning rather than just suppress it. Gemini did the actual edits.

## Why this matters more than it sounds

SessionClick's whole premise is that you should be able to use it while your hands are busy. Tap-tempo, large-targets, keep-screen-awake — those were all about the same idea. Foot-pedal start/stop is the same idea taken further: you don't need to touch the phone at all to drive the click. Set the BPM at home, walk on stage, stomp the pedal at the right moment.

It's also the first SessionClick feature where the device on the other end of the Bluetooth radio matters at all, and where the app does no Bluetooth code itself. That's a nice property — we get hardware support for free because of how Android already exposes HID keyboards system-wide.

## Today's changelog

- New: Bluetooth foot pedal / external keyboard support — any of arrow keys, Page Up, Page Down, Space, Enter, or Numpad Enter toggles play/stop. No setup needed; pair the pedal to your phone in Bluetooth settings and it just works.

---

**Time spent today:** ~1h

---

_This blog documents my attempt to build and ship a music app as a solo developer, with AI assistance. The AI does a lot of the work. I try to be specific about what._
