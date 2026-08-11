# anvil-skills

Claude Code skills that learn from your corrections. Scope, build, verify, then
distil the feedback back into the context files so the next ticket goes better.

Correct Claude once and it stays corrected. Every command logs the corrections
you make and the moments Claude got stuck or had to guess, and `/feedback`
captures the rest — the things you only say afterwards, in ordinary conversation.

`/improve` reads those logs and recommends what should enter the `.anvil/`
context files the other commands load. You approve, it writes, and the lesson is
in place for every run after that — so the process compounds instead of resetting
every session.

```bash
claude plugin marketplace add https://github.com/Alexanderdunlop/anvil-skills
claude plugin install anvil-skills
```

Then `/setup` once per repo, and `/scope` when you have an idea.

---

## The loop

```
/scope ──gate──► /kickoff ──gate──► /build ──► /verify ──► /review
   │                 │                 │           │           │
   └─────────────────┴─────────────────┴───────────┴───────────┘
        every command appends to .anvil/feedback/{human,self}.jsonl
                    /feedback adds what you say outside a run
                                      │
                                  /improve
                                      │
                    recommends, you approve, it writes
                                      │
                  read by the commands that need them, next run
```

| Command     | Does                                                                       |
| ----------- | -------------------------------------------------------------------------- |
| `/setup`    | creates `.anvil/` from questions, once per repo                            |
| `/scope`    | a rough idea becomes one sliced ticket with observable acceptance criteria |
| `/kickoff`  | the ticket becomes `CONTEXT.md`, `PLAN.md` and `TEST_CASES.md`             |
| `/build`    | the plan becomes code, and every stall becomes a log entry                 |
| `/verify`   | every case gets `PASS`, `FAIL` or `COULD NOT CHECK`, with evidence         |
| `/review`   | was it scoped right, did the slice hold, does the diff match the plan      |
| `/feedback` | captures what you say into the logs, drafted from the conversation         |
| `/improve`  | reads the logs, recommends context-file changes, writes on your yes        |
| `/research` | turns what could not be routed into a question aimed outside the repo      |

`/scope` and `/kickoff` stop at an approval gate and wait. Nothing proceeds on
"sure, I guess" — anything you would have to interpret is a revision request,
not approval.

Run `/clear` between commands. Each one starts cold from files alone, which is
what makes the context files worth having and the budgets worth enforcing.

**One writer, and it asks first.** Every command writes feedback entries. Only
`/improve` edits a context file, and it edits nothing you have not approved in
that run. Two writers to the same file is how it becomes a landfill; one writer
that never asks is how it fills with plausible lines nobody agreed to.

---

## Why the files stay small

A line in a context file is paid every time the command that reads it runs —
forever. So the default answer to "should this go in a context file?" is no.

**Nothing enters without passing all three questions:**

1. Could Claude have inferred this from the code? If yes, drop it.
2. Is it stated as a direct constraint rather than a suggestion? If not, rewrite
   it until it is.
3. Does it fit on one line? If not, it is two lessons or it is not a lesson.

Most corrections fail question 1. They die in the logs, and that is the system
working.

**Lessons are written at the narrowest scope that holds them**, and promoted
only on evidence:

| Scope   | Lands in                     | Read when                       | Needs              |
| ------- | ---------------------------- | ------------------------------- | ------------------ |
| Ticket  | `tickets/<id>/CONTEXT.md`    | once, during that ticket        | —                  |
| Command | `process/<command>.md`       | that command runs               | 2 distinct tickets |
| Global  | proposed to your `CLAUDE.md` | **every session, anvil or not** | 3 distinct tickets |

Nothing reaches the global tier on first sighting, and anvil never writes your
`CLAUDE.md` — at three sightings it proposes a line and you decide.

**Budgets are hard ceilings.** At the limit, adding a line means removing one, so
`/improve` names the line it would evict, says why the new one is worth more, and
lets you pick either or neither. When nothing existing is weaker it stops and
tells you — it never raises its own ceiling, because the whole meaning of raising
one is that a person looked at the trade and chose to pay more. Set your own
numbers in `.anvil/BUDGETS.md`; the shipped defaults are deliberately tight.

**Nothing is written that you did not approve.** `/improve` recommends —
additions, swaps, and removals — and you take them line by line. Lines leaving is
where that matters most: a line quietly disappearing is harder to notice than one
quietly appearing.

---

## The two logs

Corrections you made live in `feedback/human.jsonl`. Stalls Claude noticed in
itself live in `feedback/self.jsonl`. Same schema, split by who noticed, so
"show me only what I actually corrected" is answerable with `cat` and no tooling.

They share one threshold. **A lesson you flagged once and Claude later hit on
its own counts as two sightings, not one each** — that alternating case is the
strongest evidence in the system, because it means the model independently hit
what a person had already flagged.

**Nothing is lost from the logs. Things are lost from the context files, on
purpose.** The logs are append-only and permanent. The context files are a
distillation under hard budgets, so a line leaves when something beats it, when
the same constraint turns up in your `CLAUDE.md`, or when it has gone quiet long
enough that you agree to drop it. Every removal is written back to the log. A
line leaving a context file is a line no longer paid for on every run — not a
line deleted from the system.

---

## What it does not do

- **No always-on cost.** Every skill carries `disable-model-invocation: true`
  and none of them fires because a model thought it seemed relevant. You pay for
  the skill descriptions and nothing else until you type a command.
- **No telemetry, no endpoint, nothing phoned home.** `/research` emits a prompt
  into your session. If you want it to reach anyone, you paste it.
- **No writing your `CLAUDE.md`.** Anvil proposes; you edit.
- **No `CONFIG.md` entries for things it could read.** Build, test and lint
  commands, entry points, framework names — all inferable, all excluded. A stale
  lookup gets re-run; a stale _line_ gets trusted.

---

## Status

`0.1.0` and early. The file format has been through eight milestones of design
and is documented in full, and the shipped defaults are guesses about a repo
anvil has never seen — yours. If a budget is wrong for your codebase, change it
in `BUDGETS.md`. If something is wrong with anvil, please
[open an issue](https://github.com/Alexanderdunlop/anvil-skills/issues).

There is a research issue template for the most useful kind of report: a pattern
your repo kept parking that the routing table has no home for. That is the only
path by which evidence from a repo I will never see reaches the design.

## Docs

- [docs/FILE_CONTRACT.md](docs/FILE_CONTRACT.md) — every file, its format, its
  budget, and the rules that govern it. The authority
- [docs/PHILOSOPHY.md](docs/PHILOSOPHY.md) — why context is treated as a tax,
  and why the budgets are hard
- [docs/ROADMAP.md](docs/ROADMAP.md) — how it was built, in the order it was
  built, and why `/setup` came last
- [docs/ORIGIN_IDEA.md](docs/ORIGIN_IDEA.md) — the original note this is based on
- [evals/README.md](evals/README.md) — how each command is re-tested, and the
  ways those checks have already misled someone

## This repo's own `.anvil/`

This repo ships a `.anvil/` directory deliberately, as its real working history.
You can read what anvil actually learned about building itself rather than take
this page's word for it.

It is never read when anvil runs in your repo. Every path in every skill is
`${CLAUDE_PROJECT_DIR}/.anvil/...`, which resolves to the repo you are working
in rather than to the plugin's copy of this one.

That matters more than it sounds. Marketplace plugins are copied into
`~/.claude/plugins/cache/`, so this repo's `.anvil/` lands on your machine, and a
bare relative path would be ambiguous between two real `.anvil/` trees on disk —
with the wrong one holding a perfectly valid-looking `CONFIG.md`. The check that
this holds runs against the installed plugin in a different repo, because reading
the file cannot prove it.

## Licence

MIT.
