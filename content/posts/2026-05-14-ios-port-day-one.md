---
title: "The iOS port, day one"
date: 2026-05-14
draft: false
tags:
  [
    "ios",
    "swiftui",
    "kotlin-multiplatform",
    "kmp",
    "avaudioengine",
    "ai-assisted",
    "claude",
  ]
summary: "First day of the SwiftUI port. Most of Phase 5 cleared in one sitting — because the shared KMP module already handled the business logic, only the platform-specific layers (audio engine, file storage, UI) needed iOS implementations. The workflow also shifted: Claude wrote the Swift directly instead of routing through an in-IDE AI."
---

Today was the day I finally started the iOS side of SessionClick. By the time I stopped, the iOS build had real-time audio click via AVAudioEngine, the tempo flashlight, a working playlist with create / edit / reorder / delete, the song library picker, persistence across launches, dark mode, the sound editor with three WAV samples and a synthesized click, volume control, keep-screen-awake, Feel the Beat haptics, Save Tempo, the flash toggle, and the app icon. Roughly the first thirteen sessions from my planned Phase 5, in one sitting.

That's faster than I expected. Two things made it possible.

## The shared layer paid off

The whole reason I built SessionClick as Kotlin Multiplatform was for this moment. Today proved it. The pieces I did **not** have to rewrite for iOS:

- `SessionState` (in-memory model + mutations)
- `Song`, `Playlist`, `PlaylistItem` (the data classes)
- `SessionRepository` (JSON encode/decode with `kotlinx.serialization`)
- `SessionSeed` (the demo data — jazz, classical, pop playlists)
- `Migration`, `FreemiumConfig`, `FreemiumGate` (rules)
- The 14 `commonTest` unit tests that already validate `SessionState`

The pieces I **did** have to write per-platform:

- An `AudioEngine` implementation using `AVAudioEngine` + `AVAudioSourceNode` (instead of Oboe / JNI / C++)
- An `IosFileStorage` (NSFileManager → Documents directory) implementing the shared `FileStorage` interface
- All the SwiftUI screens

The iOS audio implementation is actually shorter than the Android one. The C++ / JNI plumbing on Android exists because Oboe is C-based; AVAudioEngine is Swift-native, so the Kotlin shared module just exposes the `AudioEngine` protocol and a Swift class conforms to it directly. No JNI bridge.

## The workflow shift

The Android workflow has been: Claude plans and writes prompts in this CLI, I paste them into Gemini in Android Studio, Gemini writes the code in the IDE. Gemini is free and in-IDE, Claude is for architecture, multi-file review, and prompt-writing.

That doesn't translate to iOS. Gemini doesn't run in Xcode. Xcode 26 has a "Claude Agent" feature that's supposed to give me Claude inside the IDE — but I burned fifteen minutes trying to authenticate it and ended up with the same "Please sign in" message after every retry. Apparently a known glitch.

So I gave up on in-IDE assistance and let Claude Code (this CLI) write the Swift files directly into the Xcode project. Claude has full read/write access to the project on disk; I sit in Xcode, build, run on simulator (or my iPhone for the bits that need hardware), tell Claude what I see, Claude edits, repeat.

It worked better than the Android setup, actually. The paste step disappears. The trade-off is that Claude is paid (subscription) whereas Gemini is free, so for slow ongoing iteration I might revisit. But for "port everything in one day," having Claude write the Swift end-to-end was clearly the right call.

I also tested on my actual iPhone via Xcode's free provisioning. "Personal Team" sideloading works without the paid Apple Developer account; the app expires in seven days. That's fine while I'm still pre-enrollment.

## The audio / visual grid trick

The metronome's pulse and click need to be in sync — otherwise it feels off. On Android, the C++ Oboe callback writes a host-time timestamp into an atomic that Kotlin polls every 8 ms; the Compose flashlight reads that timestamp and flashes when the click fires. **Audio drives, visual follows.**

On iOS I went the other direction. The flashlight derives its phase purely from `clock_gettime_nsec_np(CLOCK_UPTIME_RAW) mod periodNanos` — a continuously-running modular clock. The audio engine then aligns its `start()` and `setBpm()` calls to this same grid: when audio starts, the first click is scheduled for the next grid point, not for "right now."

**Visual leads, audio aligns.** Two consequences:

- The flashlight pulse is **continuous.** Stop the audio, the pulse keeps going. Press Play, the first click lands on the next flash, no visual jump.
- BPM changes mid-play realign cleanly. The first thing I tried — selecting a different song while the metronome was playing — exposed permanent drift between audio and visual. The fix was applying the same grid math to `setBpm()` that `start()` uses; the next click after a tempo change re-aligns to the new grid.

Claude proposed this approach when I asked for the flashlight. I had to ask for a second iteration to make the pulse continuous (originally the audio's `lastBeatNanos` was driving the visual, which meant the visual reset every time audio started). Once I described the feel I wanted — "the pulse should already be running when I press Play; the first click should join the next flash" — the rewrite was straightforward.

## State bridge between Kotlin and SwiftUI

`SessionState` in commonMain has an `onChange: (() -> Unit)?` callback. On Android, the `SessionViewModel` hooks into it and re-publishes to Compose. On iOS, a Swift `SessionStore: ObservableObject` does the same:

```swift
self.state.onChange = { [weak self] in
    MainActor.assumeIsolated {
        self?.sync()
        self?.persist()
    }
}
```

`sync()` reads the Kotlin state back out into Swift-side `@Published` properties that SwiftUI binds to. `persist()` writes the JSON snapshot via `SessionRepository`. Every mutation routed through `state.someMethod()` automatically flows back into SwiftUI and onto disk. No imperative wiring per feature.

Sealed-class casting was the one mild snag — Kotlin's `PlaylistItem.SongRef` exports to Swift as `PlaylistItem.SongRef` (nested), but I'd assumed `PlaylistItemSongRef` (flattened). One-line fix once Xcode pointed it out.

## UI in SwiftUI

This is where I had the most opinions. The Android UI is Material 3 Compose, which has a specific visual feel; the iOS port shouldn't pixel-port that. I wanted the screen to read as a native iOS app first, a SessionClick app second.

Things I **matched** with Android:

- Layout structure: playlist on top, controls below
- The flashlight as the center of the controls
- Brand color, matching Android primary (`#2E7D32` light / `#81C784` dark) via the AccentColor asset
- Flash toggle that's overridden ON while audio is playing
- Vertical volume slider on the right of the flashlight (this one I went back and forth on — see below)

Things I deliberately took the **iOS-native** path for:

- **Reorder by drag handles in Edit mode** (Apple Music / Reminders convention), not long-press (Android's pattern). Long-press is reserved on iOS for context menus.
- **NavigationLink instead of stacked sheets** when going from Settings into the Sound Editor. Apple's apps consistently use one navigation stack per modal, not "sheet on sheet."
- **Toolbar layout**: hamburger leading (Settings sheet), playlist switcher in the principal slot, "+" and Edit trailing.
- **Sheet detents** (`.medium` / `.large`) for the playlist switcher and the editors.
- **`MPVolumeView`** for system volume control — the only Apple-sanctioned way to programmatically read or write hardware volume.
- **`UIImpactFeedbackGenerator(style: .rigid)`** for Feel the Beat, fired by a 16 ms-polling Timer that watches beat-number transitions on the grid clock.

The volume slider went back and forth. I asked for vertical (Android-like) → Claude argued for horizontal (iOS-like) → I overruled. The rotation is a small hack (Apple doesn't officially support vertical `MPVolumeView`) but works. For a stage metronome, the vertical layout is genuinely better — controls flanking the flashlight at thumb reach.

## Things that needed iteration

- **Frequency slider didn't update during drag.** I'd used `Slider`'s `onEditingChanged` callback, which only fires on touch-down and touch-up. The Tone slider used a `Binding` setter, which fires on every drag delta. Switched Frequency to match.
- **Sheet stacking on Settings → Sound Editor.** First version opened Sound Editor as a sheet on top of Settings; closing Sound Editor left Settings visible underneath. Refactored to a `NavigationLink` — Sound Editor pushes into the Settings nav stack, one Back button per level.
- **Save Tempo shifting the +/− buttons** when it appeared. Moved it into a fixed-height slot between + and −, with a `ZStack` placeholder taking the same height when not dirty. The +/− positions stay stable.
- **Flashlight too dark at minimum opacity.** Asked Claude to bump the baseline so the inner circle stays "a little green" when not pulsing; the outer ring also dims when stopped, so the playing state reads at a glance.
- **Selected playlist row not bright enough**, breaks not distinct enough. Bumped the accent-color opacity from 0.18 to 0.45 and the break-row tint from 0.08 to 0.22.
- **Side columns flush to the screen edges.** Added explicit `width: 56` to both side columns and 24 pt outer padding, so the +/Save/− and the volume slider take equal space and the flashlight sits exactly centered.

None of these broke anything. Most were one-or-two-message fixes.

## What Claude did vs what I did

**Claude:**

- Wrote all the Swift. ~2,000 lines of new code across 11 new files and edits to 4 existing ones.
- Implemented the audio engine: AVAudioEngine + AVAudioSourceNode + grid alignment + WAV loading via AVAudioFile + UserDefaults persistence.
- Designed the `SessionStore` ↔ `SessionState` bridge.
- Set up the toolbar, sheets, NavigationLinks, picker, sliders, haptics, vertical volume rotation.
- Wrote the `IosFileStorage` in Kotlin/Native using the K/N Foundation interop.
- Integrated the app icons from my IconKitchen export.

**I:**

- Tested every build on the simulator and (some) on my iPhone via free provisioning.
- Made the design calls: visual references, iOS-native vs Android-parity decisions, button sizes, layout proportions, when to defer features.
- Pointed at the things that didn't feel right — slider not previewing during drag, sheet stacking, Save Tempo shifting +/−, flashlight too dark when stopped, font sizes in the playlist, side columns too close to the edges — so Claude could iterate.
- Committed and pushed at the natural break.

The split is different from the Android workflow. On Android I'm more involved with the code itself because Gemini's prompt has to be very specific; on iOS, with Claude Code writing directly, I'm closer to a designer / PM reviewing the build. Claude is doing the implementation work that Gemini does on Android, plus the planning work that Claude does on Android.

## What's still missing

- Open Source Licenses screen — placeholder in Settings
- Import / Export JSON — placeholder, needs share sheet + document picker
- StoreKit + freemium gate — €2 unlock, parity with Google Play Billing
- Landscape Stage Mode — deferred orientation work; the iOS build is portrait-only for now
- Apple Developer enrollment — still blocked on ELSTER → D-U-N-S; doesn't affect simulator/sideload dev, but needed for TestFlight and StoreKit

## Today's changelog

- iOS audio engine (AVAudioEngine, sample-accurate clicks)
- Tempo flashlight (grid-synced visual pulse, audio aligns to it)
- Swift-Kotlin state bridge via `ObservableObject` wrapping `SessionState`
- Persistence to Documents directory via `IosFileStorage`
- Top toolbar with playlist switcher, "+" add menu, Edit button
- Song Editor and Break Editor sheets (create + edit + cross-playlist propagation)
- Song Library picker (search, multi-select, "already in playlist" lockout)
- Settings sheet with live dark-mode toggle, imprint link, version display
- Sound editor with three WAV samples + synth click + frequency / tone sliders + preview on slider release and picker change
- App icon integrated from IconKitchen export
- Vertical volume slider (rotated `MPVolumeView`)
- Keep-screen-awake while foregrounded
- Feel the Beat haptic hold button
- Save Tempo button (visible only when current BPM differs from saved)
- Flash toggle (forced on while playing)
- Layout polish: side-column symmetry, brighter selection, dimmer ring when stopped, baseline green tint on the flashlight inner

---

**Time spent today:** ~6h

---

_This blog documents my attempt to build and ship a music app as a solo developer, with AI assistance. The AI does a lot of the work. I try to be specific about what._
