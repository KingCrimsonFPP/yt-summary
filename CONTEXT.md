# yt-summary

A Claude Code plugin that fetches YouTube transcripts and drives summarize / transcript / ask workflows over them.

## Language

**Snippet**:
A single transcript segment — a `{start, text}` pair where `start` is the offset in seconds. The unit that crosses the fetch seam; formatting is applied to a list of snippets, never baked into them.
_Avoid_: caption, line, segment object, cue

**Transcript**:
The ordered list of snippets for one video. Rendered into a `plain`, `timestamps`, or `json` format at the edge; the in-memory transcript stays structured.
_Avoid_: captions, subtitles

**Video ID**:
The canonical 11-character YouTube identifier extracted from any URL or bare id. The single key every workflow keys off.
_Avoid_: url, link, slug
