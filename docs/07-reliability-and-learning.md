# 7. Reliability & Continuous Learning

A production bot fails in two ways that never show up as an error: **it doesn't reply**, or **it replies badly**. Both need dedicated systems.

---

## The reply safety net

Silence has many causes: a delayed webhook, a lost webhook, a saturated runner, a crash after the reply was composed. Fixing each cause is necessary but never complete. The safety net catches **the outcome**: a customer's last message was never processed.

### The key design decision

> **The net never writes a reply and never decides whether one is needed.** It re-fires the *original webhook*, so the turn goes through every gate the bot already has: mute, human takeover, handoff, debounce, guards. If the bot chooses silence, silence is what happens.

The net answers only one question: **"was this turn processed at all?"**

### Gates where silence is correct

- A human agent messaged in the last 24 h
- The bot's last turn was a handoff
- Already recovered (dedup by message ID)
- Test numbers
- Message age outside a window (e.g. 4–90 min)
- The bot hasn't been active in this conversation (not its thread)

Caps: N conversations examined and **at most 5 recoveries per run.**

### Lessons from building it

- **Candidates have to come from the provider, not the bot's own log.** A net that only looks at conversations the bot logged is blind to conversations the bot never managed to answer, which is exactly what it exists for.
- **Outbound broadcasts look like inbound messages** in some provider APIs: no `owner` field, text in a different property. Without explicit guards (`eventType === 'message'`, `owner === false`, text present), the net would message someone who never wrote. Caught in a dry run.
- **Being conservative is correct.** Over a full day of normal traffic it should fire zero times. Its job is the rare dropped turn.
- A previous version ran green for a week and **rescued nobody**. An HTTP node had replaced the item and the phone field was gone. Always make a monitor prove it catches a **known-bad case** before trusting its silence.

---

## Zero-touch quality monitoring

Everything needed to judge a turn is already inside the bot's execution data: the input, the reply, the intent, the state flags. A **separate monitoring workflow** reads recent executions and flags silent failures **without adding a single node to the live bot.**

Deterministic signals first:
- **Language integrity:** wrong script, `???` runs
- **Errors and guard firings**, as rates over time
- **Volume against a baseline**
- **Escalation rate as a metric, not a finding.** Handing off is often intended. A spike is the signal.

Then a daily or weekly **LLM clustering** pass over the logged failures groups them into recurring patterns ("escalates on simple logistics questions") instead of one-off events, and proposes fixes. Use a **strong model** for this, since weak models produce vague clusters.

> Executions get pruned after a few days. Persist failures to a table, or "how often has this happened?" becomes unanswerable.

---

## Mining human answers into the knowledge base

When the bot hands off and a human answers, **that answer is knowledge the bot didn't have.**

```
Daily:
  log rows (customer question + human agent answer, last 24h)
  → pair, clean, dedupe
  → one strong-model call with the CURRENT prompt + already-proposed topics
  → candidate facts (upsert by topic key, scored)
  → top 3 sent as cards to the bot owner: ✅ approve / ❌ reject / 🕓 snooze 3 days
Approved → added to the prompt → replay → live E2E → marked implemented
```

- **Facts only**, not sales phrasing. Agents' wording isn't always what you want the bot to copy.
- **Rejected topics are never re-proposed.** Snoozed ones come back later.
- **Hard-exclude** price claims and non-answers. The first run tried to promote a competitor price comparison.
- Max 3 cards a day keeps the review habit sustainable.

---

## Shadow mode for new capabilities

Before a bot takes over a new kind of request, run it in **shadow**: it composes answers but doesn't send them, and those answers are compared with what humans actually replied. Promote it when the agreement rate is good enough to defend to the sales team, measured on **real traffic**, not a small replay set.
