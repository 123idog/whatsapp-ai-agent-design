# 2. Conversation Design

WhatsApp is a phone-first, interrupt-driven channel. Customers answer while doing something else, send three short messages instead of one, switch language mid-conversation and disappear for a week. Design for that.

---

## One question per message

A first message with three questions felt like a form to customers, and they abandoned it.

- **One question per outbound message.**
- A short **micro-acknowledgement** between answers ("Thanks 👍").
- For structured flows (recruitment, surveys), a light **progress indicator**: "Question 3 of 8".
- Use buttons only for genuinely closed questions. Open questions stay as text.
- **Store state per question** (one field per answer), not per batch.

---

## Order matters more than wording

The order of qualification questions decides outcomes. Example from production: the bot decided whether a lead was a fit **before** asking about group size, so large groups (often the most valuable leads) were disqualified before the bot learned they were large. Moving the decision to *after* the composition question fixed a whole class of wrong statuses.

**Rule:** a qualification decision can only run once every input it depends on has been asked.

---

## What the bot must never ask

Decide this with the sales team, and enforce it in code:
- **Budget.** Asking feels like a filter and is the salesperson's job. The bot can describe the positioning ("we work with 4–5★ properties") and let the customer react.
- **Personal data the CRM already has.** Check identity first.
- **A question it already asked.** Re-asking is the most visible sign of a bot. It usually comes from history the model can't see (see [chapter 3](03-architecture.md)).

---

## Language: follow the customer

Multilingual markets mean a Hebrew-market bot gets English conversations, and an Arabic-market bot gets dialect, transliteration and typos.

- **Detect the conversation's language from the customer's recent messages**, not from the market or the first message. Strip URLs before detecting.
- **Short replies are language-neutral.** "Yes", a date, "Hi" or a number shouldn't trigger a language switch or a "wrong language" guard.
- **Fixed texts** (handoff lines, follow-ups) need versions in every supported language. A Hebrew template inside an English conversation breaks the experience.
- **Native, not translated.** Prompts are written per language and market. A prompt translated from English produces translated-sounding replies.

---

## Disclosure and identity

- Decide explicitly whether the bot introduces itself as an AI assistant, and **when** (opening vs. only if asked). Teams change their minds on this, so make it a single switch.
- Use a consistent persona name. **Never let the model invent staff names.** Validate any name in a reply against the real staff roster in code.

---

## Handoff text is a promise

"An agent will call you in 10 minutes" is a commitment the bot can't keep.
- Handoff messages say **what will happen**, never **when**, unless the system can guarantee it.
- If the customer was handed to a customer-service rep, don't also promise a sales agent, and the reverse.
- **After a handoff, the bot keeps helping** with factual questions instead of repeating "an agent will get back to you".

---

## Abandonment is a design input

Customers who don't have a firm plan stall far more often than customers who do. Design follow-ups for each:

| Situation | Follow-up |
|---|---|
| Customer stopped mid-qualification | One gentle nudge, then stop |
| "Not now, maybe next month" | Agree on a time and follow up then |
| "Don't know when" | Periodic check-in with a cap (e.g. every 2 weeks, max 3) |
| Hot lead handed to sales, no contact after N hours | Approved template to the customer + task for the agent |

Respect quiet hours, weekends and **local public holidays**, and send outside the 24-hour window only with approved templates.
