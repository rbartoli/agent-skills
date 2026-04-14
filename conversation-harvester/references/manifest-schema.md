# Manifest Schema

Location: `~/.claude/conversation-harvester/manifest.json`

## Structure

```json
{
  "version": 1,
  "created_at": "2026-04-14T21:00:00Z",
  "sessions": {
    "<session-id>": {
      "harvested_at": "2026-04-14T21:00:00Z",
      "status": "harvested",
      "item_count": 4,
      "path": "~/.claude/projects/<slug>/<session-id>.jsonl",
      "mtime": "2026-04-11T14:23:00Z"
    }
  },
  "blocklist": [
    "<session-id-to-never-harvest>"
  ],
  "committed_sessions": [
    "<session-id-moved-to-downstream-note>"
  ]
}
```

## Fields

| Field | Type | Description |
|-------|------|-------------|
| `version` | int | Schema version. Bump on breaking changes. |
| `created_at` | ISO datetime | When the manifest was first created |
| `sessions` | object | Map of session ID → harvest metadata |
| `sessions.<id>.harvested_at` | ISO datetime | When this session was harvested |
| `sessions.<id>.status` | enum | `harvested` (in pending.md), `committed` (moved downstream), `skipped` (filtered out by noise rules) |
| `sessions.<id>.item_count` | int | Number of items extracted |
| `sessions.<id>.path` | string | Original JSONL path (for tracing) |
| `sessions.<id>.mtime` | ISO datetime | File modification time at harvest |
| `blocklist` | array of strings | Session IDs to never harvest (user explicitly blocked) |
| `committed_sessions` | array of strings | Session IDs whose items have been committed downstream |

## Operations

- **Harvest session X** → add to `sessions` with status `harvested`
- **Commit session X downstream** → change status to `committed` and move ID to `committed_sessions` array (a downstream command does this, not this skill)
- **Block session X** → add to `blocklist`
- **Unblock session X** → remove from `blocklist`

## Integrity

- Always write atomically: temp file + rename
- Keep a backup: on every write, copy the previous manifest to `manifest.json.bak` first
- On read failure, fall back to `manifest.json.bak` and report the issue
