---
title: "A false rejection and an edit-mode bug hiding in the background"
date: 2026-06-12
draft: false
tags: ["ios", "swiftui", "appstore", "bugfix", "audio", "ai-assisted", "claude"]
summary: "Apple rejected the app under 2.5.4 — 'unable to play audio in the background' — but the code was right all along; the reviewer just never pressed Play. Plus a sneaky sequel to yesterday's edit-mode fix: the invariant broke the moment you backgrounded the app."
---

Two things today: a second App Store rejection that turned out to be a false alarm, and a bug that was the direct sequel to yesterday's fix.

## "Unable to play audio in the background"

The submission came back rejected again — this time Guideline 2.5.4:

> The app declares support for audio in the UIBackgroundModes key in the Info.plist, but we are unable to play any audible content when the app is running in the background.

My first reaction was to go hunting for a bug in the audio session. But there isn't one. I checked all three things that have to be true for background audio:

- `UIBackgroundModes` contains `audio` in Info.plist — yes.
- The audio session uses the `.playback` category — yes, set on every `start()`.
- The render block reports continuous non-silence while running — yes. Between clicks it outputs zeros but still flags the buffer as non-silent, so iOS sees an active audio stream and never suspends it.

So the app *does* play in the background. The catch is obvious once you say it out loud: **a metronome is silent until you press Play.** The reviewer opened the app, went to the Home Screen, heard nothing, and filed the rejection. There was no audio because nobody started the metronome.

Apple's own message spells out the fix: if the feature is real, reply with a screen recording showing the background audio in use. So that's what I did — a recording on a physical device: press the green Play button, swipe to the Home Screen, let the click keep ticking audibly for a few seconds. The important detail I almost missed is *where* the video goes — not just the reply, but the **App Review Information → Notes** field in App Store Connect, so it persists for future submissions too. I added a short note explaining that background audio is a core feature for live musicians: the click has to keep going when the screen locks or when you switch to another app to read your sheet music.

No code changed. The rejection was a misunderstanding, but the burden is on me to make the feature obvious to a reviewer who has thirty seconds and no context.

## The edit-mode bug that came back

Yesterday I fixed edit mode not exiting when you press Play. Today a sneakier version of the same bug showed up: start editing the playlist, press Play (edit mode correctly turns off, the Edit button disables — good), then send the app to the background and come back. Edit mode is on again. While the metronome is playing. Which is exactly what it must never be.

The cause is a SwiftUI ownership detail. `EditButton` keeps its *own* private copy of the edit-mode state. Yesterday's fix only nudged that state to `.inactive` when playback started — it never owned it. Nothing in my code ever sets edit mode back to active; only the button does. So when the app is backgrounded and restored, SwiftUI's state restoration revives the button's last "active" state, and since nothing re-asserts the invariant on the way back to the foreground, edit mode reappears.

The proper fix is to stop letting the button own the state and own it myself:

```swift
@State private var editMode: EditMode = .inactive
```

injected into the environment with `.environment(\.editMode, $editMode)`. Because `@State` lives on the persistent root view, it survives a background/foreground round trip — once playback forces it inactive, it stays inactive. The `EditButton` just drives my binding now; it has no separate state left to restore.

The lesson is one I keep relearning in SwiftUI: when a built-in control manages state implicitly, "nudging" that state is fragile. If you depend on an invariant holding, own the state outright.

## What ships

The edit-mode fix goes into today's build, and the App Store submission goes back in with the video and notes attached. Fingers crossed the second review actually presses Play.

---

**Time spent today:** ~1h

---

_This blog documents my attempt to build and ship a music app as a solo developer, with AI assistance. The AI does a lot of the work. I try to be specific about what._
