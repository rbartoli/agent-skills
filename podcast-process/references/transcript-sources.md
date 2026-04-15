# Transcript Sources

Where to find public transcripts for popular podcasts. Use this as your first-check index before falling back to web search.

## AI / Tech podcasts with reliable transcripts

| Podcast | Host(s) | Transcript location |
|---------|---------|---------------------|
| **Latent Space** | swyx, Alessio Fanelli | `latent.space/p/<slug>` — every episode has a full transcript on the Substack post |
| **No Priors** | Sarah Guo, Elad Gil | `no-priors.com` episode pages, also on their Substack |
| **The Changelog** | Adam Stacoviak, Jerod Santo | `changelog.com/podcast/<num>` — full transcripts below the audio player |
| **a16z Podcast** | Steph Smith et al. | `a16z.com/podcast/` — full transcripts linked in show notes |
| **Lex Fridman Podcast** | Lex Fridman | `lexfridman.com/<guest-slug>` — full transcripts with timestamps |
| **Acquired** | Ben Gilbert, David Rosenthal | `acquired.fm/episodes` — detailed show notes + transcripts |
| **The Gradient Podcast** | Daniel Bashir | `thegradientpub.substack.com` — transcripts on Substack posts |

## Health / self-improvement

| Podcast | Host | Transcript location |
|---------|------|---------------------|
| **Huberman Lab** | Andrew Huberman | `hubermanlab.com/podcast/<slug>` — full transcripts |
| **The Tim Ferriss Show** | Tim Ferriss | `tim.blog/<year>/<month>/<day>/<slug>` — full transcripts |
| **Found My Fitness** | Rhonda Patrick | `foundmyfitness.com/episodes` — transcripts + show notes |

## Business / startup

| Podcast | Host(s) | Transcript location |
|---------|---------|---------------------|
| **My First Million** | Sam Parr, Shaan Puri | No official transcripts; search YouTube auto-captions for the mirrored video upload |
| **This Week in Startups** | Jason Calacanis | Uneven — check thisweekinstartups.com; fall back to YouTube captions |
| **Invest Like the Best** | Patrick O'Shaughnessy | `joincolossus.com/episodes` — transcripts in episode posts |

## Fallbacks for unknown / long-tail shows

1. **YouTube auto-captions** — if the episode is on YouTube (many podcasts upload video):
   ```bash
   yt-dlp --write-auto-subs --skip-download --sub-format vtt --sub-lang en -o '/tmp/%(id)s' <url>
   ```
   Clean VTT timestamps into plain text before extraction.

2. **Podscribe** — `podscribe.com/search?q=<episode title>` — third-party auto-transcripts for many podcasts. Quality varies.

3. **Web search** — `gemini -p "Find the transcript URL for <episode title> on <podcast name>"`. Verify the returned URL with defuddle before trusting.

4. **Show's own site** — many podcasts publish transcripts on their website even if not linked from the episode page directly. Try `site:<podcast-domain> "<episode title>"`.

## How to verify you got the real transcript

A real transcript is:
- **Long** — thousands of words for a 1-hour episode
- **Dialogue-formatted** — speaker names, back-and-forth exchanges
- **Specific** — names, numbers, project names, direct quotes

If what you fetched looks like show notes (a few paragraphs of summary), a teaser, or marketing copy, you don't have the transcript — keep looking.

## When nothing works

If after all sources the transcript doesn't exist:

1. Stop and tell the user clearly
2. Suggest alternatives: watch the YouTube upload and use YouTube's built-in transcript UI, or use a dedicated transcription service
3. Do NOT fabricate content from memory or synthesise from the episode title

Transcription is explicitly out of scope for this skill. The honest answer "no transcript available" is always better than a hallucinated extraction.
