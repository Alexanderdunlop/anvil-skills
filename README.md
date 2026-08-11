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

Pre-alpha, `0.1.0`, published nowhere. All eight skills exist. Most of them have
never run.

That distinction is the honest one and it is worth stating plainly:

| Milestone | State                                                                                |
| --------- | ------------------------------------------------------------------------------------ |
| M0, M1    | done — the contract, the entry format, and `/scope` verified on the installed plugin |
| M2–M7     | written, and carrying about forty checks that have not been run                      |

The checks are in [evals/README.md](evals/README.md), each one written before the
run rather than after it. Until they pass, treat every command except `/scope` as
a specification that happens to be executable.

## Install

```bash
claude plugin marketplace add /path/to/anvil-skills
claude plugin install anvil-skills
```

All eight skills are discovered from `skills/` and appear as `/name`. None of
them fires on its own — every one carries `disable-model-invocation: true`,
because a command that writes files and waits at approval gates should run when
you say so and not when a model thinks it seems relevant.

## First run

```
/setup     once per repo, creates .anvil/ from questions
/scope     a rough idea becomes one sliced ticket, behind a gate
/kickoff   the ticket becomes CONTEXT.md, PLAN.md and TEST_CASES.md, behind a gate
/build     the plan becomes code, and every stall becomes a log entry
/verify    every case gets PASS, FAIL or COULD NOT CHECK, with evidence
/review    was it scoped right, did the slice hold
/feedback  distils the logs into the context files the others read
/research  turns what could not be routed into a question for outside
```

Run `/clear` between commands. Each one is written to start cold from files
alone, and that property is only real if it is exercised.

Without `.anvil/`, the other seven stop and point at `/setup`. None of them
creates a file, guesses a config, or proceeds on defaults — missing is a clean
state, half-populated is not.

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
