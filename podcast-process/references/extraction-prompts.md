# Extraction Prompts

## Generic prompt (no hint from user)

> You are extracting useful content from a podcast transcript. Your job is to surface things the listener would want to remember — not to summarise the episode.
>
> Extract in these categories only:
>
> - **Insight** — a non-obvious argument, counter-intuitive claim, or mental model presented in the episode
> - **Reference** — a book, person, paper, tool, library, framework, or specific fact worth remembering
> - **Action item** — a concrete, do-able thing the listener could try
> - **Counter-argument** — a place where the host or guest is likely wrong or is missing a perspective
> - **Quote** — a verbatim line that is striking or memorable (≤ 40 words)
>
> Each item must be:
> - Specific and self-contained (readable without the transcript)
> - Grounded in what was actually said (not extrapolated)
> - Non-obvious — skip things that are common knowledge or just filler
>
> **Do NOT include:**
> - Banter, introductions, or sponsor reads
> - Things that only make sense in the context of the specific conversation
> - Vague generalities like "always be learning"
>
> **Output as JSON:**
>
> ```json
> {
>   "summary": "2-3 sentence plain-language summary",
>   "items": [
>     { "category": "Insight", "content": "...", "confidence": "high" }
>   ]
> }
> ```
>
> Confidence: `high` (clear, explicit in transcript) / `medium` (implied but not stated). Skip `low`.

## Focused prompts (user gave a hint)

When the user says "extract health info" / "focus on X" / "only AI stuff", tune the prompt:

```
Focus exclusively on items related to <HINT>. Skip insights that don't touch <HINT>, even
if they're interesting in other ways. If there's nothing about <HINT>, return an empty
items array — don't pad.
```

Add this to the generic prompt, after the "Do NOT include" section.

## Category definitions (more detail)

### Insight

The most valuable category. A non-obvious argument, counter-intuitive claim, or mental model. Not a fact (that's a Reference).

*Good:* "The reason MVPs fail isn't lack of features — it's that founders can't tell whether a weak signal is a bad product or a bad audience."

*Bad:* "Startups are hard."

### Reference

Something specific you could look up, read, or use. Books, people, papers, tools, libraries, frameworks, specific facts with a source.

*Good:* "pg-boss — Postgres-based job queue, good if you already have Postgres and don't want Redis."

*Bad:* "They mentioned some trading bot." (no name, no source)

### Action item

A concrete thing the listener could do. Specific and small enough to actually do.

*Good:* "Check your AWS bill for NAT Gateway charges — they're often 10x what people expect."

*Bad:* "Improve your infrastructure."

### Counter-argument

A place where the host or guest is likely wrong or is missing a perspective. This category is often empty. Only include if the counter is genuine and worth thinking about — not just disagreement.

*Good:* "The guest claims RSS is dead but ignores that podcast subscriptions are 100% RSS-driven — the category is bigger than he admits."

*Bad:* "I don't agree with their take." (no specificity)

### Quote

A verbatim line (≤ 40 words) that captures an idea memorably. Usually from the guest, sometimes from the host.

## Tuning notes

- If output is too noisy, tighten the "Do NOT include" list
- If output misses good stuff, widen category descriptions
- Test on an episode the user knows well — they should recognise the extraction as "yes, those are the points I would have flagged"
- If the user's extraction hint is very narrow (e.g. "only if they mention a specific startup"), consider skipping generic insights entirely to save output size
