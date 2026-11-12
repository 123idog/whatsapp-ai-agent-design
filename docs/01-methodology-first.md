# 1. Methodology Before Prompts

The most common reason a sales bot gets switched off isn't a technical bug. **The bot sells differently from the team.** It asks the wrong questions, in the wrong order, and hands off too early or too late. Agents stop trusting it, and customers notice.

The fix isn't a better prompt. It's writing down the **sales methodology** first, with the people who sell.

---

## The build order that works

```
1. Methodology   →  2. Approve real conversations  →  3. Build  →  4. Replay & compare  →  5. Live
   (with the team)     (with the team)                  (bot)       (same conversations)
```

### 1. Write the methodology with the team

Run a **structured questionnaire** with the sales lead, as a web page they fill in on their own time, not a meeting. Typical sections:

- **Opening:** how to introduce yourself, and whether to disclose that it's an AI (and when)
- **Qualification questions:** which ones, in what order, and which are mandatory before a handoff
- **Hot-lead signals:** what makes a lead urgent (a firm purchase date, a large group, a high-intent source)
- **Disqualification:** when a lead is *not* a fit, and who decides
- **What the bot must never do:** quote prices, ask about budget, promise callbacks at a specific time
- **Forbidden words and tone**
- **Good and bad example conversations.** These are the most valuable answers and the ones most often left blank. Chase them.

Run it in **rounds**. Round 1 answers raise round-2 questions. Every answer is stored with who answered and when, so later disputes are settled by the record, not by memory.

### 2. Approve real conversations

Take a sample of **real past conversations** (anonymised), grouped by topic, and show them in a WhatsApp-style review page. For each conversation the team marks: *this is right* / *this is wrong, here's what should happen*.

This turns abstract rules into concrete cases, and surfaces rules nobody thought to write down.

> **Don't skip step 1 to get here faster.** Approving raw conversations without written rules was tried and rejected: every reviewer applied a different method, and the bot inherited the inconsistency.

### 3. Build

The prompt is now a translation of an approved document, not a guess.

### 4. Replay and compare

Run the new bot over the **same conversations** the team approved, using a **customer simulator** that replays the real customer messages turn by turn. Diff the bot's behaviour against the approved outcomes. Fix, replay, repeat.

### 5. Go live, with the old version as rollback

Keep the previous version deployable. Expect the first week to produce new methodology questions. Route them back to the team through the same questionnaire page.

---

## Changes go through the same loop

When a salesperson says "the bot should have done X here", that isn't a prompt edit. It's a **methodology change**:
1. Write the proposed rule.
2. Put it on an approval page with the alternatives and a recommended option.
3. **Implement only after the team votes.**
4. Record the reason in the CRM note whenever the bot applies the rule, so agents can see *why* it acted.

This is slower than editing a prompt, but it's the only approach that keeps the sales team owning the method.

---

## Cloning to a new market or product

A second bot for a new market or product line starts as a **clone of a proven methodology**, not from scratch:
- Same questions, same order, same handoff logic.
- A **short questionnaire for that team only on the differences**: local specifics, seasonality, products that don't exist in the original.
- Patch only what the answers change.
