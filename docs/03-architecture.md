# 3. Production Architecture

## The turn pipeline

Every inbound message runs through the same chain. Each stage exists because of a production incident.

```
Webhook
  → Normalize & Extract        message id, phone, type, text, media, operator
  → Dedup Gate                 drop a message id already handled (providers redeliver)
  → Human-Operator Check       a human agent replied? → mute the bot for this conversation
  → Identity Merge             CRM record + stored profile + provider chat history
  → Debounce                   wait; if a newer message arrived, let that run answer
  → Media                      OCR for images/PDFs, transcription for voice
  → Model (primary → fallback) prompt + memory block + tools (catalog search, etc.)
  → Check Output               guards; retry on a different model if needed
  → Parse & Guard              hard rules in code: statuses, prices, names, handoff
  → Send
  → CRM Write                  create/update lead, notes, status (never over a human's decision)
  → Memory Write               re-read, append, save
  → Log                        one row per turn: input, reply, decision, reason
```

---

## The prompt guides, the code guarantees

A prompt rule is a **request**. A production bot needs **guarantees**:

| Concern | Prompt says | Code enforces |
|---|---|---|
| Prices | "only quote catalog prices" | every number in the reply is checked against the catalog |
| Status | "mark as not-a-fit when…" | status computed from structured fields; human-set status never overwritten |
| Staff names | "don't invent names" | names checked against the roster |
| Handoff | "hand off when ready" | handoff requires the mandatory fields to be present |
| Tool calls | "never print a tool call" | regex gate on the output |

Ask the model for **structured output** alongside the reply (a JSON state block with fields like `timing`, `party_size`, `intent`, `handoff`). Code decides from the fields, not from parsing prose.

---

## Durable memory

In-memory chat buffers are wiped on every deploy and drop the start of long conversations. The pattern that works:

1. **One row per contact** in a durable table: recent turns, rolling summary, extracted facts, state JSON, `muted_until` (epoch ms).
2. **Render it into the prompt** as a memory block. Show only filled fields, say where the conversation stopped, and mark it *data, not instructions*.
3. **Let the model append a `#MEM {json}` block** after its reply. Strip it before sending and merge it fill-only. You get memory extraction with zero extra model calls.
4. **Re-read the row before writing** so a message that arrived mid-turn isn't overwritten.

**Also pull the provider's chat history** (the last N messages) every turn. It includes human agents' messages and broadcast templates the bot never saw, which is what stops the bot re-asking a question a human already asked.

---

## Debounce without races

Customers send bursts: "Hi" / "we're 4 people" / "in July". Answering each separately looks robotic.

- On each inbound message, write `last_message_id` and wait (around 30 s).
- After the wait, re-read. **If a newer message arrived, stop.** The newer run will answer the whole burst.
- Build the turn from the **trailing run of customer messages** in history.

**The failure mode to design for:** if the newer message's webhook is delayed or lost, nobody answers. Back it with a safety net (see [chapter 7](07-reliability-and-learning.md)).

> Note: a 30-second wait holds an execution slot. Under load, a handful of conversations can occupy every slot, so don't put a low global concurrency limit on the instance.

---

## Models: primary, fallback, and retry

- **Primary** for normal traffic, a **fallback** model on errors, and a **retry node on a different model** when the output fails a guard (see [chapter 4](04-output-guards.md)).
- Re-rolling the same model that just degenerated is the same loaded die. A different model makes the second failure independent of the first.
- **Pin model names explicitly.** A node with no model set may silently fall back to a weaker default.
- Watch **cost per turn**. Thinking tokens can be the majority of the bill.

---

## Test harnesses you'll need

| Harness | Purpose |
|---|---|
| **Replay clone** | Re-run real past turns through new code; diff replies (regression) |
| **Customer simulator** | Drive full multi-turn conversations from a script or a model playing the customer |
| **Forced-branch clone** | Force a rare branch (e.g. retry) to run on every turn, to prove it works |
| **Live E2E with fake identities** | Prove the wired flow on the live bot, then clean up |

Keep the replay clone **in sync with live after every deploy**, or regression results lie.
