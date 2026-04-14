# URL Resolution

How to get from a podcast URL to an audio file.

## Pocket Casts (`pca.st/episode/...`)

The URL redirects to the episode's public page. The page contains a link to the show's RSS feed and direct audio URL.

```bash
# Follow redirects and grab the audio link from the page metadata
curl -sL "https://pca.st/episode/<id>" | grep -oE 'https://[^"]+\.(mp3|m4a)' | head -1
```

If that fails, find the show via its ID on pca.st, get the RSS URL, and fetch the episode from the feed.

## Spotify (`open.spotify.com/episode/...`)

Spotify doesn't expose audio directly in the web page. Three paths, in order of reliability:

1. **Listen Notes API** — look up the episode, get the public RSS feed
2. **`spotdl` CLI** — `spotdl download "<spotify-url>"` (installs via `pip install spotdl`)
3. **Episode title → RSS** — if you know the show, find its RSS on [PodcastIndex.org](https://podcastindex.org) and match the episode by title/date

## Apple Podcasts (`podcasts.apple.com/.../id<id>`)

Apple shows the episode in a web page but not the audio. However, the show's RSS feed is accessible:

```bash
# Get the iTunes Collection ID
COLLECTION_ID=<the id from the URL>

# iTunes Lookup API returns the feedUrl
curl "https://itunes.apple.com/lookup?id=$COLLECTION_ID" | jq -r '.results[0].feedUrl'
```

Then fetch the feed and match the episode by title or pubDate.

## Direct RSS / MP3

Just download with `curl -L -o <output>.mp3 <url>`.

## YouTube (`youtube.com/watch`, `youtu.be/...`)

```bash
# yt-dlp handles audio extraction
yt-dlp -x --audio-format mp3 -o '%(title)s.%(ext)s' <url>
```

Install: `pipx install yt-dlp` or `brew install yt-dlp`.

## Slug generation

For the cache filename, generate a slug from the podcast name + episode title:

```
<podcast-name>--<episode-title>
→ lowercased, punctuation stripped, spaces → hyphens, truncated to 80 chars
```

Example: `my-first-million--the-guy-who-sold-1-5m-in-48-hours`

## Metadata

Extract and keep:
- Original URL (for back-linking)
- Episode title
- Podcast name
- Duration (seconds) — helps estimate transcription cost
- Publish date
- Guest names (often in the title or show notes)
- Show notes URL if available

Store metadata as JSON sidecar: `~/.claude/podcast-process/cache/<slug>.json`.
