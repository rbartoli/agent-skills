# podcast-process

A Claude Code skill that **finds a podcast's public transcript** and extracts structured insights from it. Paste a podcast URL, get back a markdown file of non-obvious insights, references, action items, and quotes.

## What it does

When you share a podcast URL with an extraction instruction ("what did they say about X", "pull insights", "extract useful info"), the skill:

1. Identifies the podcast and episode from the URL
2. Finds the public transcript (show website, YouTube auto-captions, Podscribe, web search)
3. Extracts structured items from the transcript using Gemini (or Claude)
4. Writes a markdown file to `~/.claude/podcast-process/extractions/`

## What it deliberately does NOT do

- **No Whisper transcription.** This skill relies on public transcripts that already exist. Popular AI/tech/health podcasts publish them; long-tail indie podcasts often don't. For the rare podcast without a transcript, the skill stops and says so — it doesn't download audio and pay for Whisper.
- **No audio downloads.** Transcripts are text.
- **No API keys required** for the common case (Gemini CLI is free, transcripts are public). Claude Code's own credentials handle extraction if Gemini isn't installed.

This keeps the skill simple, fast, and free. For the occasional podcast without a transcript, use a dedicated transcription tool.

## Installation

Part of [rbartoli/agent-skills](https://github.com/rbartoli/agent-skills) — see the root README for marketplace or manual install options.

## Required tools

- **`defuddle`** — fetches and cleans web pages (used to pull transcripts from show websites)
- **`gemini` CLI** (recommended) — used for the extraction step (free, 1M+ context fits any transcript)
- **`yt-dlp`** (optional) — only needed for falling back to YouTube auto-captions

All three are easy to install and don't require API keys for basic use.

## Usage

Just share a URL with an instruction:

- *"Extract AI engineering insights from https://pca.st/episode/..."*
- *"What did <guest> say about <topic> on this: <URL>"*
- *"Pull insights from https://latent.space/p/..."*

The skill will find the transcript and extract. If no transcript exists, it'll tell you and stop.

## Output

`~/.claude/podcast-process/extractions/<date>-<slug>.md`:

```markdown
# Andrej Karpathy on Code Agents — No Priors

**URL:** https://pca.st/episode/...
**Transcript URL:** https://no-priors.com/...
**Host(s) / Guest(s):** Sarah Guo, Andrej Karpathy
**Processed:** 2026-04-15

## Summary
...

## Insights
- ...

## References
- ...

## Action items
- ...

## Counter-arguments
- ...

## Quotes
> "..."
```

## Structure

```
podcast-process/
├── README.md                      # this file
├── SKILL.md                       # workflow, triggers, safety rules
├── references/
│   ├── transcript-sources.md      # per-podcast index of where transcripts live
│   └── extraction-prompts.md      # the prompts + tuning guide
└── evals/
    └── evals.json                 # trigger + output test cases
```

## Known transcript sources

The skill has an index of popular podcasts and where their transcripts live (Latent Space, No Priors, Lex Fridman, Tim Ferriss, Huberman, Acquired, The Changelog, etc.) — see `references/transcript-sources.md`.

For unknown podcasts, it falls back to YouTube auto-captions (via `yt-dlp`), Podscribe search, or a general web search.

## Philosophy

- **Transcript-first, not audio-first.** 90% of the podcasts you'd want to extract from already have public transcripts. Don't reinvent transcription.
- **Honest "no" over fabricated "yes".** If there's no transcript, tell the user. Never synthesise from episode titles or general knowledge.
- **Cache aggressively.** Once you have the transcript text, re-extraction with a different hint is free.
- **Human verification welcome.** Every extraction includes the transcript URL so the user can spot-check claims.

## Contributing

PRs welcome. The most useful contributions are adding podcasts to `references/transcript-sources.md` as you discover their transcript locations.

## License

MIT — see the [root LICENSE](../LICENSE).
