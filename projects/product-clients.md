# Product clients

**Primary author of a consumer AI product's web app, Android app, and Chrome extension.**

`TypeScript` `React` `Kotlin` `Jetpack Compose` `Chrome MV3` `Vite` `Playwright`

---

## The scope

ParagraphAI is a consumer writing assistant. The clients were a web app, an Android app, a
Chrome extension, and an iOS app sharing a backend. Across three years I was the primary
author of three of them:

| Client | My commits | Total | Share |
|---|---:|---:|---:|
| Web app | 2,363 | 2,602 | 91% |
| Android | 524 | 832 | 63% |
| Chrome extension | 318 | 401 | 79% |

165 tracked tickets. All three clients were inherited mid-flight, not greenfield.

## Web app

- Rebuilt the app from Gatsby to Vite and TypeScript with a PWA shell and a Lexical editor,
  and ran the migration without a feature freeze.
- Built the Write, Reply, Longform, and Humanize flows that turn one input into up to ten
  output formats.
- Implemented server-sent-event streaming per output format, with per-output loading state,
  abort support, and cancellation on navigation, so each result renders the moment it is
  ready instead of waiting on the slowest generation.
- Built custom output templates that users define with their own instructions and examples,
  then save, archive, and share, synced live to the Chrome extension through Firebase RTDB.
- Added web-page and PDF attachments by fetching and cleaning third-party content in
  Cloudflare Workers behind JWT and ID-token verification, with token-aware size limits and
  per-file error states.
- Set up the GitHub Actions pipeline: Vitest unit tests and Playwright end-to-end tests on
  every pull request, results posted as a commit status and PR comment, plus a
  tag-and-release workflow. Shipped 400+ releases through one path.
- Instrumented the app with New Relic real-user monitoring and custom dashboards, giving
  regressions a number and an owner instead of an opinion.

## Android

- Shipped an AI keyboard in Kotlin: instant fixes in any editor, replies generated from
  email, chat, and clipboard, and controls for tone, style, formality, and length.
- Migrated eleven screens — splash, onboarding, keyboard activation, auth, sign-up, account,
  settings, keyboard settings, language settings, history, trial — from XML layouts to
  Jetpack Compose, deleting the legacy layouts and introducing a shared Material 3 theme.
- Moved sign-in from Google One Tap to Credential Manager, raised `minSdk` to 26, and
  removed desugaring and deprecated Play Core dependencies.
- Integrated the Fleksy keyboard SDK with a vendored AAR, a local billing module, Proguard
  rules, and Gradle tuning, unblocking release builds.

## Chrome extension

- Extended the MV3 extension with streamed edit suggestions across eleven categories.
- Added a content-script toolbar with keyboard shortcuts and per-site hiding, and an AI
  Summary side panel.
- Added plan-based gating between free and paid actions.

## What the work taught me

- **Migrations are a product decision.** The Gatsby → Vite rebuild was only safe because it
  shipped behind the existing routes, with no feature freeze and no user-facing flag day.
- **Streaming is an interaction design problem.** Per-format loading state, abort, and cancel
  on navigation mattered more than the transport.
- **The extension is a second frontend.** Syncing templates through Firebase RTDB meant
  treating the extension as a client with its own state, not a script tag.
