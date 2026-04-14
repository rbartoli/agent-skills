# conversation-harvester

A Claude Code skill that extracts useful insights from your past sessions. Surfaces decisions, references, ideas, and tasks that got buried in conversation logs — before they're lost.

Produces a **staging file** for human review. Nothing is committed anywhere else until you explicitly move approved items downstream.

## What it does

Claude Code stores every session as a JSONL transcript at `~/.claude/projects/*.jsonl`. This skill reads those transcripts and extracts, per session:

- **Decisions** made (with reasoning)
- **References** worth keeping (links, libraries, patterns)
- **Tasks** the user said they'd do
- **Ideas** raised during the conversation
- **Reflections** about habits and preferences
- **Questions** that didn't get a satisfying answer

Items are written to a staging file. You review, delete noise, and then a separate (personal) command moves the survivors wherever you keep your notes.

## Installation

One command:

```bash
git clone https://github.com/rbartoli/conversation-harvester ~/.claude/skills/conversation-harvester
```

To update: `cd ~/.claude/skills/conversation-harvester && git pull`.

## Usage

Just ask naturally:

- *"Harvest my last 7 days of sessions"*
- *"Review past conversations for anything useful"*
- *"Extract insights from my Claude history"*
- *"Dry run — show me what you'd harvest"*

The skill will:

1. List candidate sessions (filtered by date, deduplicated against its manifest)
2. Extract items from each using a tuned prompt
3. Write results to `~/.claude/conversation-harvester/pending.md`
4. Update the manifest to mark sessions as harvested

You then open `pending.md`, delete what's noise, and commit the survivors downstream however you like.

## Downstream commit (personal, not part of this skill)

This skill deliberately does NOT know about your note system. To commit approved items to your notes, write your own small command. Example for an Obsidian user:

```markdown
---
description: Move approved items from conversation-harvester staging into my Inbox.md
---

Read `~/.claude/conversation-harvester/pending.md`, append everything left in it to my Obsidian Inbox under `## Extracted from Claude Sessions`, then clear pending.md and mark all those sessions as `committed` in the manifest.
```

Save as `~/.claude/commands/obsidian/commit-sessions.md`, invoke with `/obsidian:commit-sessions`.

Users with other note systems write their own equivalent — the skill stays generic.

## Structure

```
conversation-harvester/           ← clone target
├── README.md                     # this file
├── LICENSE
├── SKILL.md                      # workflow, state tracking, safety rules
└── references/
    ├── extraction-prompt.md      # the prompt used to extract items
    ├── manifest-schema.md        # JSON schema for the manifest file
    └── staging-format.md         # format spec for pending.md
```

## State files

Created on first run:

```
~/.claude/conversation-harvester/
├── manifest.json        # tracks harvested session IDs
├── manifest.json.bak    # rolling backup
└── pending.md           # staging file you review
```

## Filters

Sessions are skipped if:

- Fewer than 5 message events (likely cancelled)
- No assistant messages (user typed and quit)
- Only a single system command (`/clear`, `/help`, etc.)
- Session ID is on the user's blocklist

You can tune these filters by editing SKILL.md in your local clone.

## Privacy

- Everything runs locally — no data leaves your machine except via Claude Code's normal LLM calls
- The extraction prompt includes rules to avoid extracting credentials, API keys, or third-party PII
- You review every item before it moves anywhere

## Contributing

PRs welcome. If you add a new extraction category or a new filter, update:

1. `SKILL.md` workflow section
2. `references/extraction-prompt.md` if extraction logic changes
3. `README.md` if the public surface changes

## License

MIT — see `LICENSE`.
