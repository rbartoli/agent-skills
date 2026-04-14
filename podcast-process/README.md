# podcast-process

A Claude Code skill that turns a podcast URL into a structured extraction of **non-obvious insights, references, and action items**. Works across common podcast sources — Pocket Casts, Spotify, Apple, RSS, direct MP3, YouTube — via Whisper transcription and Claude-based extraction.

## The problem this solves

You hear something useful on a podcast, mean to come back to it, and the insight dies in "things I meant to follow up on". This skill captures it before that happens: paste the URL, get a structured extraction, review, keep what matters.

## What it does

When you share a podcast URL with a follow-up instruction ("extract info", "what did they say about X", "pull health tips from this"), the skill:

1. Resolves the URL to an audio file (pca.st / Spotify / Apple / RSS / YouTube all handled)
2. Transcribes via Whisper (caches transcripts — never re-pays for the same episode)
3. Extracts Insights / References / Action items / Counter-arguments / Quotes
4. Writes a structured markdown file to `~/.claude/podcast-process/extractions/`

You review the extraction, keep what matters, route it wherever you keep your notes. A separate personal command can automate the routing step.

## Installation

```bash
git clone https://github.com/rbartoli/podcast-process ~/.claude/skills/podcast-process
```

Update: `cd ~/.claude/skills/podcast-process && git pull`.

## Required keys

Create `~/.config/podcast-process/.env`:

```bash
OPENAI_API_KEY=sk-...        # Whisper transcription (~$0.006/min)
ANTHROPIC_API_KEY=sk-ant-... # Extraction (or use Claude Code's own credentials)
```

No OpenAI key? The skill falls back to local `whisper.cpp` — slower but free.

## Usage

Just paste a URL with a request:

- *"Extract useful info: https://pca.st/episode/abc123"*
- *"What did the guest say about gut health on https://podcasts.apple.com/us/podcast/..."*
- *"Pull insights from https://youtube.com/watch?v=xyz"*

## Output

Extractions land at `~/.claude/podcast-process/extractions/<date>-<slug>.md`:

```markdown
# Atomic Habits Revisited — The Tim Ferriss Show

**URL:** https://pca.st/episode/...
**Duration:** 1:43:22
**Host(s) / Guest(s):** Tim Ferriss, James Clear
**Processed:** 2026-04-14
**Extraction hint:** health and focus

## Summary
Two-sentence recap...

## Insights
- The 2-minute rule works because it bypasses...

## References
- *Atomic Habits* (Clear, 2018)
- Huberman Lab episode on dopamine scheduling

## Action items
- Stack a new habit onto an existing morning routine...

## Quotes
> "You do not rise to the level of your goals. You fall to the level of your systems."
```

## Structure

```
podcast-process/
├── README.md                       # this file
├── LICENSE
├── SKILL.md                        # workflow, triggers, safety rules
├── references/
│   ├── url-resolution.md           # pca.st / Spotify / Apple / YouTube resolution
│   ├── extraction-prompts.md       # tuned prompts, category definitions
│   └── transcription.md            # Whisper chunking, local fallback
└── evals/
    └── evals.json                  # trigger + output test cases
```

## Caching — the most important design choice

Every step caches:

```
~/.claude/podcast-process/
├── cache/         # audio files (can be cleared — will re-download if needed)
├── transcripts/   # transcribed text (KEEP — re-extraction is free once transcribed)
└── extractions/   # final markdown (the valuable output)
```

**Never re-pay for transcription.** The skill always checks `transcripts/<slug>.txt` before calling Whisper. You can re-extract with different prompts or different hints for free once a transcript exists.

## Cost estimates

| Episode length | Whisper API | Claude extraction | Total |
|----------------|:-----------:|:-----------------:|:-----:|
| 30 min | $0.18 | ~$0.03 | ~$0.21 |
| 60 min | $0.36 | ~$0.06 | ~$0.42 |
| 2 hrs | $0.72 | ~$0.10 | ~$0.82 |
| 3 hrs (MFM / Lex) | $1.08 | ~$0.15 | ~$1.23 |

Extraction is cheap; transcription dominates. Re-extraction (different hint on cached transcript) costs only the extraction line — basically free.

## Philosophy

- **Extraction > summarisation.** You can find summaries anywhere. What's valuable is the specific non-obvious thing.
- **Human review before routing.** Skill writes to staging; you decide what to keep.
- **Cache aggressively.** Never re-pay for work already done.
- **Fail gracefully.** Missing API keys → local fallback. Paywalled content → ask the user.

## Contributing

PRs welcome. Especially useful: adding new URL source patterns (overcast.fm, castro.fm, etc.) to `references/url-resolution.md`.

## License

MIT — see `LICENSE`.
