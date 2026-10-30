# Parlez

**A real-time voice agent for couples, built in eight weeks on the Gemini Live API.**

`TypeScript` `React` `Cloudflare Workers` `Gemini Live` `Web Audio`

---

## The problem

ParagraphAI's writing assistant was a text product. The team's next bet was to find out
whether the same company could hold a conversation — not generate a paragraph, but talk
with two people at once, in real time, about something difficult.

Parlez is a relationship coach for couples. Two partners share one session with an AI
therapist for ten minutes: it opens the conversation, keeps both people heard, and closes
on time. The product is not the model. It is everything around the model that makes a
conversation feel like one.

## Why voice changes the engineering

A text product waits for you to finish. A voice product does not.

- **Either person can interrupt.** Barge-in has to stop playback immediately, cancel the
  in-flight response, and track which audio source is active so the agent does not talk
  over the person it just acknowledged.
- **Silence is ambiguous.** Voice-activity detection has to distinguish a pause from a
  turn ending, then clean up its own state when the turn is abandoned.
- **Sessions need a clock.** The model has to know how much time is left. That is not a
  UI concern; it is part of the instruction the model runs on.
- **Latency is perceptible.** Bidirectional streaming over a WebSocket is the only way the
  reply lands before the pause becomes awkward.

## What I built

**The app.** React 18 web client and a Cloudflare Workers API, in one monorepo, on
`@google/genai` with the Gemini Live API for bidirectional audio. 66 of the project's 115
commits are mine.

**The conversation.** A session structure written as the system instruction: stages,
goals, and concrete actions for each phase — opening, understanding the pattern, going
deeper, introducing tools, wrapping up. The wrap-up phase changes the model's objective
from exploring to concluding: no new topics, wait for a natural pause, give a time warning
in varied language, consolidate. Past a hard stop, it closes regardless.

**Time awareness.** The client sends elapsed time with each turn. When sessions were cut
from thirty minutes to ten, the tick interval moved from 30 seconds to 60 seconds, and the
phases were rewritten around the shorter arc.

**The rest of the product.** A Radix UI settings sidebar with theme toggle, SEO metadata
and `robots.txt`, and versioned releases from `0.1.1` to `0.1.7`.

## What happened to it

The project was handed to the investor who initiated it, and it is live at
[parlez.com](https://parlez.com). I built the working product, not the business around it.

## What I learned

- **Talking is a different medium.** Most of the work was not the model call; it was
  interruption, silence, and turn-taking — the things a text interface never has to model.
- **Put the clock inside the prompt.** Session pacing belongs in the instruction, next to
  the persona, not in a timer that fires at the UI layer.
- **Shorten the demo, lengthen the structure.** Ten minutes forced a clearer arc than
  thirty. The constraint improved the product.

---

The code is in a private company repo. This write-up is the public artifact.
