---
name: process-raindrop
description: Interactive triage of Raindrop.io bookmarks — summarize, propose actions (keep/delete/save to Obsidian), and execute them in batches. Use this skill whenever the user mentions raindrop, bookmarks, saved links, link hoarding, "process my links", "triage bookmarks", "clean up my saves", or wants to go through a backlog of saved URLs. Also use when the user says things like "I have too many bookmarks" or "help me go through my saved stuff". This skill works with zero scraping by default, making it very cheap to run.
---

# Process Raindrop Links

You're helping someone who saves too many links and never goes back to read them. The goal is to make triage fast, cheap, and satisfying — process 10 bookmarks at a time, propose what to do with each, and execute immediately on approval. The user should feel like they're making real progress through their backlog.

## Why This Exists

Bookmark hoarding is a common trap: you save 20 links a day, read 2, and the backlog grows forever. This skill turns that into an interactive sweep where the user spends ~5 seconds per link instead of opening each one. The trick is that Raindrop already stores enough metadata (title, excerpt, tags) to make a good triage decision without visiting the page.

## Requirements

- `raindrop` MCP server with `RAINDROP_ACCESS_TOKEN`
- `obsidian` MCP server (for saving to vault)
- Optional: `firecrawl-mcp` MCP server (only if user asks to "deep read" a specific item)

## Workflow

### 1. Ask Scope

Ask which collection to process and sort order. Keep it brief:

```
Which collection? (all / unsorted / collection name)
Sort? (newest first / oldest first)
```

- `collectionId: 0` = All, `collectionId: -1` = Unsorted
- Default to all + newest first if user just says "go"

### 2. Fetch and Clean

Fetch 10 bookmarks via `list_raindrops`. Before presenting, clean up click-tracker URLs — some bookmarks pass through redirect services (e.g. `feedback-production-*.railway.app/click?url=...`). Extract the real URL from the `url` query parameter so the user sees the actual destination.

### 3. Present Triage Table

Show each item as a compact two-line block with a proposed action:

```
#1  [obsidian]  Claude Code steering tips (reddit.com) — 2d ago
    "Spent way too long manually steering Claude Code..."
#2  [delete]    cPanel vulnerability exploit (slashdot.org) — 3d ago
    "Hackers exploiting critical cPanel CVE-2026-41940..."
#3  [keep]      Loopsy cross-machine agent comms (github.com) — 3d ago
    "Cross-machine AI agent communication + mobile app..."
```

The proposed action is your best guess based on the metadata — the user will confirm or override. When the excerpt is empty or useless (e.g. "Explore this post and more from..."), show `[no excerpt — deep read?]` to signal the metadata isn't enough.

### 4. Propose Actions

The point of proposing actions is to save the user cognitive effort — they should be able to say "next" and trust that your proposals are reasonable most of the time.

Heuristics for proposals:

| Signal | Action | Why |
|--------|--------|-----|
| Matches user's interests (AI, coding, tools, productivity) | **obsidian** | Worth extracting to their knowledge base |
| Time-sensitive content that's gone stale (news > 7d old) | **delete** | No longer actionable |
| Shopping, deals, product pages | **delete** | Ephemeral by nature |
| Reference repos, docs, tools | **keep** | Good as a bookmark, doesn't need Obsidian |
| Useless or empty excerpt | **deep read?** | Can't decide without seeing the content |

### 5. Wait for User Input

After the table, prompt:

```
Accept all? Or override by number (e.g. "3 keep", "1 delete", "5 deep read")
Commands: next (accept all + continue) | skip (skip batch) | stop (end session)
```

- **next** — accept proposed actions, execute, fetch next batch
- **3 keep** / **1 obsidian** — override specific items
- **deep read 5** — scrape item #5 with Firecrawl, show full summary, re-propose
- **skip** — move to next batch without acting
- **stop** — end session, show summary

### 6. Execute Actions

- **keep**: Leave in Raindrop, add tag `triaged` so future runs can skip already-processed items
- **delete**: `bookmark_manage` with `operation: "delete"`
- **obsidian**: Append to Obsidian `Inbox.md` under `## Extracted from Raindrop` using `obsidian_append_content`. Format: `- [Title](clean_url) — one-line summary from excerpt`. Create the section if it doesn't exist.

### 7. Progress Tracking

After each batch:
```
Batch 1: 3 kept, 4 deleted, 3 → Obsidian | Progress: 10/321
```

On stop or completion:
```
Session done: 40 processed (12 kept, 18 deleted, 10 → Obsidian) | 281 remaining
```

## Cost Design

This skill is intentionally cheap. The only "AI cost" is the conversation itself — there are no extra LLM calls, no scraping, no embeddings. You read Raindrop metadata and reason about it directly. The one exception is "deep read" which uses Firecrawl to fetch page content, but only when explicitly requested by the user.

## Edge Cases

- **Click-tracker URLs**: Always unwrap before presenting or saving to Obsidian
- **Duplicates**: If two bookmarks share the same domain+path, flag the second as `[duplicate of #N]`
- **Empty excerpts**: Show `[no excerpt — deep read?]` and let the user decide
- **Raindrop rate limits**: The 10-item batch size with human interaction between batches naturally stays well under API limits
