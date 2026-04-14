---
name: podcast-process
description: Transcribe podcast episodes and extract useful insights. Use this skill whenever the user shares a podcast URL (Pocket Casts / pca.st, Spotify, Apple Podcasts, an RSS / MP3 link, or a YouTube podcast upload) and asks to "extract info", "pull insights", "summarise", "transcribe", "what's in this", "what did X say about Y", or gives any podcast URL with a follow-up instruction about what to learn from it — even if they don't explicitly say "podcast" or "transcribe". Also trigger if the user is processing daily notes or an inbox and encounters pending podcast URLs. Produces a structured extraction written to a staging file for human review and downstream routing.
---

# Podcast Process

Turns a podcast URL into a structured extraction of non-obvious insights, references, and action items. Works across common podcast sources — Pocket Casts, Spotify, Apple, RSS, direct MP3, YouTube — via Whisper transcription and Claude-based extraction.

## When NOT to use

- The user shares a blog post or article URL (use `defuddle` instead — it's cheaper and faster than transcription)
- The user wants to transcribe their own voice note or meeting recording — this skill is tuned for third-party podcast content with extraction; a generic transcription tool is a better fit
- The user shares a podcast URL but explicitly wants the full transcript, not extraction — this skill emphasises insight-extraction over verbatim transcription
- The episode is behind a paywall the user can't bypass (e.g. private Patreon feed they don't subscribe to)

## Workflow

### 1. Resolve the URL to an audio file

Different sources need different resolution paths. See `references/url-resolution.md` for specifics.

Quick reference:

| Source pattern | Resolution |
|----------------|-----------|
| `pca.st/episode/...` | Follow redirects → show page has RSS / direct audio link in metadata |
| `open.spotify.com/episode/...` | Spotify doesn't expose audio directly; use `spotdl` or resolve show to its RSS via Listen Notes / public RSS registry |
| `podcasts.apple.com/.../id...` | Apple exposes the show's RSS in the page; find the episode in the RSS by title/date |
| Direct `.mp3` / `.m4a` / RSS feed | Use directly |
| `youtube.com/watch` | Use `yt-dlp -x --audio-format mp3 <url>` to extract audio |

Download the audio to `~/.claude/podcast-process/cache/<episode-slug>.mp3`. Skip download if cached.

### 2. Transcribe

Default: OpenAI Whisper API (`whisper-1` model).

```bash
curl https://api.openai.com/v1/audio/transcriptions \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -H "Content-Type: multipart/form-data" \
  -F file=@<audio-file> \
  -F model=whisper-1 \
  -F response_format=text > <cache>/transcript.txt
```

Cost: ~$0.006/minute. A 60-min episode = ~$0.36. Cache transcripts — never re-transcribe the same episode.

See `references/transcription.md` for longer files (>25MB — need chunking), language options, and a local `whisper.cpp` fallback if the user prefers offline.

### 3. Extract insights

Read the extraction hint from the user's request. If they said:

- "Extract useful info" → use the generic prompt
- "Extract X info" (e.g. "health and focus info") → use the focused prompt tuned to X
- No hint, bare URL → default to generic + ask once what they're specifically after

The extraction prompt is in `references/extraction-prompts.md`. It produces items in these categories:

- **Insight** — non-obvious argument, counter-intuitive claim, or framework
- **Reference** — book, person, paper, tool, quote worth remembering
- **Action item** — a concrete thing the listener could do
- **Counter-argument** — where the host / guest is likely wrong
- **Quote** — verbatim line worth keeping

Only include items the user's extraction hint actually cares about. Don't force all categories into every output.

### 4. Write to staging

Write the extraction to `~/.claude/podcast-process/extractions/<YYYY-MM-DD>-<episode-slug>.md` with this structure:

```markdown
# <Episode Title> — <Podcast Name>

**URL:** <original URL>
**Duration:** <mm:ss>
**Host(s) / Guest(s):** <names if identifiable from transcript>
**Processed:** <YYYY-MM-DD>
**Extraction hint:** <what the user asked for, or "generic">

## Summary (2-3 sentences)

<plain-language summary of what the episode was about>

## Insights

- ...

## References

- ...

## Action items

- ...

## Quotes

> "..." — <speaker>

## Raw transcript

<link or path to cached transcript.txt>
```

### 5. Report back

Tell the user:
- Path to the extraction markdown
- Count of items per category
- Cost of the Whisper call (approximate)
- Link to raw transcript if they want to read it

If a downstream routing command exists (e.g. a personal `/obsidian:commit-podcast`), suggest running it. Otherwise the user can copy-paste the extraction to their note system manually.

## State & caching

```
~/.claude/podcast-process/
├── cache/                    # downloaded audio (can be cleared, will re-download)
│   └── <episode-slug>.mp3
├── transcripts/              # transcribed text (keep — re-extraction is free once transcribed)
│   └── <episode-slug>.txt
└── extractions/              # final markdown outputs (the valuable ones)
    └── <YYYY-MM-DD>-<slug>.md
```

Never re-transcribe an episode you've already done. Check `transcripts/<slug>.txt` before calling Whisper.

## Required keys

- `OPENAI_API_KEY` — for Whisper. Load from `~/.config/podcast-process/.env` or the user's shell.
- `ANTHROPIC_API_KEY` — for the extraction step (or use Claude Code's own credentials).

If `OPENAI_API_KEY` is missing, offer the local `whisper.cpp` fallback (see `references/transcription.md`).

## Safety

- **Respect copyright** — transcription for personal analysis is fair use in most jurisdictions; don't republish transcripts
- **Paywalled content** — if the episode is behind a paywall, check that the user has access before downloading
- **API cost** — always show estimated cost before transcribing anything over 60 minutes

## Reference files

- `references/url-resolution.md` — detailed URL → audio resolution for each source
- `references/extraction-prompts.md` — the prompts used, tuning guide, category definitions
- `references/transcription.md` — Whisper chunking, local fallback, language options

Load these only when the relevant phase needs them.
