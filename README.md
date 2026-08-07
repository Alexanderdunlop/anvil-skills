# anvil-skills

Claude Code skills that learn from your corrections. Scope, build, verify, then
distil the feedback back into the context files so the next ticket goes better.

Every command logs corrections as they happen, and logs the moments Claude got
stuck or had to guess. `/feedback` is the only command that reads those logs,
and it distils what it finds into the `.anvil/` context files the other commands
load. A lesson from one ticket is already in place on the next, so the process
compounds instead of resetting each session.

The two kinds of entry are kept in separate files — `feedback/human.jsonl` and
`feedback/self.jsonl` — but they share one threshold. A lesson you flagged once
and Claude later hit on its own counts as two sightings, not one each, which
makes it the best-evidenced kind of lesson in the system rather than the kind
that falls through the gap.

## Status

Pre-alpha. Nothing is built and nothing is installable yet. Currently at M0.

## Docs

- [docs/ORIGIN_IDEA.md](docs/ORIGIN_IDEA.md) — the original note this repo is based on
- [docs/ROADMAP.md](docs/ROADMAP.md) — the milestones, M0 to M7
- [docs/FILE_CONTRACT.md](docs/FILE_CONTRACT.md) — the file format and its rules
- `docs/PHILOSOPHY.md` — why context is treated as a tax. Not written yet; it is
  an M0 deliverable, and `FILE_CONTRACT.md` §1 points at it

## This repo's own `.anvil/`

Once M0 lands, this repo will ship a `.anvil/` directory deliberately, as its
real working history; it is never read when anvil runs in another repo.
