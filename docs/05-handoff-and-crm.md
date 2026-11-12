# 5. Human Handoff & CRM State

The bot's output that matters most isn't the chat. It's **the CRM record a salesperson opens next.**

---

## Two kinds of handoff

| | Sales handoff | Escalation |
|---|---|---|
| **When** | Lead is qualified and ready | Bot can't or shouldn't handle it (complaint, complex request, explicit human request) |
| **Goes to** | Sales agent / owner of the lead | Customer-service rep on duty |
| **CRM** | Status → "qualified" with a summary | Task / alert, status unchanged |
| **Alert** | Normal assignment | Real-time alert to the duty channel |

Treating every sales handoff as an escalation floods the duty channel and trains people to ignore it. Keep them separate.

**Hot-lead alerts:** when a lead crosses a score threshold (a firm purchase date, a large group, a high-intent source), push an alert to team leads in the CRM and in the team's chat group.

---

## Mute on human takeover

When a human agent replies in the conversation, the bot must stop.

- Detect human messages from the provider's operator field, and **distinguish real people from API tokens and the bot's own identity.** A bot whose own messages count as "human" mutes itself forever.
- The **assigned** agent and the agent who **actually** wrote may differ. Read the right field.
- Set `muted_until` (epoch ms) and let it **expire**. A permanent mute leaves returning customers unanswered months later.
- **Agent-initiated** conversations (the agent writes first) should mute too.

---

## Status rules: the bot proposes, humans own

Rules agreed with the sales team, enforced in code:

1. **Never overwrite a status a human set.** Track who set the current status. If it was a person, the bot doesn't touch it.
2. **"Not a fit" requires all of its inputs.** No disqualification while timing, group size or intent is still unknown, and never after the lead was handed to a person.
3. **Record the reason.** When the bot disqualifies, it prepends a one-line reason to the CRM note: `❌ Marked not-a-fit by bot (date): reason`.
4. **Reversible.** If new information arrives (the customer asks for a call, reveals a large group), the bot moves the lead back to "working" and notes it: `↩️ Returned to working (date): reason`.
5. **Strong buying signals override missing details.** A customer who has already committed (e.g. bought the main product elsewhere, buying within 2 weeks) goes to sales immediately, even with qualification gaps.

---

## Identity: one person, many records

- Phones come in many formats (`+972 58-…`, `05…`, `972…`). **Normalise to digits** and match on the last 9.
- Returning customers: if the last conversation was weeks ago and the profile predates the gap, **re-confirm** key details instead of trusting them.
- **Converted leads are frozen** in many CRMs. Write to the converted record (opportunity/contact) instead, or every update fails silently.

---

## Follow-ups as a state machine

Follow-ups run as a small table plus a scheduler, not as long Wait nodes:

```
follow_ups: phone, kind (postpone | stopped | hot_6h | ...), stage, status,
            due_at, ref_at, attempt_count, stop_reason, last_sent_at
```

- The bot **arms** a follow-up from the turn's structured output ("customer said: next month").
- A scheduler (off the round minutes) sends the due ones, **inside allowed hours, skipping weekends and holidays**.
- Close automatically when the reason disappears: an agent called (check the phone system), a human messaged, the lead converted.
- **Cap attempts** and expire overdue items. Never let a stale follow-up fire weeks late.

---

## "Has someone contacted me?"

Customers write "nobody got back to me". To answer correctly the bot needs to know whether **a human messaged after the handoff**. The handoff message itself doesn't count, even when a human sent it. Compute `human_after_handoff` from chat history and route complaints about unreachable agents to the duty channel.
