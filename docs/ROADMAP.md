# Roadmap

How anvil was built, in the order it was built, and why that order.

Eight milestones. The ordering is not a build sequence chosen for convenience —
each milestone exists to retire one specific risk, and the risks are ordered so
that an early wrong answer is cheap to fix.

The single organising decision: **`/setup` is built last.** `/setup` is a
generator for the file format, and generators written against a moving format
get rewritten. The format has to stop moving first, which means it has to be used
in anger first.

---

## The milestones

| M   | Built                                    | Risk it retired                                                                          |
| --- | ---------------------------------------- | ---------------------------------------------------------------------------------------- |
| M0  | the contract, the entry format           | a schema designed against imagined corrections rather than real ones                     |
| M1  | `/scope`                                 | approval gates that do not hold; commands that cannot start cold; paths reading the wrong tree |
| M2  | `/feedback`                              | a compounding loop that is a diagram rather than a mechanism; budgets that are advisory  |
| M3  | six tickets, scoped by the tool          | a process that works on toy input and collapses on real input                            |
| M4  | `/kickoff` and `/build`                  | plan files too thin to build from, or too fat to read                                    |
| M5  | `/verify`                                | an MCP dependency quietly breaking any-repo portability                                  |
| M6  | `/review`                                | the `/verify`–`/review` overlap, and a second writer to context files                    |
| M7  | `/setup`, `/research`, packaging         | a generator written against a format that was still moving                               |

---

## M0 — the contract

The file format, the feedback entry schema, and the first entries — hand-written
from corrections that had actually been given, not from plausible ones.

That constraint is the milestone. A schema designed against imagined corrections
fits imagined corrections; the only way to find out what an entry needs is to
write down things that really happened and see which fields are missing.

`FILE_CONTRACT.md` is the authority from here on. Where a skill and the contract
disagree, the contract is right and the skill is a bug.

## M1 — `/scope`

One command end to end, proving the two hardest structural properties: a cold
start from files alone, and an approval gate that holds against a model that
wants to be helpful.

The gate is tested by replying with something deliberately ambiguous — "sure, I
guess" — and confirming nothing is written. The path rule is tested by installing
the plugin and running it in a *different* repo, because reading the file cannot
prove which `.anvil/` a path resolves to.

M1 changed the contract twice, which is the milestone working as intended:
`/scope` may read one prior ticket the user names, and the `fingerprint` field
grew a warning about prose keys. Both came from running the command rather than
from reading it.

## M2 — `/feedback`

The compounding loop, and budget resolution. Raw entries in, distilled context
files out, with the admission test and the budgets enforced mechanically rather
than by good intentions.

Built second rather than last, because everything after it depends on the loop
being real.

Budget resolution is three separate things and M2 keeps them separate: the
mechanism is not optional, the number is a project's to set, and the tool may
never move the number itself. At the ceiling with nothing weaker to evict,
`/feedback` fails, presents both ways out, and takes neither.

Three contract questions had to be answered before anything could be counted:

- **`ticket: null` is one bucket.** Every entry without a ticket counts as a
  single distinct ticket, so entries written in one sitting cannot promote a line
  between them, while a null plus a real ticket is two sightings
- **`command: "manual"`** is legal for entries written by hand
- **"tickets since"** is the count of distinct non-null ticket values in later
  entries — the one place `stale_after` is compared against anything

## M3 — the tickets

The six remaining commands are specified by the tool rather than by hand: six
tickets produced by `/scope`, all four sections, criteria numbered and
observable, every one inside the 40-line budget.

The unrouted pile was read and a decision recorded: no new category. One parked
entry argued for a `process:setup` key and was rejected — its lesson already
lives in the seeding rule, and `/setup` runs about twice in a repo's life, so a
sixth process file would be created and reasoned about by everyone to hold
lessons for the command that runs least.

Reading the pile produced a contract change. Two entries were questions that had
been *answered* at M2 and were still sitting there, because nothing ever took a
bullet out of `unrouted.md`. The count printed on exit is the only signal that
the category table is wrong, and a count that can only rise stops being one — so
`dropped` now also means "answered", a human says when that is true, and the
removal is logged like every other.

## M4 — `/kickoff` and `/build`

The plan-then-execute pair, built together because `/build` exists only to
consume `/kickoff`'s output and the interesting question is whether the plan
files are enough to work from.

`/build` starts cold from the ticket body and the plan trio, writes code and
nothing else, and logs every point the plan was not enough. The routing rule is
what makes that worth doing: **a hole in the plan is a `process:kickoff` lesson,
not a `process:build` one.** `command` records who saw it, `category` records who
should change, and getting that backwards files every lesson where it cannot be
acted on.

`/build` also carries its own justification. It earns its place if entries with
`command: "build"` produce lessons that improve `/kickoff`. If three tickets
produce no build entries, it is a wrapper around "read the plan" and `PLAN.md`
becomes the interface.

## M5 — `/verify`

The machine-checkable half of quality. Three verdicts — `PASS`, `FAIL`,
`COULD NOT CHECK` — and no opinions at all.

Every line in `TEST_CASES.md`, `CRITICAL_PATHS.md` and `REVIEW_RULES.md` gets a
verdict, because a case that was skipped is invisible in the output and a partial
run then looks like a clean one. Evidence is a command and what it returned;
where there is no command and no observation, the verdict is `COULD NOT CHECK`
rather than a pass nobody witnessed.

M5 closed the MCP question. An absent smoke driver and `smoke: none` are both
normal: the case gets `COULD NOT CHECK`, the line names which cause it was and
carries the manual steps, and the run continues. Never fail a run for want of a
smoke driver, never convert an unchecked case into a pass, never make the user
install something first — a command that only works where MCP works does not work
in any repo.

## M6 — `/review`

The judgment half: was this scoped right, is the PR one a person can review in
one sitting, does the diff do what the plan said, are the acceptance criteria
actually met.

`/review` writes only to the feedback logs. It is the command most likely to
break the one-writer invariant, because it produces exactly the kind of opinion
that looks like it belongs in `REVIEW_RULES.md`.

The overlap with `/verify` was real and specific: both read `REVIEW_RULES.md` and
both look at the diff. The split is now in the contract rather than in either
skill — `/verify` reads that file to know **what to check**, `/review` reads it to
know **what not to say**. A rule stated in one of two skills is a rule the other
drifts away from, which is how an overlap comes back after being closed once.

## M7 — `/setup`, `/research` and packaging

The generator, written once the format had survived six milestones of use.

`/setup` scans a repo to ask better questions, never to fill in answers.
Detecting a test runner tells you what to ask; it does not tell you what to
write, because a line anyone could read off the manifest fails the admission
test. Update mode changes no existing line and removes nothing.

`/research` reads the parked pile and both logs and writes nothing at all — not
even a log entry. It is built here rather than earlier because a contribution
channel is worthless until people can install the thing.

Two decisions were taken here. **`version: 0.1.0`**, which made
`claude plugin validate . --strict` pass for the first time — the missing-version
warning was its only diagnostic, and strict treats warnings as errors. And **no
`hooks/`**: none was ever needed, and a hook would be the first piece of anvil
that fires without being asked.

M7 also found the one legitimate exception to the path rule. `/setup` is the
first skill that *emits* Tree B content rather than *resolving* a path: the
`CONFIG.md` it generates carries a repo-relative `tickets:` value, and prefixing
that would make it generate a wrong file. The rule always governed paths a skill
resolves; the contract now says so, and the check skips fenced blocks so it can
tell the two apart.

---

## Choosing what to build against

Milestones M0–M3 were built against this repo itself. They exercise `/scope` and
`/feedback`, which need real corrections and nothing else, and a markdown repo is
sufficient for that.

From M4 the subject has to be a repo with something to build: a test runner, a
build, and a smoke path. `/verify`'s whole output is per-case verdicts with
command output as evidence, and a repo with no commands to run cannot produce
one.

`/setup` needs two different subjects and no single repo supplies both. **Fresh
generation** wants a repo that is alien in useful ways — an unusual test runner,
no npm, an existing `CLAUDE.md`, and critical paths that resist being written as
executable checks. The risk being retired there is *a generator that only ever
saw one stack*, and a repo whose stack shaped the format cannot retire it.
**The update path** wants a repo that has carried a working `.anvil/` for a
while, because that is the only honest test that re-running `/setup` is an update
rather than an overwrite.

A fresh-generation subject is never prepared in advance. It receives a `.anvil/`
generated by `/setup` and nothing else — hand-editing one to resemble a repo the
format already fits is how it stops being the unfamiliar case worth testing.

---

## Risk ledger

| M   | Risk retired                                                                                                                                                     |
| --- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| M0  | Schema designed against imagined corrections rather than real ones                                                                                               |
| M1  | Approval gates that do not hold; skills that cannot start cold; bare `.anvil/` paths reading the plugin cache instead of the user's repo                          |
| M2  | A compounding loop that is a diagram rather than a mechanism; budgets that are advisory in practice; a ceiling the tool can raise for itself; files that only grow |
| M3  | A process that works on toy input and collapses on real input                                                                                                    |
| M4  | Plan files too thin to build from or too fat to read; `/build` existing without a reason                                                                          |
| M5  | MCP dependency breaking any-repo portability; `/verify` drifting subjective                                                                                      |
| M6  | `/verify`–`/review` overlap; a second writer to context files                                                                                                    |
| M7  | A generator written against a moving format, and one that only ever saw a single stack; skills that do something unhelpful when `.anvil/` is absent               |

---

## Kill criteria

Stated up front, so they are decisions rather than sunk-cost arguments later.

- **`/build`** — deleted if the feedback logs hold no entries with
  `command: "build"` across three consecutive tickets, or if none of them ever
  routes to another skill's context file. `PLAN.md` becomes the interface.
- **`/research`** — deleted if no prompt it emits is ever acted on, run or sent
  upstream. A channel nobody uses is a skill description charged to every session
  for nothing.
- **Any `process/*.md`** — housekeeping, not a rule. A file still empty after a
  dozen or so tickets is evidence that command does not need one; delete it then.
  It costs nothing while empty, so there is no deadline.
- **The whole system** — if no context-file line has ever changed a later run's
  behaviour in a way you can point at, the loop is decorative. That is the honest
  test, and it is worth applying early enough to walk away from.
