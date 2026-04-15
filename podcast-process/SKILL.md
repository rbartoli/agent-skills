---
name: podcast-process
description: Find the public transcript of a podcast episode and extract useful insights, references, and action items from it. Use this skill whenever the user shares a podcast URL (Pocket Casts / pca.st, Spotify, Apple Podcasts, an RSS / MP3 link, or a YouTube podcast upload) and asks to "extract info", "pull insights", "summarise", "what's in this", "what did X say about Y", or gives any podcast URL with a follow-up instruction about what to learn from it — even if they don't explicitly say "podcast" or "transcript". Also trigger if the user is processing daily notes or an inbox and encounters pending podcast URLs. This skill is transcript-based: if no public transcript exists, it stops and tells the user rather than attempting audio transcription.
---

# Podcast Process

Finds the public transcript of a podcast episode and runs a structured extraction against it. Produces a markdown file of non-obvious insights, references, action items, counter-arguments, and quotes — saved to a staging location for human review.

**This skill does not transcribe audio.** It relies on transcripts that already exist online (show websites, Podscribe, YouTube captions, etc.). If no transcript can be found, the skill reports that and stops. Paying for Whisper transcription is out of scope — it adds cost and complexity for the rare case where no transcript exists. For those, use a dedicated transcription tool.

## When NOT to use

- The user shares a blog post or article URL — use `defuddle` instead
- The user wants to transcribe their own voice note or meeting recording — use a dedicated transcription tool
- The episode is behind a paywall the user can't bypass (e.g. private Patreon feed they don't subscribe to)
- No public transcript exists and the user needs extraction urgently — tell them this skill can't help and suggest alternatives

## Workflow

### 1. Identify the podcast and episode

From the URL, extract what you can about the show and episode:

- **Podcast name** (e.g. "No Priors", "Latent Space", "My First Million")
- **Episode title** (from metadata, page title, or show notes)
- **Host and guest names** (often in the title or show description)
- **Publish date**

Use `defuddle` on the URL to pull the page's title/description metadata. This is your starting point for finding the transcript.

### 2. Find the transcript

See `references/transcript-sources.md` for a per-podcast index of known transcript locations. The common patterns:

| Source | Where to look |
|--------|--------------|
| Latent Space | `latent.space/p/<slug>` — swyx publishes full transcripts on every episode |
| No Priors | `no-priors.com` / Substack — transcripts in episode posts |
| My First Million | `shaanpuri.com` / Shaan's newsletter sometimes has edited highlights |
| Lex Fridman | `lexfridman.com/<guest-name>` — full transcripts with timestamps |
| Huberman Lab | `hubermanlab.com/podcast/<slug>` — full transcripts |
| Tim Ferriss | `tim.blog/<year>/<month>/<day>/<slug>` — full transcripts |
| YouTube podcast uploads | auto-captions via `yt-dlp --write-auto-subs --skip-download` (no API key needed) |
| Unknown / long tail | Podscribe (`podscribe.com/search`), a web search for `"<episode title>" transcript`, or the show's own site |

If `gemini` CLI is installed, ask it to find the transcript URL directly — it can web search and return a link. Verify the URL with `defuddle` before assuming it's correct.

If no transcript is findable, **stop** and report clearly: *"No public transcript found for this episode. This skill relies on existing transcripts. Try asking the show directly or use a dedicated transcription tool."*

### 3. Fetch the transcript

Use `defuddle` to pull the transcript text:

```bash
defuddle parse <transcript-url> --md -o /tmp/transcript.md
```

Sanity-check: the file should be thousands of words, not a short blurb. If it's short, you probably fetched show notes, not the transcript — go back to step 2 and look harder.

For YouTube auto-captions (no transcript on the show's site):

```bash
yt-dlp --write-auto-subs --skip-download --sub-format vtt --sub-lang en -o '/tmp/%(id)s' <youtube-url>
# then clean VTT timestamps into plain text
```

### 4. Extract

Read the extraction hint from the user's request. Apply the prompt from `references/extraction-prompts.md` — generic if no hint, focused if the user named a topic.

The preferred extraction path is `gemini` CLI with the transcript passed inline, because:
- Gemini's 1M+ context fits any full podcast transcript
- It's free (or covered by the user's subscription)
- One call produces structured output

```bash
TRANSCRIPT=$(cat /tmp/transcript.md)
EXTRACTION_PROMPT=$(cat ~/.claude/skills/podcast-process/references/extraction-prompts.md)
timeout 90s gemini -p "$EXTRACTION_PROMPT

---

Transcript below:

$TRANSCRIPT" > /tmp/extraction.md
```

**Always wrap gemini calls in `timeout 90s`.** Gemini occasionally hangs silently on queries that trigger its web-search tool (observed: "find the URL of X" queries, "what does Y say in Z episode" queries). Without a timeout, the call can sit indefinitely while producing zero bytes of output. 90 seconds is enough for any legitimate extraction.

### Fallback ladder when gemini fails or times out

If the first gemini call returns empty / times out:

1. **Retry once with a simpler prompt** — drop the extraction template, just ask: `timeout 60s gemini -p "Summarise this transcript and list 3 non-obvious insights: $TRANSCRIPT"`. Sometimes the full structured prompt triggers more tool-use than needed.
2. **Switch to Claude API** — if gemini is broken for the session, call Claude directly via Claude Code's credentials (no gemini dependency).
3. **Stop and report** — if both fail, tell the user clearly: *"Transcript fetched successfully (path: X) but extraction failed via gemini (timeout) and Claude (error). Here's the transcript URL — you can extract manually or retry later."* Never fabricate an extraction when the tools failed.

If `gemini` isn't available at all, skip straight to Claude API (no retry ladder needed).

Extracted item categories:

- **Insight** — non-obvious argument, counter-intuitive claim, or mental model
- **Reference** — a specific book, person, paper, tool, library, framework, or fact worth keeping
- **Action item** — a concrete thing the listener could do
- **Counter-argument** — where the host or guest is likely wrong (often empty; that's fine)
- **Quote** — a verbatim line (≤40 words) worth remembering

Only include categories the user's extraction hint cares about.

### 5. Write to staging

Write the extraction to `~/.claude/podcast-process/extractions/<YYYY-MM-DD>-<episode-slug>.md`:

```markdown
# <Episode Title> — <Podcast Name>

**URL:** <original URL>
**Transcript URL:** <where you found the transcript>
**Host(s) / Guest(s):** <names>
**Processed:** <YYYY-MM-DD>
**Extraction hint:** <what the user asked for, or "generic">

## Summary (2-3 sentences)

<plain-language summary>

## Insights
- ...

## References
- ...

## Action items
- ...

## Counter-arguments
- ...

## Quotes

> "..." — <speaker>

## Source transcript

<cached path or original URL>
```

### 6. Route the result

Two paths depending on how the skill was invoked:

**A. Invoked from an inbox/daily-notes task line** (e.g. user is triaging `Inbox.md` and asks to run the skill on `- [ ] **Task:** Extract useful info from <URL>`):

Replace the originating task line in-place with a condensed extraction summary. Format:

```markdown
- **Reference:** <Episode title> — <Podcast name> (host info). Source: <URL> (from [[<original date>]])
	- **<Insight 1 headline>** — one-line condensation
	- **<Insight 2 headline>** — one-line condensation
	- (3-6 bullets total; pick the highest-signal points)
	- **Caveat:** <if extraction came from a non-verbatim source like a written summary, flag it>
	- Full extraction (with quotes + counter-arguments): `<staging file path>`
```

The full extraction still gets written to staging at `~/.claude/podcast-process/extractions/<date>-<slug>.md` — that's where quotes, counter-arguments, and detail live. The inbox line just gets the condensed reference + a backlink.

This closes the loop: the user invokes the skill from triage and gets the extraction *in the same place they started*, no separate routing step.

**B. Invoked standalone** (user pastes a URL without inbox context):

Just write the full extraction to staging. Tell the user the path and ask where they want it routed (or leave it for later).

### 7. Report back

Tell the user:
- Path to the staging extraction markdown
- For path A: confirmation that the inbox line was replaced
- Where the transcript / source came from (so they can verify claims)
- Any caveat about source quality (e.g. "extracted from a written summary, not verbatim")

## State & caching

```
~/.claude/podcast-process/
├── transcripts/           # cached transcripts (keep — extraction is free once you have the text)
│   └── <episode-slug>.md
└── extractions/           # final outputs
    └── <YYYY-MM-DD>-<slug>.md
```

Cache transcripts aggressively. Once you have the text, re-extraction with a different hint is essentially free.

## Safety

- **Respect copyright** — transcripts are typically free to read but don't republish full transcripts as your own content
- **Never fabricate a transcript** — if you can't find one, stop
- **Flag verification concerns** — if the extraction relies on a transcript you couldn't verify (e.g. Gemini synthesised from its own knowledge rather than the actual transcript), note that clearly in the output

## Reference files

- `references/transcript-sources.md` — per-podcast index of known transcript locations + fallback search strategies
- `references/extraction-prompts.md` — the prompts used, tuning guide, category definitions

Load on demand.
