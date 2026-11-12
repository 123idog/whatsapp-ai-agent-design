# 6. Classification Traps

Most bots route each message by intent first: a sales inquiry, a job applicant, support, spam. Routing errors are rare, but each one can end a sale.

---

## The gate with no exit

**Incident:** a sales bot also handled job applicants. After five clean sales turns, a customer sent one garbled message (fast typing, missing letters), and the classifier labelled it **"job"**.

- The bot sent the job-campaign text and set `is_recruitment = true`.
- The customer replied "what job?!" and got the **same text again**.
- The customer wrote "we were thinking about a trip" and got **total silence**, permanently.

**The real bug wasn't the misclassification.** The "in recruitment mode?" gate's true branch was **wired to nothing**, and nothing ever cleared the flag. One bad turn locked the conversation forever.

**Rules:**
- **Every sticky flag needs an exit.** A later message that clearly contradicts the flag must override it.
- **No branch ends in silence by accident.** Audit every gate output. An unconnected output is a silent drop.
- Measure the blast radius: misclassification was around 0.5% of turns, but each one cost a whole customer.

---

## Veto rules beat a better classifier

**Fix:** a deterministic **veto** ahead of the LLM classifier. A message with clear domain words (travel, booking) and **zero** employment words can't be classified as a job, and the LLM isn't consulted. The gate then reads `is_recruit && !is_travel`.

Cheap, explainable and testable.

---

## Build the corpus before the regex

- A naive veto would have **broken real job leads** who mentioned the destination country ("heard you're hiring for tourism in …"). That only showed up because the fix was tested against a **corpus of every real job message**, collected first.
- A missing word in the job lexicon accounted for **a meaningful share of real applicants**. Without it, the veto would have swallowed genuine candidates.

**Rule:** collect real positive and negative examples first, then write the regex, then run it against both.

---

## Arabic (and Hebrew) regex pitfalls

- **`\b` doesn't work for Arabic script** in most regex engines. Anchor stems with `(?:^|[\s،.,!?؟\n])`.
- **Bare stems overmatch.** The stem for "worker" matched inside words meaning "usage" and "business".
- **Typos change meaning.** A common dropped-letter typo turned "three families" into something that matched "workers". A loosened lexicon then sent the job text to a honeymoon customer.
- **Phrase-based tokens beat stems.** The original lexicon used deliberate phrases. "Improving" it into bare stems caused a regression two days later.

**Rule:** when editing someone else's classifier lexicon, assume the narrowness was deliberate. Widen only with corpus evidence, and re-run the whole corpus after every change.

---

## Auto-replies and noise

Business auto-responders ("Thank you for contacting…, our office is closed") look like customers. Detect them early, from the incoming text, before any expensive step, and don't reply to them.

---

## Signals that look reliable but aren't

"The bot has messaged this contact before" seemed like proof of an active sales conversation, and a good gate. Measurement showed **job applicants had it set on their very first message**, because of an outbound campaign. **Measure a signal against real data before gating on it.**
