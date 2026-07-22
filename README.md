# North Star Support Bot

A customer support chatbot for a fictional outdoor apparel and camping gear
store, built for the Upwork Talent Accelerator simulated project
(AI Chatbot Developer).

**Zero setup. Zero API keys. Zero dependencies.**

## Demo video

A 2-3 minute walkthrough covering all four use cases, fallback handling, and the
live-agent handoff accompanies this delivery (submitted with the project files).

## How to run

Open `index.html` in any modern browser. That's it.

Or serve it locally if you prefer:

```
python3 -m http.server 8000
# then visit http://localhost:8000
```

No API keys, no subscriptions, no accounts, no build step, no network calls.
Everything runs client-side with deterministic logic and the provided mock data.

## Requirements coverage

| Requirement | Where it lives / how to test |
|---|---|
| **1. Order Tracking** | Say "Track my order" or "Where's my package?" → bot asks for the order number (or answers instantly if you typed it inline, e.g. "track order 222") |
| Mock data (exact) | `#111` → Shipped, arriving tomorrow · `#222` → Processing, ships in 24 hours · `#333` → Delivered + follow-up question · any other number → invalid order with retry options |
| **2. Returns & Exchanges** | Say "I want to return something" → 30-day policy, unused items, original packaging + returns link |
| **3. Product Recommendations** | Say "Recommend a product" or "what should I buy" → 2 clarifying questions (activity, conditions) → recommends a product category |
| **4. Human Handoff** | Say "talk to a live agent" — or, after two unrecognized inputs, accept the bot's live-agent offer (tap the chip or just say "yes") → clear transfer notice, header badge switches to **Live Agent**, simulated agent (Riley) takes over, user can keep chatting or return to the main menu |
| **Intent recognition** | Whole-word/phrase matching with many variations per intent ("Where is my order?", "track my package", "order status", "where's my stuff", etc.). Apostrophes and punctuation are normalized. Order tracking additionally requires both a subject and a cue, so an unrecognized noun ("Where is my \<abc\>?") falls back instead of being guessed into a lookup. |
| **Conversation flow** | Guided flows with quick-reply chips at every step; every resolved flow returns the user to the main menu |
| **Shipping info** | Say "shipping info" → Standard 3-5 business days, Expedited 1-2 business days (also included with recommendations) |
| **Fallback handling** | Unrecognized input → clear "I didn't quite catch that" + option chips; a second consecutive miss offers escalation to a live agent |
| **Return after handoff** | "Return to main menu" chip (or type "menu") restores the bot state; badge switches back to **Bot** |

## Suggested test script

1. Type: `where's my order?` → then `111` → shipped/arriving tomorrow
2. Type: `track order 222` → processing/24 hours (inline number path)
3. Type: `333` (a bare order number is recognized as a tracking request) → delivered + follow-up → tap "Start a return"
4. Type: `track order 999` → invalid order handling
5. Type: `what should i get for camping` → the bot catches "camping" from the sentence and only asks the remaining question (warm or cold) → category recommendation. (A generic "recommend a product" asks both questions.)
6. Type: `how fast is shipping` → shipping options
7. Type: `asdfghjkl` twice → fallback, then escalation offer
8. Tap "Talk to a live agent" → Live Agent state → "Return to main menu"

## Architecture (single self-contained file)

- **Mock data layer** — order statuses and policies exactly as provided in the brief
- **Intent engine** — normalized whole-word/phrase matching with extensive phrasing
  variations. Two rules keep it from over-matching:
  - **Order tracking needs a subject and a cue.** A lookup must name something
    shippable (order, package, delivery, an order number, or a pronoun standing
    in for one) *and* ask a tracking question (where, when, status, arrived,
    late). Shopping questions about stock, price, or size are excluded. This is
    why "Where is my order?" is a lookup and "Where is my \<abc\>?" is not.
  - **Short replies only count when they are the reply.** Words like "ok",
    "fine", "later" and "perfect" are ordinary English, so they register as
    agreement, refusal, thanks, or goodbye only when the message is essentially
    just that word — "Is this jacket ok for rain?" is a question, not a yes.
- **State machine** — `MENU`, `AWAIT_ORDER`, `DELIVERED_FU`, `RECO_ACTIVITY`, `RECO_COND`, `AGENT`. A request for a live agent or the main menu is honored from any bot state; the live-agent chat manages its own exit so ordinary words don't eject the user. A consecutive-miss counter offers escalation to a human.
- **UI layer** — chat rendering with typing indicator, quick-reply chips, live-agent mode badge, reset button, keyboard and click input, `aria-live` for accessibility

## Design note: why deterministic logic instead of an LLM

This bot intentionally uses rule-based intent matching rather than a large
language model, for three reasons aligned with the brief:

1. **Testability** — the requirements state the bot must be reviewable with no
   API keys, subscriptions, or accounts. Every LLM lives behind an API key;
   deterministic logic runs anywhere, forever, for free.
2. **Exactness** — order statuses and policies must follow the provided data
   *exactly*. Rules guarantee that on every run; a generative model can only
   promise it probabilistically.
3. **Fit** — for a bounded support domain (4 use cases, fixed policies), guided
   flows are the industry-standard architecture (the same model as Botpress or
   Dialogflow flows).

The architecture is built to grow, though: the `handle()` router cleanly
separates intent detection from flow logic, so a production version could swap
the keyword matcher for an LLM classifier (e.g. Claude) for richer natural
language understanding, while keeping the deterministic flows and mock-data
guarantees intact.

Built by Mark Zaki · Kelson Works
