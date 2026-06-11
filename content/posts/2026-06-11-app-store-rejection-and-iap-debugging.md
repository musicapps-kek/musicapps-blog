---
title: "App Store rejection, IAP debugging, and a pile of small iOS fixes"
date: 2026-06-11
draft: false
tags: ["ios", "swiftui", "storekit", "bugfix", "appstore", "ai-assisted", "claude"]
summary: "The app came back rejected for an unresponsive IAP button. Tracking it down led through four separate bugs in the purchase flow, plus a handful of smaller fixes: edit mode not exiting on play, background audio sync, and a haptics regression on iPhone 15 Pro."
---

The submission came back rejected. Guideline 2.1(b) — the unlock button was unresponsive in review.

That one stings because it looked fine in my testing. But there were real bugs, and fixing them took most of today's session.

## Four bugs in the purchase flow

**The button was disabled but didn't look it.** The product is loaded asynchronously after the view appears. While it's loading, the button showed "Unlock" but was disabled — it looked tappable but wasn't. The fix: track loading state explicitly so the button shows a spinner while waiting and "Retry" on failure. A button that looks active must be active.

**Nested sheets delayed the paywall.** The "Unlock Premium" button lives inside Settings, which is already a sheet. Triggering the paywall from there queued it behind the current sheet — it only appeared after the user pressed Done to close Settings. Fixed by dismissing Settings first and letting the paywall present from the top level.

**StoreKit returned `userCancelled` with no dialog.** Even after fixing the sheet nesting, the purchase returned `userCancelled` immediately without showing the system payment dialog. The cause: StoreKit couldn't determine which window to present on. Fixed by passing the active window scene explicitly via `product.purchase(confirmIn: scene)` — the right API for this.

**The paywall didn't close after purchase.** After a successful purchase the paywall stayed open showing the buy button as active. A one-line `onChange` on `isUnlocked` now dismisses the view as soon as the transaction is verified.

## Other fixes

**Edit mode didn't exit on play.** If the playlist was in edit mode when you pressed Play, the Edit button disappeared but the list stayed in rearrange state. Starting the metronome now sets `editMode` to `.inactive` first.

**Background audio and sync.** Sending the app to the background faded the audio, and returning left the sound out of sync with the Flashlight. The fix is to declare the "Audio, AirPlay, and Picture in Picture" background mode in Xcode — no code change needed, the audio session category was already correct. With the capability declared, the engine keeps running and sync is never lost.

**Feel the Beat haptics on iPhone 15 Pro.** A tester reported the haptic pulse not working. Most likely cause is a system setting (Sounds & Haptics must be on, Low Power Mode must be off — both suppress all feedback generator haptics). On the code side, the generator was stored as a plain `let` property on a SwiftUI View struct, meaning a new instance was created on every render and the prepared state discarded. Switched to `@State`, bumped the style to `.heavy`, and set `impactOccurred(intensity: 1.0)` explicitly. Waiting to hear if it helped.

## What ships today

A new TestFlight build goes out tonight. Here's what changed:

**Purchase flow** — the paywall button now always communicates its state: spinner while loading, price when ready, Retry on failure. The system purchase dialog appears reliably. The screen dismisses itself after a confirmed purchase. All four bugs in the original flow are fixed.

**Settings → Unlock Premium** — tapping it now opens the paywall immediately, without the Done-first workaround.

**Edit mode exits on play** — starting the metronome while the playlist is in rearrange mode now exits edit mode automatically.

**Background audio** — the metronome keeps running when you switch to another app. Coming back to the app, the sound and the Flashlight are still in sync. This is the behavior musicians would expect and something I should have enabled from the start.

**Feel the Beat haptics** — stronger style, proper generator lifecycle. Whether this fixes the iPhone 15 Pro report is still open, but the code is better regardless.

After this build is out, the App Store submission goes back in. The rejection was frustrating but the bugs were real. The purchase flow in particular had four independent issues that only surfaced under review conditions — a good reminder that sandbox testing on a real device is not optional for IAP.

---

**Time spent today:** ~2.5h

---

_This blog documents my attempt to build and ship a music app as a solo developer, with AI assistance. The AI does a lot of the work. I try to be specific about what._
