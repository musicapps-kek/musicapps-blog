---
title: "Day 11(b): Registering a business for a side project"
date: 2026-04-18
draft: false
tags: ["business", "apple-developer", "indie-dev", "germany", "admin"]
summary: "To publish on the App Store under a brand name, you need an Apple Developer Organization account. That requires a D-U-N-S number. That requires a registered business. Two hours of research with Perplexity, 45 minutes online with the German trade registration portal, 26 EUR, and MusicApps.eu exists as a legal entity."
---

Before you can ship an app on the App Store under a brand name, you need an Apple Developer Organization account. Before you can get that, Apple requires a D-U-N-S number — a business identifier issued by Dun & Bradstreet. Before you can get a D-U-N-S number in Germany, you need a registered business (_Gewerbeanmeldung_). So that's what today's non-coding session was about.

## The research first

German business registration touches tax law, trade law, and social insurance — all of which I needed to understand well enough to make the right decisions, not just fill in a form. I used Perplexity Pro for this research, in German, because the relevant rules are German-language and I wanted precise answers rather than generic summaries.

The key questions I needed answered:

### Individual vs. Organization Apple account.

An Individual account lets you ship without any business registration, and apps appear under your personal name. An Organization account requires a D-U-N-S number but lets you publish under a brand name. The annual fee is the same either way. For a project with a specific brand (_MusicApps_, _SessionClick_) and a plan to ship multiple apps, the Individual route would mean either shipping under my personal name or restarting the account later — neither appealing.

### What does registering a _Gewerbetrieb_ actually cost and oblige you to?

The registration fee varies by municipality but is typically 15–70 EUR. After that: the tax office (_Finanzamt_), the Chamber of Commerce (_IHK_), and any other relevant authorities are automatically notified. Ongoing obligations are an annual income tax return with a simple revenue/expense statement (_EÜR_). Trade tax (_Gewerbesteuer_) doesn't apply below 24,500 EUR annual profit — which covers any realistic scenario for this project. If the project ends, you actively de-register and the obligations end once the final tax return is filed.

### Freelance (_freiberuflich_) vs. commercial (_gewerblich_)?

This distinction matters in Germany. Freelance activities — typically creative or intellectual work — are registered only with the tax office and are exempt from trade tax. Commercial activities go through the trade office and bring IHK membership. Software development is unambiguously commercial. Some of my other occasional activities (musician gigs, PA rental, sound tech) sit in a grey area, but the pragmatic answer is: register everything under one commercial description, keep internal records by category, and the actual tax impact at small scale is minimal.

### How broad should the activity description be?

A trade registration can be amended but it's cleaner to get it right upfront. The description I filed covers software development, mobile apps (especially for musicians), instrumental accompaniment and sound tech at musical performances, and rental/provision of event technology. Broad enough to cover everything I realistically might invoice under this entity; narrow enough to be coherent.

**Kleinunternehmerregelung?** Germany has a small-business exemption that lets you invoice without charging VAT (_Umsatzsteuer_) if your annual revenue stays below a threshold. For a project in its first year with no guaranteed revenue, this avoids a significant layer of quarterly reporting. I'll decide this when filling in the tax registration questionnaire via ELSTER — but it's likely the right choice for now.

## The registration

The actual registration was done via the _Wirtschafts-Service-Portal NRW_ — the digital trade registration portal for North Rhine-Westphalia. It was fully online, took about 45 minutes including reading everything carefully, and cost 26 EUR paid electronically through the portal.

Key details from the registration:

- **Business name:** MusicApps.eu
- **Legal form:** Non-registered sole proprietorship (_nicht eingetragenes Einzelunternehmen_) — the standard form for a Kleingewerbe
- **Activity summary:** Software development and support at music events
- **Side business:** Yes — no employees, operated alongside a main job
- **Start date:** 20 April 2026
- **Registered with:** Gewerbeamt Bünde

## What this unlocks

The registration itself doesn't mean much administratively until the tax office responds with a questionnaire (expected within a few weeks). The actual sequence from here is:

1. Fill in the ELSTER tax questionnaire (_Fragebogen zur steuerlichen Erfassung_) — decide on Kleinunternehmerregelung
2. Request a D-U-N-S number via Dun & Bradstreet (free, 5–14 business days)
3. Set up Apple Developer Organization account once D-U-N-S arrives
4. Set up simple bookkeeping — income/expense tracking by category from day one

The Google Play developer account doesn't require a business registration — that was already on the todo list as a straightforward 25 USD one-time fee.

## Is it worth it for a side project?

The honest answer is: maybe not, if the only goal was to ship one app as quickly as possible. An Individual Apple account would have worked.

But the goal isn't one app. The goal is a small portfolio of music-focused utilities under a consistent brand, with a professional enough presentation that musicians take it seriously. Publishing as "MusicApps" rather than a personal name is part of that. Having a legal entity for the invoicing, the store accounts, and the eventual domain registrations makes everything cleaner as it grows.

The bureaucratic overhead is real but manageable — one registration, one annual tax return, simple bookkeeping. AI assistance makes all of it lighter: Perplexity handled the research; Claude will help with the ELSTER questionnaire; I'll use a simple spreadsheet for the bookkeeping. For a solo developer with limited time, that's the right stack for the admin layer too.

## Keeping the paperwork organised

A registered business generates documents: the registration form, letters from the tax office and the IHK, receipts for every deductible expense, and eventually annual tax returns. For a solo developer who already juggles coding sessions, blog posts, and a day job, letting that pile up is a bad idea.

I set up a folder structure in Obsidian to hold everything from day one:

```
Gewerbe/
├── Anmeldung/              registration form and confirmations
├── Finanzamt/              ELSTER questionnaire, tax returns, EÜR drafts
├── IHK/                    Chamber of Commerce correspondence
├── D-U-N-S/                application and confirmation
├── Belege/2026/
│   ├── Apps-Software/      receipts: domains, subscriptions, store fees
│   └── Musik-PA-Tontechnik/ receipts: equipment, travel to gigs
└── Einnahmen-Ausgaben/2026/ running income/expense table (EÜR prep)
```

The income/expense table is already started with the registration fee as the first entry. The naming convention for receipts is `YY-MM-DD_description_amount.pdf` — simple enough to sort chronologically without any additional tooling.

The key insight from the Perplexity research: for German tax purposes, the **invoice** (_Rechnung_) is the relevant document for deducting a business expense, not the payment receipt (Which might also be relevant, but the invoice is primary). So for subscriptions like Claude Code or domain hosting, it's the monthly invoice PDF that goes into `Belege/`. For one-time fees like the Gewerbeanmeldung, it's the confirmation email PDF from the portal. For the D-U-N-S application, it's the confirmation email from Dun & Bradstreet. But I will also keep the payment receipts for reference. Keeping those organised from day one means when tax time comes, everything is in one place and easy to find.

---

**Time spent today:** ~3h (research + registration + infrastructure setup)

---

_This blog documents my attempt to build and ship a music app as a solo developer, with AI assistance. The AI does a lot of the work. I try to be specific about what._
