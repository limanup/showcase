# Cross-platform measurement

**Making paid spend attributable across web, iOS, Android, and a browser extension.**

`TypeScript` `Kotlin` `Swift` `Stripe` `GA4` `Adjust` `Firebase`

---

## The problem

The company was buying traffic on Google and Meta with no reliable join between an ad click
and a subscription. An outside agency had been hired first, and that did not lead to a
working solution. Nobody internally owned the problem, so it landed as an undefined task:
make ad spend measurable.

The hard part is not installing an SDK. It is that a subscription is real only when the
payment webhook fires, and the click that earned it happened on a different surface, in a
different app, possibly weeks earlier. Attribution is a data-model problem disguised as an
integration problem.

## What I built

Measurement on every surface, each writing into the same user record:

- **Android** — Adjust from scratch: dependencies, configuration, session tracking,
  signed-up events for email and Google sign-in, subscription events with a period
  property, sandbox to production. Later, Facebook SDK purchase events for subscriptions.
- **iOS** (my part: Nov 2023 – Feb 2024) — Adjust attribution callback into Segment `identify`, with null handled as `""`
  rather than dropping the update; manual subscription events split trial/regular and
  monthly/annual; Firebase and Google Analytics conversion events.
- **Web** — the Adjust Web SDK, GA4 `client_id` threaded into `subscription_data.metadata`
  and financial analytics events, and Adjust server-to-server subscription events.
- **Chrome extension** — GA4 Measurement Protocol events with the `client_id` passed from
  the web app through local storage, `user_properties` attached, `debug_mode` gated out of
  production, and event names renamed to avoid double counting against Segment.
- **Affiliate and paid channels** — Awin mastertag and conversion tag, `awc` captured from
  the URL and persisted to Firestore with the click timestamp, then passed to Stripe
  metadata only inside a one-year window. Taboola conversion pixels for sign-up and test
  sign-in.
- **Backend** — click IDs and the GA4 `client_id` live in Stripe subscription metadata,
  because the conversion is only real when the payment webhook fires.

## The bugs that made the data lie

Installing the SDKs was the easy half. The numbers were still wrong until these were fixed:

- **Trial subscriptions reporting `$0` revenue** — trial offers were priced at zero, so the
  Subscribed event reported zero. Fixed the revenue property.
- **A broken attribution deep link** — the link that carries attribution into an installed
  app was not working. Fixed in May 2024.
- **Dropped affiliate click IDs** — the `awc` parameter had to survive from the landing page
  to the commission report. It is now persisted to Firestore with a click timestamp and
  passed into Stripe metadata.

## Why it matters

Before this, paid spend could not be tied to a channel. After it, the team could see which campaigns produced subscriptions instead of
which produced clicks. The work is less visible than a feature and more valuable than most
of them: it changes what the company decides to spend money on.

## What I would do differently

- Define the event schema and the identity join before touching an SDK. Half the bugs came
  from each surface inventing its own notion of a user.
- Build a small reconciliation job that compares platform-reported conversions against the
  payment webhooks daily. Attribution silently drifts; a diff makes it loud.
- Treat the one-year click window as a product decision with a documented owner, not a
  constant buried in code.
