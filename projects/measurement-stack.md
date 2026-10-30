# Cross-platform measurement

**Making paid spend attributable across web, iOS, Android, and a browser extension.**

`TypeScript` `Kotlin` `Swift` `Stripe` `GA4` `Adjust` `Firebase` `Cloudflare Workers`

---

## The problem

The company was buying traffic on Google and Meta with no reliable join between an ad click
and a subscription. A marketing agency had been hired to fix it and produced nothing the
team could use. Nobody internally owned the problem, so it landed as an undefined task:
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
- **iOS** — Adjust attribution callback into Segment `identify`, with null handled as `""`
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
- **Backend** — Stripe subscription webhooks, Android billing through a Pub/Sub topic, and
  Cloudflare Workers for token minting. The click IDs live in Stripe metadata because that
  is where the conversion becomes real.

## The bugs that made the data lie

Installing the SDKs was the easy half. The numbers were still wrong until these were fixed:

- **Trial subscriptions reporting `$0` revenue** — the trial offer was priced at zero and
  the revenue property copied the price. Fixed so trials reported the eventual plan value.
- **A deep link dropping attribution on app open** — the click context was lost between
  the redirect and the first session.
- **Affiliate click IDs lost between the landing page and the payment redirect** — the
  `awc` parameter was captured but not persisted before the Stripe handoff.

## Why it matters

Before this, reporting lag was measured in days and paid spend could not be tied to a
channel. After it, the team could see which campaigns produced subscriptions instead of
which produced clicks. The work is less visible than a feature and more valuable than most
of them: it changes what the company decides to spend money on.

## What I would do differently

- Define the event schema and the identity join before touching an SDK. Half the bugs came
  from each surface inventing its own notion of a user.
- Build a small reconciliation job that compares platform-reported conversions against the
  payment webhooks daily. Attribution silently drifts; a diff makes it loud.
- Treat the one-year click window as a product decision with a documented owner, not a
  constant buried in code.
