# Liman Shen — selected work

Software engineer in Toronto. I build AI products end to end: the interface, the
integration layer behind it, and the measurement that tells you whether it worked.

Three years at ParagraphAI as the primary author of its web app, Android app, and Chrome
extension — one of four products from that team. Before that, six years trading
Canadian money markets at CIBC, automating the desk's reporting. Looking for a remote,
Canada-eligible product team building AI features at a sustainable pace.

[GitHub](https://github.com/limanup) · [LinkedIn](https://www.linkedin.com/in/limanshen/) · [Email](mailto:liman123shen@gmail.com) · [Site](https://limanup.github.io/)

---

## Selected work

### 1. [Product clients](projects/product-clients.md)

Primary author of ParagraphAI's three clients across three years: 2,363 of 2,602 commits
in the web app, 524 of 832 in Android, and 318 of 401 in the Chrome extension, across 165
tracked tickets. Rebuilt the web app from Gatsby to Vite and shipped 400+ releases through
one GitHub Actions path. Migrated eleven Android screens from XML layouts to Jetpack
Compose and shipped an AI keyboard. Extended the MV3 extension with streamed edit
suggestions, a content-script toolbar, and an AI Summary side panel.

`TypeScript` `React` `Kotlin` `Jetpack Compose` `Chrome MV3` `Vite` `Playwright`

### 2. [Cross-platform measurement](projects/measurement-stack.md)

Built the attribution stack that made paid spend accountable across the web app, iOS,
Android, and the Chrome extension. Adjust from scratch on Android, Adjust plus Facebook
and Firebase conversion events on iOS, the Adjust Web SDK and GA4 client ID into Stripe
metadata on web, and Awin and Taboola click IDs persisted and carried into payment
metadata inside a one-year window. Also fixed the bugs that made the data untrustworthy:
trial subscriptions reporting `$0`, a deep link dropping attribution on app open, and
affiliate click IDs lost between landing page and payment redirect.

`TypeScript` `Kotlin` `Swift` `Stripe` `GA4` `Adjust` `Firebase` `Cloudflare Workers`

### 3. [Parlez](projects/parlez.md)

A real-time voice agent for couples, from an empty repo to a working product in eight
weeks on the Gemini Live API. 66 of 115 commits. Barge-in with active-source management,
voice-activity detection, and a ten-minute session structure encoded in the system
instruction so the model knows when to explore and when to close. Handed to the investor
who initiated it; live at [parlez.com](https://parlez.com).

`TypeScript` `React` `Cloudflare Workers` `Gemini Live` `Web Audio`

### 4. [On-device speech recognition](projects/on-device-speech.md)

Converted a wav2vec2 phoneme-recognition model to Core ML so an iPhone could score a
child's pronunciation without uploading the recording. The obvious PyTorch → ONNX → Core
ML route failed; the working path traces the PyTorch module and declares a 30-second
runtime input range. The public rebuild produces FP32, FP16, and INT8 packages and
benchmarks them by size, cold load, and latency; FP16 is half of FP32's size with the same
output tokens, so it is the one that ships.

`Python` `PyTorch` `coremltools` `Core ML` `iOS`

### 5. [MailScience](projects/mailscience.md)

An email deliverability checker for SPF, DKIM, DMARC, MX, reverse DNS, WHOIS, and 19 DNS
blocklists, implemented in both Python and TypeScript. Built in a day to test a product
hypothesis — and to answer it, which was the point.

`Python` `TypeScript` `DNS`

---

## How I work

- **Ship the whole loop.** A feature is not done when it renders; it is done when I can
  tell whether it worked. I instrument what I build.
- **Prefer the boring path that survives.** The ONNX detour looked standard and failed
  four times. Tracing the module directly was less obvious and worked.
- **Keep the user in the room.** I sat in on 30+ customer interviews and was the
  engineering contact in client-facing Slack and Discord channels.

## Background

- **ParagraphAI** — Software Engineer, then Senior Software Engineer · May 2023 – Present · Remote
- **CIBC Capital Markets** — Money Market Trader · Jun 2015 – Jan 2022 · Toronto
- **University of Texas at Austin** — MS Computer Science · May 2025
- **University of Waterloo** — BMath, Honours, With Distinction
