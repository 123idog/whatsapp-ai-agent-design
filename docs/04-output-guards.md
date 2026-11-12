# 4. Output Guards

Between the model and the customer sits a Code node whose only job is to answer: **can this reply be sent as it is?** It can pass the reply, repair it, retry it, or replace it with a safe handoff.

---

## Guard 1: invented prices

**Incident:** a customer asked about a specific tour. The model quoted prices that sounded plausible and were both wrong. The catalog had different figures.

**Guard:** extract every number that looks like a price from the reply and verify it against the catalog entries the tool actually returned in this turn. Any figure that isn't on the card fails the turn.

**Stray-figure variant:** a reply that mentions a sum of money with no catalog context ("it'll be around…") is replaced with a handoff. In production this fired on under 1% of turns, and every one of those was a reply that shouldn't have gone out.

---

## Guard 2: printed tool calls and leaked reasoning

**Incident:** a customer asked a question the bot could fully answer. The model wrote:

```
Sure, happy to help :)
One moment, checking which parks we have.tool_code:print(search(destination='...', category='...'))
```

It **printed** the tool call instead of invoking it. Another time it emitted thousands of characters of English "Thinking Process:" notes.

**What didn't work:** the prompt already said "never write a tool call as text". Adding another wording would have been theatre. This was model degeneration, not an instruction gap.

**Guard:** a regex for tool-call syntax, code fences, JSON fragments, internal tags and planning-text markers. On a match, **retry**.

---

## Retry, on a different model

```
Agent → Check Output → needs_retry? ─yes→ Agent-Retry (different model) → Parse
                                     └no──────────────────────────────→ Parse
```

- `Check Output` runs **all** checks (scaffolding regex and price verification) on the raw output. It uses **the same functions** as the final parser, so the retry decision and the block decision can't drift apart.
- The retry gets the same prompt plus one line naming what went wrong.
- **Use a different model for the retry**, because every failure seen came from the primary.
- If the retry also fails, send a safe handoff.

**Bug the retry shipped with:** at the retry node, `$json` is the *first agent's output*, so the inherited `{{ $json.text }}` was empty and the retry never saw the customer's question. It was caught only by a **forced clone**: a copy where the retry condition is always true, so every turn goes through the rare branch. Do this for every branch that depends on the model misbehaving.

---

## Guard 3: promises the business can't keep

- **Fake agent promises:** "an agent will call you in 5 minutes" gets the time stripped.
- **Wrong department:** a customer-service handoff can't promise a sales agent, and the reverse.
- **Invented staff names:** any name presented as a colleague is checked against the roster.
- **Invented emergencies:** a model escalating "urgently" on a routine message needs a guard tied to real urgency signals, not the model's tone.

---

## Guard 4: invented operational facts

Asked "are you open on Saturday?", a model will confidently invent opening hours. Facts like hours, holidays and policies come from **config injected by code**, never from the model's memory. A holiday table drives both the bot's answers and the follow-up scheduler. Keep a single copy, or keep the copies synced.

---

## Guard 5: language integrity

Flag replies in the wrong script for the conversation, or containing replacement characters (`???`, `�`). These are cheap deterministic checks and catch encoding bugs before customers see them.

---

## Guards must fail safe and be visible

- **Fail safe:** when a guard can't decide, prefer a handoff over sending.
- **Log every guard action** with a reason (`price_offcard`, `scaffolding`, `name_guard`). From the customer's side, every guard ends the same way ("let me pass you to a colleague"), so the reason column is the only way to count which bug actually caused it.
- **Track guard rates over time.** A rate climbing from 0.5% to 11% in two days is a model degrading, and you want to see it before your customers do.
