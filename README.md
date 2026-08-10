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

**Nothing is lost from the logs. Things are lost from the context files, on
purpose.** The two logs are append-only and permanent — a correction you gave is
still recoverable long after it stopped mattering. The `.anvil/` context files
are a distillation of them under hard line budgets, so a line is evicted when
something beats it, when the same constraint turns up in your `CLAUDE.md`, or
when it has gone quiet for long enough that you agree to drop it. That eviction
is lossy by design: the log still holds the line, and the file's job is to stay
small enough to be worth reading on every run.

## Status

Pre-alpha. One of the eight commands exists.

M0 and M1 are done: the file contract and the feedback entry format were
hand-authored from real corrections, and `/scope` is built and verified against
the installed plugin rather than the working tree. It installs from a local
marketplace and is published nowhere. `/feedback` is next, and until it exists
nothing is distilled and nothing compounds — the logs just accumulate.

## Docs

- [docs/ORIGIN_IDEA.md](docs/ORIGIN_IDEA.md) — the original note this repo is based on
- [docs/ROADMAP.md](docs/ROADMAP.md) — the milestones, M0 to M7
- [docs/FILE_CONTRACT.md](docs/FILE_CONTRACT.md) — the file format and its rules
- [docs/PHILOSOPHY.md](docs/PHILOSOPHY.md) — why context is treated as a tax, and
  why the budgets are hard
- [evals/README.md](evals/README.md) — how each command is re-tested, and the
  ways those checks have already misled someone

## This repo's own `.anvil/`

This repo ships a `.anvil/` directory deliberately, as its real working history.
It is never read when anvil runs in another repo: every path in every skill is
`${CLAUDE_PROJECT_DIR}/.anvil/...`, which resolves to the repo you are working
in rather than to the plugin's copy of this one.

That matters more than it sounds. Marketplace plugins are copied into
`~/.claude/plugins/cache/`, so this repo's `.anvil/` lands on every user's
machine, and a bare relative path would be ambiguous between two real `.anvil/`
trees on disk — with the wrong one holding a perfectly valid-looking
`CONFIG.md`. The check that this holds runs against the installed plugin in a
different repo, because reading the file cannot prove it.
