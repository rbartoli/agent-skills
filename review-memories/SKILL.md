---
name: review-memories
description: Review and triage auto-memory files for the current project — verify staleness, check referenced paths, prune what's outdated. Use this skill whenever the user mentions reviewing memories, cleaning up memory, auditing memory, pruning stale memories, or asks "what do we remember?" or "are our memories current?". Also trigger when the user types `/review-memories`. This is a maintenance skill that keeps the memory system healthy — without it, stale memories accumulate and mislead future sessions.
---

Interactively review every memory file in the current project's memory directory. The goal: keep the memory system high-signal by removing stale, wrong, or redundant entries.

## Scope

**Default:** Current project's memory directory only.
- Path pattern: `~/.claude/projects/<project-slug>/memory/`
- The project slug matches the current working directory (slashes replaced with hyphens, leading hyphen).

**With `--all` argument:** Walk every project directory under `~/.claude/projects/` that has a `memory/` subdirectory. Process each project in sequence, announcing the project name before its memories.

## Setup

1. Determine the memory directory path from the current working directory.
2. Read `MEMORY.md` (the index file) in full.
3. Read every `.md` file in the directory (excluding `MEMORY.md`).
4. For each memory file, parse the YAML frontmatter (`name`, `description`, `metadata.type`) and the body.
5. Count memories and announce: `N memories to review. Starting.`

If the memory directory doesn't exist or is empty, say so and exit.

## Presenting each memory

For each memory file, run these checks silently before presenting:

### Staleness checks

1. **File age:** Run `stat -c %Y <file>` to get last-modified time. Flag if older than 30 days.
2. **Referenced paths:** Scan the body for file paths (anything matching `src/`, `~/`, `/home/`, or common code path patterns). For each, check if the file still exists. Flag missing files.
3. **Referenced names:** Scan for `[[wikilinks]]` (Obsidian-style). Check if a matching `.md` file exists in the project or vault. Flag broken links.
4. **Frontmatter completeness:** Flag if `name`, `description`, or `type` is missing.
5. **MEMORY.md consistency:** Check that the file has a corresponding entry in `MEMORY.md`. Flag orphans (file exists but no index entry) and ghosts (index entry but no file).

### Display format

```
[N left] TYPE — name
  description line from frontmatter
  Age: X days | Paths: OK / 2 missing | Links: OK / 1 broken
  ──────────────────────────────
  <body text, first 300 chars — if longer, append "…">
  ──────────────────────────────
  a:keep  b:update  c:delete  d:expand  j:skip
```

**For memories with issues** (stale, broken paths, missing frontmatter), add a line:

```
  ⚠ 45 days old, references deleted file src/old-handler.ts
```

### Keys

- **a** = keep as-is (no changes, advance to next)
- **b** = update (ask what to change, rewrite the file, advance)
- **c** = delete (remove file + remove MEMORY.md entry)
- **d** = expand (show full body text, then re-show options)
- **j** = skip (keep in memory, advance — same as "a" but signals "I'll deal with this later")
- **q** = quit early

## Executing actions

### Keep / Skip
No changes. Advance to next memory.

### Update
Ask: `What should change?`
Wait for the user's response. Then:
1. Rewrite the memory file with the updated content, preserving frontmatter structure.
2. If the `description` changed, update the corresponding line in `MEMORY.md`.
3. Write both files immediately.
4. Print: `→ updated <name>`

### Delete
1. Remove the memory file.
2. Remove the corresponding line from `MEMORY.md`.
3. Write `MEMORY.md` immediately.
4. Print: `→ deleted <name>`

If the deletion leaves a section header in `MEMORY.md` with no entries below it (next line is another `##` or EOF), remove that empty section header too.

### Expand
Show the full body text of the memory (not truncated). Then re-show the same options (a/b/c/d/j). This doesn't consume a "turn" — the user still needs to choose an action.

## After each action

Print the one-line result, then immediately present the next memory. No pause, no extra commentary.

## Ordering

Present memories in this order (most likely to be stale first):
1. Memories with broken paths or missing references (immediate attention)
2. Memories older than 30 days (staleness candidates)
3. Project memories (context changes fastest)
4. Reference memories (external pointers may go stale)
5. User memories (rarely change)
6. Feedback memories (most durable — corrections tend to stay valid)

Within each group, oldest first.

## Orphan and ghost handling

Before the main review loop, check for consistency issues:

**Orphan files** (`.md` files in memory dir with no MEMORY.md entry):
```
⚠ Found 2 orphan memory files (no MEMORY.md entry):
  - old_note.md
  - unnamed_feedback.md
  a:add to index  b:delete  j:skip each
```

Present each orphan for the user to add to the index or delete.

**Ghost entries** (MEMORY.md lines pointing to non-existent files):
```
⚠ Found 1 ghost entry in MEMORY.md (file missing):
  - [old_reference.md](old_reference.md) — pointer to deleted resource
  a:remove from index  j:keep
```

Handle these before the main loop so the index is clean.

## Final summary

After all memories are processed (or user quits):

```
Review complete: N memories processed
  Kept:      X
  Updated:   Y
  Deleted:   Z
  Skipped:   S
  Remaining: R
```

## Rules

- Write files after every single action — crash-safe by design.
- Never create new memory files during review. This skill is for maintenance, not creation.
- Preserve exact frontmatter structure when updating (don't reformat YAML style).
- When deleting, also check for `[[name]]` references in other memory files. If found, warn: `⚠ Referenced by <other-file> — delete anyway? (y/n)`
- Don't re-read files you've already read unless the user chose "update" and you need to verify the write.
- The memory directory path uses the project slug convention: working directory with `/` replaced by `-` and a leading `-`. Example: `/home/riccardo/Sync/Projects/my-app` → `-home-riccardo-Sync-Projects-my-app`.
