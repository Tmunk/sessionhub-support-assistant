# SessionHub Support Assistant

A self-serve support demo: a small knowledge base paired with an AI assistant that only answers from that knowledge base, cites its source, and flags anything it can't resolve for a human agent instead of guessing.

## Why this exists

Built as a follow-up to real work automating B2C self-serve support (Zoho Desk automations, knowledge base content, and contributing ticket data to an internal support chatbot) at a SaaS EdTech company. This is an independent, from-scratch build exploring the same problem: how to safely deflect support volume without letting an AI make things up when it doesn't actually know the answer.

The original B2C work followed a pattern that shows up across several projects in this portfolio: at a small company, engineering was consistently overwhelmed with higher-priority work, so self-serve support tooling was something the CS team had to build and maintain itself rather than wait on. That automation reduced inbound ticket volume without needing engineering time, freeing engineering to stay on urgent product issues and freeing the rest of the CS team to focus on higher-touch account work instead of repeat tickets.

## How it works

- 8 short knowledge base articles for a fictional platform called SessionHub
- With an API key set, every question is sent to Claude (Anthropic's model) with instructions to answer using only those articles
- The model returns structured output: an answer, which article(s) backed it, and whether the question should be escalated to a human instead
- Grounded answers show a clickable citation chip linking to the source excerpt
- Anything outside the knowledge base — refund disputes, security concerns, anything sensitive — gets flagged with an escalation banner instead of a guessed answer
- A running counter tracks answered-from-KB vs. escalated-to-human, which is the actual metric real support/CS Ops teams are measured on for self-serve tooling

### Without an API key

Rather than blocking every question until you bring a key, the page falls back to a plain keyword search against the 8 articles — no AI, no cost, works instantly for anyone. It's tagged **Keyword match (no AI)** in a neutral gray, deliberately different from the teal/gold AI-derived states, because it's a genuinely different mechanism: word overlap, not understanding. It can't paraphrase, and it can't reliably judge nuance the way the real model does — "how do I reset my password" and "can I get a refund" might share zero useful keywords, so a keyword-only match is a rough approximation of the assistant, not a replacement for it. Add a key for the real thing.

## Try it live

Enable GitHub Pages on this repo (Settings → Pages → deploy from the `main` branch, root folder) and it'll be live at:
`https://<your-username>.github.io/sessionhub-support-assistant/`

To ask a question, paste an Anthropic API key into the box at the top of the page. It's stored only in that browser's `localStorage`, sent directly to `api.anthropic.com`, never to any third party, never committed to this repo — the same deliberate, disclosed simplification used in the Client Health Dashboard project: a real product would proxy this through a backend so the key never touches the browser.

## Known limitations & fixes

This was originally prototyped inside Claude's artifact sandbox, which quietly provides things a real deployed page doesn't have. Two real bugs from that came with it and were fixed for this version:

- **Missing API key entirely.** The original `fetch` call to `api.anthropic.com` had no `x-api-key` header at all — it only worked because the sandbox proxies that call for you. Once deployed as a real file, it would fail silently. Fixed by adding the key-entry UI described above.
- **`window.storage` isn't a real browser API.** Conversation history was saved with `window.storage.get/set`, which only exists inside Claude's artifact runtime. Outside it, every save silently failed (wrapped in a try/catch that swallowed the error), so the "Clear conversation" button and persistence implied by `STORAGE_KEY` never actually worked in a real browser — each reload just started empty. Fixed by swapping to real `localStorage`.

Separately, each question is still answered independently — the assistant doesn't retain memory of earlier questions in the same session. That one's a deliberate simplification, not a bug.

## Built with

Vanilla HTML, CSS, and JavaScript. No framework or build step. Uses the Anthropic Claude API directly from the browser.
