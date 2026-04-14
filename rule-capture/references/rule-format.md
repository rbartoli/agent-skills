# Rule Format Spec

## Anatomy

```markdown
- **<rule in imperative, one sentence>** (YYYY-MM-DD)
  - Why: <reason>
  - Context: "<short verbatim quote, ≤120 chars>"
```

## Rules for the rule phrasing

1. **Imperative voice.** *"Do X"* or *"Never do X"*. Not *"I should do X"* or *"The agent does X"*.
2. **One sentence, one behaviour.** If you need `and` to join two requirements, split into two rules.
3. **Concrete.** *"Run `date` before stating the date"* beats *"Be careful with dates"*.
4. **No jargon without definition.** If the rule uses project-specific terminology, either define it inline or link to the defining note.
5. **No references to specific values that'll rot.** *"The API key is in `.env`"* becomes useless when the key rotates. Instead: *"API keys live in `.env`, never inline."*
6. **Positive over negative when possible.** *"Do X when Y"* is clearer than *"Don't do Z unless W"*.

## Good vs bad examples

| Bad | Good |
|-----|------|
| Be careful with files | Before deleting a file, show its contents first |
| Use bun | In this project, use `bun`, not `npm` or `pnpm` |
| The date thing | Always run the `date` command before stating today's date |
| Don't be verbose | Keep chat-between-tool-call updates to ≤25 words |
| User prefers X | Default to X in this project unless told otherwise |

## The "Why" line

Gives the reader (future you, or the agent next session) enough context to judge edge cases.

**Example:**

```markdown
- **Always confirm before force-pushing to shared branches.** (2026-04-14)
  - Why: force push on main destroyed teammate's in-progress work last quarter
  - Context: "never again — you almost nuked Lisa's branch"
```

If a future context arises that the rule doesn't cover exactly, the "Why" helps Claude judge whether the rule applies.

## The "Context" line

A short quote from what triggered the rule. Traces the rule back to its origin for later audit. Max ~120 chars, can be a paraphrase if the original was very long.

If the user dictated the rule explicitly ("from now on always X"), the Context line is redundant — you can omit it or just quote the user.

## Merging rules

When capturing a new rule, scan existing rules. If one covers the same topic, don't append — merge.

**Example scenario:**

Existing rule:
```markdown
- **Use Server Components by default in this Next.js project.** (2026-03-10)
  - Why: reduces client JS, improves perf
```

New correction: *"Remember, 'use client' should only go on interactive components."*

Don't add a new rule. Extend the existing one:

```markdown
- **Use Server Components by default; add 'use client' only for interactive components.** (2026-03-10, updated 2026-04-14)
  - Why: reduces client JS, improves perf; 'use client' is opt-in per the RSC model
```

## Pruning

The skill itself doesn't auto-prune. But during capture, flag rules older than 6 months:

*"There's a rule from 2025-10-03 about this — keep, replace, or merge?"*

This keeps CLAUDE.md from accumulating stale guidance.
