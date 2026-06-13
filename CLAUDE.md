## Project Context
- Plugin type: Claude Code marketplace plugin for YouTube transcript/summary workflows
- Stack: Python (scripts), Markdown skills (SKILL.md pattern), pytest for tests
- Skills live in `skills/<name>/SKILL.md`

## Agent skills

### Issue tracker

Issues and PRDs live as markdown files under `.scratch/<feature>/`. See `docs/agents/issue-tracker.md`.

### Triage labels

Five canonical triage roles, all using their default strings (`needs-triage`, `needs-info`, `ready-for-agent`, `ready-for-human`, `wontfix`). See `docs/agents/triage-labels.md`.

### Domain docs

Single-context: one `CONTEXT.md` + `docs/adr/` at the repo root. See `docs/agents/domain.md`.

## Lab Notes
> When Claude makes a mistake that required correction, append a one-liner here.
> These are harvested by /insights-collect and cleared after capture.

## Failure Log
> Permanent learnings promoted by /insights-apply from past lab notes.
