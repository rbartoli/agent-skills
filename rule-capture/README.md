# rule-capture

A Claude Code skill that turns corrections into **durable rules** written to your `CLAUDE.md` or a subagent's `agents.md` — so the same mistake doesn't repeat across sessions.

## The problem this solves

LLMs don't learn inside a conversation the way humans do. If you tell Claude "don't do X" in session A, and start a fresh session B tomorrow, X is back. The fix is well-known: add a rule to `CLAUDE.md`, which the harness loads on every session.

Doing that manually is friction. `rule-capture` makes it one sentence.

## What it does

When you correct Claude ("don't do X", "stop Y", "no — this way", "from now on always Z") or explicitly ask to capture a rule ("remember this", "turn it into a rule"), the skill:

1. Formulates the rule in concise imperative voice
2. Picks the right file (global vs. project-local vs. subagent `agents.md`)
3. Shows you the exact diff
4. Waits for `y` / `n` / `edit`
5. Writes atomically to a `## Captured Rules` section

Each rule includes a date and a short quote from what triggered it, so you can audit and prune later.

## Installation

```bash
git clone https://github.com/rbartoli/rule-capture ~/.claude/skills/rule-capture
```

To update: `cd ~/.claude/skills/rule-capture && git pull`.

## Usage

Just talk naturally. The skill triggers when you correct or give preference:

- *"No please stop — we use bun in this project, not npm"* → proposes a project-local rule
- *"Remember, always run date before stating today's date"* → proposes a global rule
- *"From now on the research subagent shouldn't write files"* → proposes an agents.md rule

You can also invoke it explicitly:

- *"Capture that as a rule"*
- *"Turn this preference into a CLAUDE.md rule"*
- *"Add a rule so you don't forget"*

## Structure

```
rule-capture/                 ← clone target
├── README.md                 # this file
├── LICENSE
├── SKILL.md                  # workflow, triggers, format
├── references/
│   ├── scope-selection.md    # picking the target file (monorepo/worktree/subagent/missing)
│   └── rule-format.md        # format spec, dos and don'ts, merging
└── evals/
    └── evals.json            # trigger + output test cases
```

## Rule format

Rules are appended to a `## Captured Rules` section:

```markdown
## Captured Rules

- **Before deleting any file, show its contents first.** (2026-04-14)
  - Why: lost work last quarter from a reflex `rm`
  - Context: "no, don't just nuke it — show me what was there"
```

See `references/rule-format.md` for the full spec.

## Philosophy

- **Every rule must have a "why"** — makes it judgeable in edge cases
- **Imperative voice, one sentence** — unambiguous
- **Traceable** — the quote links back to what triggered the rule
- **Lean CLAUDE.md** — merge with existing rules when possible; old rules (>6 months) get flagged at next capture

## Compared to manual CLAUDE.md editing

You *can* just edit `CLAUDE.md` yourself. The skill adds value when:

- You're mid-task and don't want to break flow
- You're not sure which file to target (global / project / agents.md)
- You want consistent formatting across captured rules
- You want the traceability back to what triggered the rule

## Why open source

Forking a personal `CLAUDE.md` workflow into a skill makes it reusable. Anyone running Claude Code can install it, and everyone's rules stay in their own files.

## Contributing

PRs welcome. If you change extraction logic or rule format, update:

1. `SKILL.md` workflow section
2. `references/rule-format.md` if format changes
3. `evals/evals.json` if new behaviour needs tests

## License

MIT — see `LICENSE`.
