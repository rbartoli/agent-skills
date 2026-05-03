---
name: rule-capture
description: Capture durable behavioural rules when the user corrects you, so the same mistake doesn't repeat. Use this whenever the user says "don't do X", "stop doing Y", "no — do it this way", "from now on always/never", or explicitly asks to "capture that", "remember this", "add a rule", "turn this into a rule", or expresses frustration that you keep doing something wrong. Also use when the user confirms an approach that worked well and is worth codifying. Writes the rule to the appropriate CLAUDE.md (global, project, or a subagent's agents.md) with a diff-and-approve flow.
---

# Rule Capture

Turns corrections and preferences into durable rules so you don't drift or repeat mistakes across sessions.

## Why this matters

LLMs don't learn inside a conversation the way humans do — useful corrections fade once the session ends unless they're written down. CLAUDE.md and agents.md files are the one place the harness reliably reads on every new session, so that's where durable rules belong.

Without this skill, the loop looks like: user corrects → agent fixes → next session, same mistake. With this skill: user corrects → rule captured in CLAUDE.md → next session, correct by default.

## When to trigger

Trigger yourself when you observe any of these patterns:

| Pattern | Example |
|---------|---------|
| Explicit capture request | "Remember this", "capture that as a rule", "turn this into a rule", "add a rule for this" |
| Correction phrasing | "Don't do X", "stop doing Y", "no — this way", "instead, you should..." |
| Repeated correction | The user just corrected the same thing they've corrected before |
| Positive confirmation of non-obvious approach | "Yes, exactly — keep doing that", "that's the right call" on a judgment the agent made |
| Expressed frustration | "You keep doing X", "you always forget Y" |

When any of these apply, propose a rule proactively: *"Want me to capture this as a rule so it sticks next session?"*

Only skip if the correction is clearly one-off and situational — e.g. "don't add that comment **here**" refers to one specific spot, not a general preference.

## When NOT to trigger

- Casual observations that aren't prescriptive ("huh, interesting")
- One-off situational fixes that don't generalise
- Rules about specific files or values that'll become stale (e.g. "the API key is xyz")
- Ideas or brainstorming that haven't been acted on

## Workflow

### 1. Formulate the rule

Take what the user said and rewrite it as a concise, actionable rule in the imperative. Strip emotion and ambient context; keep the behaviour change.

**Example:**

- User: "No, don't just delete the file — I need to see what was in it first. You always do this."
- Rule: *"Before deleting any file, show its contents to the user first."*

If the user explicitly dictated the rule ("from now on, always X"), use their words verbatim.

### 2. Determine scope

Ask or infer which file to write to:

| Target file | When to use |
|-------------|-------------|
| `~/.claude/CLAUDE.md` | General habit across all projects (e.g., "always run `date` before stating the date") |
| `./CLAUDE.md` (project-local) | Project-specific behaviour (e.g., "in this Next.js project, always use Server Components by default") |
| `<agent-path>/agents.md` | Subagent-specific (e.g., the `explore` subagent shouldn't write files) |

Heuristics:
- If the correction mentions "this project", "this repo", "in <project-name>" → project-local
- If the correction is about a specific subagent → that agent's `agents.md`
- Otherwise → global

When uncertain, ask the user: *"Global (all projects) or just this project?"*

Always show the full target path before writing.

See `references/scope-selection.md` for edge cases (monorepos, worktrees, missing files).

### 3. Format the rule

Append to a `## Captured Rules` section at the end of the target file. Create the section if it doesn't exist. Use this format:

```markdown
## Captured Rules

- **<rule in imperative, one sentence>** (YYYY-MM-DD)
  - Why: <reason — either the user's explanation or inferred from context>
  - Context: "<short verbatim quote from the correction, ≤120 chars>"
```

The date lets you prune stale rules later. The "Why" makes the rule judgeable in edge cases. The "Context" makes the rule traceable back to what triggered it.

See `references/rule-format.md` for more examples and formatting edge cases.

### 4. Show diff and get approval

Show the user:

1. **Target file** (full path)
2. **Section** being appended to (existing `## Captured Rules` or new)
3. **Exact text** to be added

Ask: *"Add this rule? (y / n / edit)"*

- `y` → write atomically (temp + rename)
- `n` → abort, report nothing was changed
- `edit` → let the user rewrite, then re-confirm

Never write without explicit approval. Never batch multiple rules without showing each one.

### 5. Confirm

After writing, tell the user:

- Rule captured in: `<path>`
- Total rules now in that file: `<count>`
- If this is the first rule in a new file, remind them to commit if the file is version-controlled

## Avoiding bloat

CLAUDE.md is loaded on every session — every rule has a context cost. Keep the file lean:

- Before adding a rule, scan existing rules. If the new rule duplicates or conflicts with an existing one, propose merging or updating instead of appending.
- Periodically (user-initiated) review rules and prune stale ones. The skill doesn't auto-prune, but does flag rules older than 6 months during capture: *"There's an older rule about this from YYYY-MM-DD — replace or keep both?"*
- Prefer one general rule over many specific ones. If a user is hitting the same class of correction repeatedly, generalise.

## Edge cases

- **File doesn't exist**: create it with a basic frontmatter-free structure and add the `## Captured Rules` section
- **File is read-only** (permissions): report and ask the user to adjust
- **Rule contradicts an existing rule**: show both, let the user pick (replace / keep both / merge)
- **User asks to capture multiple rules at once**: process sequentially, diff-and-approve each one individually
- **Unclear scope**: default to global, note the assumption in the approval diff

## Gotchas

- Don't write a rule without showing the diff and getting explicit `y`. The whole point of the approval step is that the user owns their CLAUDE.md — never auto-append, even if you're "sure" they'd approve.
- Don't capture one-off situational fixes as rules. "Don't add that comment **here**" is about one line of code, not a general preference. Only capture if the correction generalizes.
- Don't let CLAUDE.md bloat with near-duplicate rules. Before appending, scan existing rules — if one already covers the same ground, propose updating it rather than adding another.
- Don't write vague rules. "Be careful with files" means nothing actionable. The rule must be specific enough that a future session can mechanically comply without judgment calls.
- Don't default to project-local when the rule is clearly global. If the user says "stop doing X" without mentioning "in this project", it's almost always a general preference — ask if genuinely ambiguous.
- Don't skip the "Why" field. Without it, the rule can't be judged in edge cases — future sessions will either over-apply or under-apply it.

## Reference files

- `references/scope-selection.md` — edge cases for picking the target file (monorepos, worktrees, subagents, missing files)
- `references/rule-format.md` — format spec, examples, and dos/don'ts for rule phrasing

Load these only when the relevant edge case comes up.
