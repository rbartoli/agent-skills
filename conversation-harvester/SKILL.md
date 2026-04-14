---
name: conversation-harvester
description: Extract useful insights, decisions, questions, and references from past Claude Code sessions so knowledge doesn't get buried in chat history. Use this skill whenever the user asks to "harvest sessions", "review past conversations", "extract insights from my Claude history", "find useful stuff from prior chats", "surface what I worked on last week", "recover that thing I figured out in a previous session", "mine my session archive", or says they keep losing track of what they decided in past conversations — even if they don't explicitly say "sessions" or "history". Produces a staging file for human review before anything is committed downstream.
---

# Conversation Harvester

Extracts insights from your Claude Code session transcripts. Writes results to a **staging file** — nothing is committed to any other location until you review and approve.

## When NOT to use

- User is asking about the **current** session's content — use scrollback or conversation context, not the harvester (which only reads completed sessions on disk)
- User wants real-time tracking of what they work on — this is a batch tool that runs against history, not a live observer
- User wants to summarise a single, specific session by ID — open that JSONL directly; the harvester is for extracting across many sessions
- Sessions are stored somewhere other than `~/.claude/projects/` — this skill assumes the standard Claude Code layout

## Data sources

Claude Code stores every conversation as a JSONL file:

```
~/.claude/projects/<cwd-slug>/<session-id>.jsonl
```

Each line is a message event (user, assistant, tool call, tool result). This skill reads those files — no API calls, no cloud dependency.

Additional sources you can opt into:

- `~/.claude/tasks/<task-id>.output` — subagent task transcripts (same JSONL format)
- `~/.claude/history.jsonl` — flat log of prompts you've typed (no responses)

## State files

The skill maintains two files under `~/.claude/conversation-harvester/`:

- `manifest.json` — tracks which session IDs have been harvested, with timestamps and status (`harvested` | `committed`)
- `pending.md` — staging file where extracted items accumulate

Create these on first run if missing.

## Workflow

### 1. Scope the run

Before harvesting, ask the user which sessions to include if not obvious:

- *"Last 7 days / 30 days / everything?"* (first run should default to last 7 days to avoid dumping hundreds of items)
- *"Include subagent tasks?"* (default: yes)
- *"Include already-harvested sessions?"* (default: no, skip via manifest)

### 2. List candidate sessions

```bash
find ~/.claude/projects -name '*.jsonl' -newermt '<cutoff-date>' | sort
```

Cross-reference with `manifest.json` to filter out already-harvested session IDs.

### 3. Filter out noise

Skip a session if any of these apply:

- Fewer than 5 message events (likely cancelled or trivial)
- No assistant messages (user typed and quit)
- Only contains a single system-level command (e.g. `/clear`, `/help`)
- User explicitly blocked it (see `manifest.json` → `blocklist`)

### 4. Extract per session

For each remaining session:

1. Read the JSONL file
2. Build a compact representation (user prompts + key assistant responses + decisions made)
3. Pass to the extraction prompt in `references/extraction-prompt.md`
4. Parse the response into structured items

Extracted item categories:

- **Task** — concrete action the user said they'd do but might've forgotten
- **Idea** — exploratory thought worth keeping
- **Reference** — link, quote, tool, library, pattern worth remembering
- **Decision** — architectural or strategic choice made during the session, with reasoning
- **Reflection** — personal observation about habits, preferences, working style
- **Question** — open question the user raised that didn't get a satisfying answer

### 5. Append to staging

Write extracted items to `~/.claude/conversation-harvester/pending.md`, grouped by session date:

```markdown
## Session 2026-04-11 · project: rbartoli/myproject
_Session ID: `abc-123-def` · Started: 2026-04-11 14:23_

- **Decision:** Chose PostgreSQL over SQLite because... (exact reasoning from conversation)
- **Reference:** <url> — <what it is>
- **Task:** Finish the deploy script with the retry logic discussed
- **Idea:** What if we ran the cron in reverse (process oldest first)?
```

Each item carries a source tag back to its session so you can trace it.

### 6. Update manifest

Mark harvested sessions:

```json
{
  "sessions": {
    "abc-123-def": {
      "harvested_at": "2026-04-14T21:00:00Z",
      "status": "harvested",
      "item_count": 4,
      "path": "~/.claude/projects/slug/abc-123-def.jsonl"
    }
  }
}
```

### 7. Report

Tell the user:
- How many sessions were scanned
- How many were harvested (new extraction) vs skipped (already done, too short, blocked)
- How many items were added to `pending.md`
- Path to the staging file for review

## Reviewing the staging file

The user reviews `pending.md` as plain markdown:

- **Delete** lines they don't want kept
- **Edit** lines to clarify or reword
- **Leave** lines that should be kept

After review, the user runs their own "commit" command (the skill doesn't do this itself — it only produces the staging file). This keeps the skill generic and reusable: different users route committed items to different places (Obsidian, a journal file, a project notes folder, etc.).

## Options / invocations

- *"Harvest the last 7 days"* → filter by mtime
- *"Harvest everything"* → no date filter, warn if > 50 sessions
- *"Dry run"* → report what would be harvested, don't write `pending.md` or update manifest
- *"Re-harvest session abc-123"* → bypass manifest check for specific ID
- *"Block session abc-123"* → add to blocklist in manifest, never harvest again

## Safety

- **Never delete JSONL files.** Read-only.
- **Never skip the manifest update.** A crashed run that extracted items without updating the manifest would cause duplicate extraction next run.
- **Write `pending.md` atomically** (write to temp, rename) so a partial failure doesn't corrupt the staging file.
- **Respect privacy** — some sessions may contain API keys, passwords, or personal data the user pasted during debugging. The extraction prompt in `references/extraction-prompt.md` includes rules to redact these; honour them.

## Reference files

- `references/extraction-prompt.md` — the full prompt used to extract items from a session
- `references/manifest-schema.md` — JSON schema for the manifest file
- `references/staging-format.md` — format spec for `pending.md`

Load these on demand, not eagerly.
