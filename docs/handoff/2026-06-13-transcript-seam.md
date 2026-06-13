# Handoff — transcript-seam deepening (yt-summary plugin)

**Repo:** `fpp-yt-summary` (origin: github.com/KingCrimsonFPP/fpp-yt-summary) · branch `master`
**Date:** 2026-06-13 · **Reason for handoff:** continuing on a different laptop

## ⚠️ Read first — push before switching

Everything this session produced lives in the working tree. Commit and push it (including this doc) so it reaches the other laptop. New/changed paths:

- `CLAUDE.md` — added `## Agent skills` block
- `docs/agents/{issue-tracker,triage-labels,domain}.md` — Matt Pocock skills config (local-markdown tracker, default labels, single-context)
- `CONTEXT.md` — domain glossary (Snippet, Transcript, Video ID)
- `docs/handoff/2026-06-13-transcript-seam.md` — this doc
- `.scratch/transcript-seam/PRD.md` — the spec
- `.scratch/transcript-seam/issues/01-fetch-structured-snippets.md`
- `.scratch/transcript-seam/issues/02-remove-dead-last-video.md`

## What happened this session

1. Ran `/setup-matt-pocock-skills` — configured the engineering skills for this repo (issue tracker = local markdown under `.scratch/`, default triage labels, single-context domain docs).
2. Ran `/improve-codebase-architecture` — found three deepening candidates (report was a temp HTML file, now stale). Picked Candidate 1.
3. Grilled the design tree to closure (decisions are all captured in the PRD's *Implementation Decisions*).
4. Ran `/to-issues` → published issues 01 and 02.
5. Ran `/to-prd` → published `PRD.md`.

## Where things stand

- Design is fully settled — no open questions. See `.scratch/transcript-seam/PRD.md`.
- Both issues are `Status: ready-for-agent`, neither blocked, both AFK-implementable.
- Pipeline position: brainstorm → architecture → to-issues ✓ → to-prd ✓ → **triage → tdd (next)**.
- Test seam decided: function-import seam (mock `YouTubeTranscriptApi`), not a subprocess/CLI seam. `format_transcript` is the pure primary seam.

## Next steps (user will decide order on the other session)

1. Optionally `/triage` (not strictly needed — issues already `ready-for-agent`).
2. `/tdd` on issue 01 (`01-fetch-structured-snippets.md`) — the substantive slice. Then issue 02 (trivial cleanup).

## Suggested skills

- **`tdd`** — implement issue 01 test-first at the `format_transcript` / `fetch_snippets` seams.
- **`triage`** — only if adding/reordering incoming work; current two issues are already ready.
- **`verify`** — after building, confirm `--format json` and the `ask` skill's `&t=` links work against a real video.

## Gotchas

- Windows + Git Bash environment. Script writes state to `scripts/output/.last_video` (README currently mislabels this as `output/.last_video` — issue 02 fixes it).
- `deprecated/summarize_youtube.py` imports `get_transcript` — keep it as a thin wrapper (decision in PRD), don't touch the deprecated file.
- `--timestamps` must keep working as a deprecated alias for `--format timestamps`.
