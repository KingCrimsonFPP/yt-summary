# Remove the dead `load_last_video` state function

Status: ready-for-agent

## What to build

`load_last_video` has no production callers — only tests exercise it. The `ask` skill reads `scripts/output/.last_video` directly. The two-function state module is shallow (interface ≈ implementation) and half-dead, and the state path is restated in several docs with the README disagreeing.

Delete `load_last_video` and its two tests. Keep `save_last_video` (still called from `main`). Fix the README path drift so it names the actual location the script writes (`scripts/output/.last_video`, not `output/.last_video`).

## Acceptance criteria

- [ ] `load_last_video` removed from `scripts/yt_transcript.py`
- [ ] `test_save_and_load_last_video` and `test_load_last_video_returns_none_when_missing` removed (or the save half kept as a `save_last_video`-only test)
- [ ] `save_last_video` still called from `main`
- [ ] README references the correct `.last_video` path
- [ ] `pytest` is green

## Blocked by

None - can start immediately
