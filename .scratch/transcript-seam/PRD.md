# PRD: Keep the transcript seam structured

Status: ready-for-agent

## Problem Statement

When a user asks "where in the video was X discussed?", the `ask` skill returns timestamp links that can be imprecise. Under the hood, the fetch already knows each snippet's exact start time in seconds, but it throws that away — it flattens the transcript into a formatted string (`[01:23] text…`) before any skill sees it. The `ask` skill then has to reverse-parse `[MM:SS]` back into seconds to build a `&t=` jump link, a lossy round-trip that can land the viewer a second off.

More broadly, the fetch interface destroys structure at the seam: every consumer receives a pre-formatted string and must re-derive structure it never should have lost.

## Solution

Keep the transcript structured across the seam. The fetch returns a list of **snippets** (`{start, text}`, `start` in seconds) and holds the API fetch + fallback logic; formatting becomes a thin presentation applied at the edge in one of three formats — `plain`, `timestamps`, or `json`. The CLI selects the format.

The `ask` skill consumes `json` and reads the exact `start` seconds directly, so jump links are precise and the `[MM:SS]` display is formatted *from* the seconds rather than parsed back into them.

From the user's perspective: timestamp links in `ask` land exactly on the moment, and the plugin gains a structured `json` output other tools can build on.

## User Stories

1. As a viewer using `ask`, I want jump links that land on the exact second a topic was discussed, so that I don't have to scrub around after clicking.
2. As a viewer using `ask`, I want the `[MM:SS]` shown next to each quote to match where the link actually jumps, so that the display and the link agree.
3. As a user of the `transcript` skill, I want the existing timestamped output to keep working unchanged, so that my current workflow doesn't break.
4. As a user of the `summarize` skill, I want the plain transcript output to keep working unchanged, so that summaries are unaffected.
5. As a script user, I want my existing `--timestamps` invocations to keep working, so that I'm not forced to migrate immediately.
6. As a script user, I want a `--format json` option, so that I can pipe structured snippets into my own tooling.
7. As a script user, I want a single `--format {plain,timestamps,json}` option, so that there's one clear way to choose output shape.
8. As a maintainer of the deprecated summarize script, I want `get_transcript` to keep working, so that the deprecated path isn't disturbed by this change.
9. As a maintainer, I want timestamp math to live in one place, so that a formatting bug is fixed once.
10. As a maintainer, I want the fetch and the formatting to be separately testable, so that I can assert each without the network.
11. As a contributor, I want the snippet to be the documented unit of a transcript, so that new features build on structured data instead of strings.

## Implementation Decisions

- **`fetch_snippets(video_id, lang_priority=None) -> list[dict]`** — new deep fetch. Returns `{"start": float, "text": str}` snippets. Holds the current behavior: fetch in priority languages, fall back to the first available transcript if none match. Decouples callers from the `youtube_transcript_api` object shape (removes the defensive `getattr` access at the seam).
- **`format_transcript(snippets, fmt) -> str`** — new pure presentation. `fmt ∈ {plain, timestamps, json}`. `plain` joins text; `timestamps` prefixes `[MM:SS]`/`[HH:MM:SS]` via the existing `format_timestamp`; `json` serializes the snippet list (preserving `start` as a number).
- **Snippet shape** — plain dict `{"start": float, "text": str}`, chosen over the library's native object or a dataclass: lowest ceremony, JSON-serializable for free, trivial test fixtures.
- **CLI** — add `--format {plain,timestamps,json}`. Keep `--timestamps` as a deprecated alias for `--format timestamps` (non-breaking for existing skills and documented direct usage).
- **`get_transcript`** — retained as a thin back-compat wrapper delegating to `fetch_snippets` + `format_transcript`, so the deprecated importer is untouched.
- **`ask` skill** — invokes `--format json`, reads `start` seconds directly to build `&t=<seconds>` links, formats the `[MM:SS]` display from those seconds.
- **Glossary** — `Snippet`, `Transcript`, `Video ID` defined in `CONTEXT.md`; use those terms throughout.

## Testing Decisions

- **Good test = external behavior, not implementation.** Assert what crosses the seam (the returned snippets, the formatted strings), never private structure.
- **`format_transcript`** — primary seam. Pure function, no I/O. Assert `plain`, `timestamps`, and `json` output for representative snippet lists, including the hour-boundary case.
- **`fetch_snippets`** — in-process seam with the `YouTubeTranscriptApi` client mocked. Assert the `{start, text}` shape and the fallback-to-first-available-transcript path. No network.
- **Prior art** — existing tests import functions directly and `patch` module globals (`STATE_FILE`); follow that pattern for mocking the API client. Existing `format_timestamp` and `extract_video_id` tests stay green.
- **`ask` skill** — not unit-tested (markdown driving Claude); verified by demo: `json` output produces a precise `&t=` link.

## Out of Scope

- The `--last` CLI subcommand and any change to how `.last_video` is read (the `ask` skill still reads the file directly). Removing the dead `load_last_video` is tracked separately as issue 02.
- Touching `deprecated/summarize_youtube.py`.
- Sub-second timestamp display precision (links use exact seconds; display stays `[MM:SS]`).
- Language-selection UX (the `lang_priority` default is unchanged).

## Further Notes

Two issues already drafted under this feature: `01-fetch-structured-snippets` (this PRD's scope) and `02-remove-dead-last-video` (independent cleanup). Both `ready-for-agent`, neither blocked. Test at the function-import seam, not a new subprocess/CLI seam.
