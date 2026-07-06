---
title: "One-command releases, real numbers, and a finished safety net"
date: 2026-07-06
draft: true
tags:
  [
    "process",
    "ai-assisted",
    "claude",
    "fastlane",
    "release-automation",
    "testing",
    "app-store",
    "google-play",
    "kmp",
  ]
summary: "The infrastructure phase is done. Two days took SessionClick from 'releasing is a click-odyssey through two consoles' to one-command releases on both stores, a monthly report with real download numbers, and UI smoke tests on Android and iOS — with an AI agent doing the work and me holding the credentials. Every failure along the way became documentation."
---

[Last post](/posts/2026-07-04-making-sessionclick-robust-for-the-long-haul/) ended
with a promise: fastlane lanes, a usage report, UI smoke tests — then, and only then,
features. Two days later the plan file says **complete**. Here's what those
workstreams looked like in practice, including everything that went wrong — because
the failures turned out to be the valuable part.

## The agent works, I hold the keys

Release automation means credentials: a Google Play service account, App Store
Connect API keys, keystore passwords. The split we settled on: everything secret
lives outside the repo in `~/.config/musicapps/`, created by me in the two consoles,
and the AI agent that builds the automation **cannot read that directory at all** — a
deny rule in the agent's own permission config, which the agent set up itself when I
asked how to lock it out. It then verified its own blindfold by planting a canary
file and failing to read it back.

This had a funny consequence later: the agent had to write a parser for a
credentials file it couldn't see. First guess at the variable names — wrong. Second
guess — wrong again, my file used `KEYID: …` with colons where it expected
`KEY_ID=…`. The parser now accepts both separators and any common naming variant,
which is objectively more robust than if it had just read my file. Constraints
breed generality.

The Gradle side follows the same pattern: the release `signingConfig` reads a
properties file from that directory, and if the file is absent — CI, fresh clone,
nosy agent — the build silently falls back to unsigned. CI never sees a secret.

## fastlane, and two educational failures

Four lanes now cover both stores: `android_beta` (bump versionCode → signed AAB →
Play internal track), `android_promote` (internal → production), `ios_beta` (bump
build number → TestFlight), `ios_release` (prepare the App Store version entry —
release notes and the actual submit stay manual, on purpose). Release tags, phased
rollouts, and real-device smoke tests also stay manual. The lanes automate the
click-odyssey, not the judgment.

First `android_beta` run: **failed.** fastlane auto-detected a leftover debug APK
next to the release AAB and refused to upload both. One `skip_upload_apk: true`
later, versionCode 13 landed on the internal track — now with the R8 `mapping.txt`
uploaded alongside, so Play Console crash reports arrive deobfuscated.

First `ios_beta` run: **failed better.** Apple rejected the upload with error 90186:
*"The train version '1.0.1' is closed for new build submissions."* Once a version is
approved on the App Store, that version string is burned — no new TestFlight builds
under it, ever. Google Play happily takes new versionCodes under an unchanged
versionName; Apple does not. So the iOS release checklist now has a step zero: bump
`MARKETING_VERSION` before the first beta of a new cycle. Which, thanks to an
earlier task this phase, is one line in one file — the version numbers used to be
duplicated between `Config.xcconfig` and the Xcode project file, where a hard-coded
target-level value silently overrides the xcconfig. The duplication is gone;
1.0.2 build 8 went to TestFlight on the second attempt.

One more paper cut automated away: every TestFlight build sat at "Missing
Compliance" until I clicked through the encryption questionnaire. A metronome
implements no cryptography; `ITSAppUsesNonExemptEncryption = false` in the
Info.plist answers the question permanently.

All of it — the lanes, the failures, the fixes — went into `docs/RELEASING.md`,
written for someone who has never used fastlane. That someone is me, next January.

## Sixteen downloads and €1.42

The reporting decision from last post — store APIs only, no analytics SDK — became
a small Python script: one command, one markdown report per month, both platforms
side by side. Downloads, premium unlocks, proceeds, crashes, ratings.

Reality adjusted the plan here too. The Play *Developer Reporting API*, which the
plan confidently named as the data source, turns out to expose only vitals (crash
rates). The actual installs and ratings live in monthly CSV exports in a Google
Cloud Storage bucket — encoded in UTF-16, because Google apparently exports them
straight from 2009. And after granting the service account bucket access, it kept
getting 403s for hours; the API probe proved the permission was active while the
bucket ACL lagged behind. Google documents "up to 24 hours" for that sync. The
script now degrades per-platform — if one store's API is down or a permission
lags, you still get the other store's numbers plus a note explaining what's missing.

June 2026, SessionClick's first almost-full month: **16 iOS downloads, 1 premium
unlock, €1.42 in proceeds.** Break-even on the ~€38 of annual fixed costs needs
roughly 190 unlocks. So: 1 down, 189 to go. Honest indie numbers — and now they
arrive monthly without me clicking through two dashboards.

## Smoke tests: four on Android, seven on iOS

The last gap in the safety net was UI-level: nothing verified that the app actually
*launches*. Now both platforms have smoke tests — deliberately few, pinning the
critical path only: launch, play/stop (through the real audio engine and foreground
service), add-a-song including the JSON hitting disk, and the freemium gate.

The freemium test demonstrates a trick worth stealing: instead of tapping "add song"
fifty times to reach the free-tier limit, the test writes a pre-built 50-song
`session.json` using the shared Kotlin persistence code, then launches the app into
it. Same data path as production, three orders of magnitude faster.

Each platform contributed one gotcha. On Android, all four tests initially failed
with `NoSuchMethodException: InputManager.getInstance` — Compose's UI-test library
bundles an Espresso old enough to use reflection that newer Android versions
removed. Pinning the current Espresso on the test classpath fixed all four at once.
On iOS, the new test targets inherited `PRODUCT_NAME=SessionClick` from the
project-level xcconfig — every target claimed to *be* the app, and the build failed
on colliding Swift modules until the test targets got their own names. (The test
targets themselves were added programmatically with the `xcodeproj` Ruby gem rather
than hand-editing 500 lines of pbxproj — the gem ships inside fastlane, which was
already installed.)

And the extract-pure-logic-then-test pattern from last post delivered again, for the
third time in three uses. Extracting the StoreKit unlock condition into a testable
`StoreRules` revealed that the iOS transaction listener would set `isUnlocked = true`
for **revoked** transactions — refunds arrive through the same stream as purchases,
and the revocation date was only checked on one of the two code paths. A refunded
purchase could re-unlock premium. One condition, four tests, bug gone. At this point
the pattern has found an out-of-bounds crash, deduplicated a billing condition, and
fixed a refund bug — extraction isn't test ceremony, it's a bug-finding tool.

Small symmetric bonus: both platforms' play buttons now have proper
accessibility labels ("Play"/"Stop"). Added as test handles; welcome regardless.

## The loop is closed

Final inventory of the phase: 43 shared tests + 12 Android unit + 4 Compose smoke +
6 XCTest + 1 XCUITest, detekt with a frozen baseline, CI, a pre-push hook, release
lanes proven against both real stores, a monthly numbers report, and a documentation
net where every file points back to one master map. An agent — or me, cold, in
January — can go from `git clone` to a safe, bounded work session in three hops.

The meta-lesson of these two days: **every failure became documentation.** The
leftover-APK failure is a line in the Fastfile. The closed-train rejection is step
zero of the iOS checklist. The UTF-16 CSVs and the lagging bucket ACL are comments
in the report script. None of it lives in my head, which is good, because my head
has gigs to remember.

Next post: an actual feature. The backlog has three specs with their open questions
answered, and a two-day-old safety net that's itching to catch something.
