# Fetch transcripts as structured snippets with `--format json`

Status: ready-for-agent

## What to build

Stop flattening the transcript to a string inside the fetch. The fetch should return structured **snippets** (`{start, text}`, `start` in seconds) and keep them structured across the seam; formatting becomes a thin presentation chosen at the CLI edge.

Split the current fetch-and-format into two: a fetch that returns the list of snippets (holding the existing API fetch + fallback-to-first-available-transcript logic), and a formatter that renders a list of snippets into one of three formats — `plain`, `timestamps`, or `json`. The CLI selects the format via a new `--format` option. The existing `--timestamps` flag stays as a deprecated alias for `--format timestamps` so existing skills and any direct script usage keep working. `get_transcript` remains as a thin back-compat wrapper that delegates to the two new functions (the deprecated script keeps importing it untouched).

The `ask` skill switches to `--format json` and reads the exact `start` seconds straight off each snippet to build `&t=<seconds>` links — no more reverse-parsing `[MM:SS]`. It formats the `[MM:SS]` display *from* the seconds.

End-to-end demoable: `python scripts/yt_transcript.py <id> --format json` emits structured snippets; `ask` produces precise timestamp links from them.

## Acceptance criteria

- [ ] `fetch_snippets(video_id, lang_priority=None)` returns a list of `{"start": float, "text": str}`, preserving the current API fetch and fallback behavior
- [ ] `format_transcript(snippets, fmt)` renders `plain`, `timestamps`, and `json`
- [ ] CLI accepts `--format {plain,timestamps,json}`; `--timestamps` still works as a deprecated alias for `--format timestamps`
- [ ] `get_transcript` still exists and delegates to the two new functions (`deprecated/summarize_youtube.py` import unchanged)
- [ ] `ask` skill (`skills/ask/SKILL.md`) invokes `--format json` and builds `&t=` links from exact `start` seconds
- [ ] Tests cover `fetch_snippets` (with the API mocked) and `format_transcript` for each format; existing `format_timestamp` tests still pass
- [ ] `pytest` is green

## Blocked by

None - can start immediately
