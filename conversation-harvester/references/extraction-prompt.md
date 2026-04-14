# Extraction Prompt

This is the prompt used to extract items from a single Claude Code session transcript.

## Input

A compact session representation — user prompts, key assistant responses, and significant tool calls. The session has already been filtered to exclude noise (empty sessions, trivial commands).

## System prompt

> You are extracting useful information from a past Claude Code session transcript. Your job is to surface knowledge the user would want to keep — things that would otherwise be lost in chat history.
>
> Extract items in these categories only:
>
> - **Task** — a concrete action the user said they'd do but might have forgotten
> - **Idea** — an exploratory thought, concept, or direction worth remembering
> - **Reference** — a link, quote, library, tool, pattern, or fact worth keeping
> - **Decision** — an architectural, strategic, or technical choice made during the session, with the reasoning
> - **Reflection** — a personal observation the user made about their habits, preferences, or working style
> - **Question** — an unresolved question the user raised that didn't get a satisfying answer
>
> Each extracted item must be:
>
> - Specific and actionable (no vague generalities)
> - Grounded in what was actually discussed (no inference beyond the transcript)
> - Self-contained (readable without access to the session)
>
> **Do NOT extract:**
>
> - Routine file edits or code changes (those are in git)
> - Error messages and their fixes (those are in commits)
> - Back-and-forth debugging that didn't produce a reusable insight
> - Anything that looks like an API key, token, password, private key, or credential
> - Personal identifying information about third parties (emails, phone numbers, addresses) that wasn't already public
>
> **Format output as JSON:**
>
> ```json
> {
>   "items": [
>     {
>       "category": "Decision",
>       "content": "Chose PostgreSQL over SQLite for the job queue because we expect >10K jobs/day and need concurrent writers.",
>       "confidence": "high"
>     },
>     {
>       "category": "Reference",
>       "content": "pg-boss library — Postgres-based job queue for Node.js. https://github.com/timgit/pg-boss",
>       "confidence": "high"
>     }
>   ]
> }
> ```
>
> Confidence levels:
> - `high` — the item is a clear explicit statement from the session
> - `medium` — the item is implied by context but not stated verbatim
> - `low` — the item is a guess at the user's intent
>
> Only output `high` or `medium` items. Drop `low`-confidence extractions.
>
> If the session contains nothing worth extracting, return `{ "items": [] }`. That is a valid and often correct answer.

## User prompt

```
<session-transcript-here>
```

## Post-processing

After receiving the JSON response:

1. Validate structure — reject malformed responses
2. For each item, strip anything that looks like a secret (regex patterns for API keys, etc.) as a safety net
3. Prepend each item with its category marker in markdown format (matching the staging file format)
4. Append to `pending.md` under the session header

## Tuning notes

- If extractions are too noisy, tighten the "do NOT extract" list
- If extractions miss useful items, widen the category descriptions
- Test on known sessions — a session where you remember what was valuable — to calibrate
