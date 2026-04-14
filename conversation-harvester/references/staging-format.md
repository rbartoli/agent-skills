# Staging File Format

Location: `~/.claude/conversation-harvester/pending.md`

## Structure

```markdown
# Conversation Harvester — Pending Items

> Items extracted from Claude Code sessions. Review, delete what's noise, edit what needs refining. A downstream command (e.g. `/obsidian:commit-sessions`) moves surviving items to your target note.

---

## Session 2026-04-11 · project: <project-slug>

_Session ID: `abc-123-def` · Started: 2026-04-11 14:23_

- **Decision:** ...
- **Reference:** ...
- **Task:** ...
- **Idea:** ...

---

## Session 2026-04-12 · project: <another-slug>

_Session ID: `xyz-456-ghi` · Started: 2026-04-12 09:00_

- **Reflection:** ...
- **Question:** ...
```

## Rules

1. **One section per session**, headered with date and project slug
2. **Session ID and timestamp in italics** right under the header — for traceability
3. **Items are markdown list entries** with the category as the first bold word
4. **No duplicate items within a session** — deduplicate before appending
5. **Items remain in the file** until a downstream command moves or deletes them

## Review behaviour

The user reviews this file as plain markdown. They can:

- **Delete** a line → that item is dropped
- **Delete** an entire session block → no items from that session get committed
- **Edit** a line → the edited version is what gets committed
- **Leave as-is** → the item moves to the downstream note unchanged

## Atomic writes

- Harvest phase writes to `pending.md.tmp`, renames to `pending.md`
- Never truncate `pending.md` mid-write — appends only
- A downstream commit command should similarly write the output file atomically before clearing or archiving `pending.md`
