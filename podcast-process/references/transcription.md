# Transcription

Default path: OpenAI Whisper API. Fallback: local `whisper.cpp`.

## OpenAI Whisper API

Cost: **$0.006 per minute** (2026 pricing — verify). 60 min = $0.36. 3-hr MFM episode = ~$1.08.

Always cache transcripts. Never re-transcribe the same audio file.

```bash
curl https://api.openai.com/v1/audio/transcriptions \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -F file=@<audio-file> \
  -F model=whisper-1 \
  -F response_format=text \
  > <cache>/transcript.txt
```

## File size limit: 25MB

Whisper API rejects files >25MB. For longer episodes, split with ffmpeg:

```bash
# Split into 20-minute chunks
ffmpeg -i input.mp3 -f segment -segment_time 1200 -c copy "chunk-%03d.mp3"

# Transcribe each, concatenate
for f in chunk-*.mp3; do
  curl https://api.openai.com/v1/audio/transcriptions \
    -H "Authorization: Bearer $OPENAI_API_KEY" \
    -F file=@$f -F model=whisper-1 -F response_format=text
  echo ""
done > full_transcript.txt
```

Chunk boundaries can cut words. This is usually fine for extraction — losing one word in a 3-hour transcript won't change insights.

## Compression for faster upload

Audio files can be heavy. Compress before uploading:

```bash
ffmpeg -i input.mp3 -b:a 32k -ac 1 -ar 16000 output.mp3
```

This drops quality enough for speech but keeps Whisper accurate. A 60-min file drops from ~60MB to ~15MB.

## Language

Whisper auto-detects. If the episode is in a language other than English, pass `language=<iso-code>` for more accurate output:

```bash
-F language=it    # Italian
-F language=es    # Spanish
```

## Local fallback: whisper.cpp

If the user is offline or doesn't want API costs:

```bash
# Install
brew install whisper-cpp    # macOS
# or build from source: https://github.com/ggerganov/whisper.cpp

# Download a model (once)
bash ./models/download-ggml-model.sh base.en    # fast, decent
bash ./models/download-ggml-model.sh medium     # slower, much better

# Transcribe
whisper-cli -m models/ggml-medium.bin -f audio.wav > transcript.txt
```

`whisper.cpp` expects WAV input. Convert first:

```bash
ffmpeg -i input.mp3 -ar 16000 -ac 1 input.wav
```

**Quality comparison:** API (`whisper-1`) ≈ `large-v3` local model. `medium` local is noticeably worse but still workable. `base.en` is usable for clear speech but miss-prone in crosstalk.

## Speaker diarization

Whisper doesn't identify speakers. For interview formats, the extraction prompt handles this — ask Claude to guess speaker turns from context (host asking questions vs. guest answering). It's imperfect but fine for extraction.

For high-quality diarization, use `pyannote.audio` or `whisperx`. Overkill for most extractions.
