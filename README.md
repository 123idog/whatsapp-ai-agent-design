# WhatsApp AI Agent Design

**How to design, build and operate AI agents that talk to real customers on WhatsApp, qualify them, and hand them to a sales team, based on bots running in production in Hebrew, Arabic and English.**

Building a bot that answers questions is easy. Building one a sales team trusts is hard. It has to know when to stop talking, never invent a price, never leave a customer in silence, never message someone who didn't write, and leave the CRM in a state a human can pick up in seconds.

This repo is the design method and the production patterns behind that.

---

## Contents

| # | Chapter | What's inside |
|---|---|---|
| 1 | [Methodology Before Prompts](docs/01-methodology-first.md) | Write the sales method with the team, approve real conversations, then build |
| 2 | [Conversation Design](docs/02-conversation-design.md) | One question at a time, language detection, what the bot must never ask |
| 3 | [Production Architecture](docs/03-architecture.md) | The turn pipeline, durable memory, debounce, "the code holds the floor" |
| 4 | [Output Guards](docs/04-output-guards.md) | Invented prices, printed tool calls, fake promises, retry on a different model |
| 5 | [Human Handoff & CRM State](docs/05-handoff-and-crm.md) | Sales handoff vs. escalation, mute on takeover, status rules, follow-ups |
| 6 | [Classification Traps](docs/06-classification-traps.md) | The gate with no exit, veto rules, Arabic regex pitfalls |
| 7 | [Reliability & Continuous Learning](docs/07-reliability-and-learning.md) | Safety nets, zero-touch monitoring, mining human answers into the knowledge base |

---

## The bot's job, in one sentence

> **Answer what it knows, collect what sales needs, hand off at the right moment, and leave a clean record.** Everything else is a feature request.

## Design principles

1. **Methodology first.** The bot encodes how your best salesperson works. If that isn't written down, the bot can't learn it from prompts.
2. **The prompt guides, the code guarantees.** Every hard rule (prices, statuses, who gets messaged) is enforced in code after the model.
3. **Silence must be a decision, never an accident.** Every path that ends without a reply needs a reason that can be traced.
4. **Never touch what a human decided.** A status, an owner or a conversation a human took over belongs to the human.
5. **Measure in conversations, not executions.** A green run that left a customer waiting is a failure.

---

## Stack these patterns come from

WhatsApp Business API (via a BSP) · Messenger / Instagram · n8n · Gemini / OpenAI (primary + fallback models) · Salesforce · Data Tables · Telegram / WhatsApp groups for team alerts · Langfuse.

## About

Written by **Ido Guy**, AI & Automation Engineer. I design end-to-end AI solutions for sales and operations teams, including production AI agents in Hebrew and Arabic.

[LinkedIn](https://www.linkedin.com/in/ido-guy) · Also: [n8n Production Playbook](https://github.com/123idog/n8n-production-playbook) · [Claude Code for Automation Engineers](https://github.com/123idog/claude-code-for-automation-engineers)

*All examples are generalised. No client data, prompts, credentials or identifiers are included.*
