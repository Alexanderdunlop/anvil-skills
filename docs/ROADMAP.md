# Roadmap

Eight milestones, M0 to M7. The ordering is not a build order chosen for
convenience — each milestone exists to retire one specific risk, and the risks
are ordered so that an early wrong answer is cheap to fix.

The single organising decision: **`/setup` is built last.** `/setup` is a
generator for the file format, and generators written against a moving format
get rewritten. The format has to stop moving first, which means it has to be
used in anger first.

---

## Progress

| M     | State       | What it produced                                                    |
| ----- | ----------- | ------------------------------------------------------------------- |
| M0    | done        | contract, entry format, 8 hand-written entries, manifests validate   |
| M1    | done        | `/scope`, and eight written checks that all pass on the installed plugin |
| M2    | in progress | `/feedback` written, and thirteen written checks it has yet to pass   |
| M3    | in progress | six tickets scoping the remaining commands, ids 0001–0006            |
| M4–M7 | not started | —                                                                    |

**The ticket numbering above was wrong and is corrected here.** This list ran
0002–0007 on the assumption that an earlier milestone had minted 0001. None had
— M1's live runs happened in another repo, so `.anvil/tickets/` was empty and
`max(existing) + 1` is 0001. Numbering from 0002 would have meant skipping an id
to protect a plan, which is the plan overriding the contract.

**M2 is written, not verified.** The skill and its checks exist; every check that
needs a live run against the installed plugin is still outstanding, and the
milestone is not done until they pass. M2 has already resolved three questions
the contract had left open — `ticket: null` counting (`fb-0008`),
`command: "manual"` (`fb-0007`), and how "tickets since" is measured for trigger
3 — all in `FILE_CONTRACT.md`, all forced by having to actually count something.

M1 changed the contract twice, which is the milestone working as intended:
`/scope` may now read one prior ticket the user names (§3), and the `fingerprint`
field grew a warning about prose keys (§4.6). Both came from running the command,
not from reading it — see `evals/README.md` for the checks that caught them.

---

## Reading the file lists

Every milestone below lists files in two trees, always labelled. Confusing them
is the main structural hazard in this project — see `FILE_CONTRACT.md` §0.

- **Tree A** — this repo, `anvil-skills/`. Hand-authored, versioned, shipped.
- **Tree B** — `${CLAUDE_PROJECT_DIR}/.anvil/` in whichever repo anvil is
  pointed at. Generated and maintained by the skills.

Where a milestone touches Tree B, it means *this repo's own* `.anvil/` unless it
says otherwise — anvil is dogfooded on itself from M0. That directory ships
with the plugin; see `FILE_CONTRACT.md` §8 for why, and for the two conditions
that make it safe.

All eight are `skills/<name>/SKILL.md` with `disable-model-invocation: true`,
not `commands/`. Every `.anvil/` path inside them is written
`${CLAUDE_PROJECT_DIR}/.anvil/...`.

**Skills are discovered, not declared.** A `skills/<name>/SKILL.md` loads because
the directory exists. `plugin.json` names no component paths, so adding a skill
changes no manifest — which is why no milestone below lists `plugin.json` as
changed for adding one. Listing skills explicitly would *replace* the default
directory scan rather than supplement it, buying nothing but a manifest that lags
the tree by one edit, and a hard validation error for every path not yet created.

---

## Dogfooding targets

Three repos, each retiring a different risk. They are not three attempts at the
same test.

**M0–M3 — `anvil-skills` itself.** These milestones exercise `/scope` and
`/feedback`, which need real corrections and nothing else. A markdown repo is
sufficient for that and convenient for it.

**M4–M6 — `site`.** `anvil-skills` stops being adequate at M4. It has no test
runner, no build, and no smoke path, so its `CONFIG.md` is near-bare and its
`CRITICAL_PATHS.md` is near-empty. `/verify`'s entire definition of done
is per-case verdicts with command output as evidence, and there are no commands
to run. `site` has `vitest`, `eslint`, `next build` and a browser smoke path, so
every definition of done in M4–M6 is satisfiable and iteration is fast.

**M7 — `dodgeball` for fresh generation, `site` for the update path.** These are
different halves of `/setup`'s definition of done and neither repo can supply
both.

`dodgeball` is the fresh-generation test, and the point is not extra coverage —
it is that M7's risk is *a generator written against a format that only ever saw
one stack*, and `site` cannot retire that risk because `site` is the stack the
format was shaped around. `dodgeball` is alien in exactly the useful ways: an
unusual test runner, no npm, an existing `CLAUDE.md`, and critical paths that
resist being written as executable checks. If `CRITICAL_PATHS.md` cannot express "the throw arc feels right", that is a
finding about the contract. Far better to find it against your own game than
after someone else installs anvil.

`site` is the update-path test, because by M7 it already has a working `.anvil/`
built up over three milestones. That makes it the only honest test of
idempotence: re-running `/setup` must be an update that leaves human-written
lines alone, not an overwrite.

**Do not prepare `dodgeball`.** Nothing gets synced between these repos.
`dodgeball` receives a `.anvil/` generated by `/setup` at M7 and not before. If
you find yourself hand-editing `dodgeball`'s `.anvil/` to resemble `site`'s, you
have started designing for it, and it has stopped being the unfamiliar case.

---

## Shape of the system

```
/scope ──gate──► /kickoff ──gate──► /build ──► /verify ──► /review
   │                 │                 │           │           │
   └─────────────────┴─────────────────┴───────────┴───────────┘
        appends to .anvil/feedback/{human,self}.jsonl
                                      │
                                 /feedback
                                      │
                     distils into .anvil/ context files
                                      │
                  read by the commands that need them, next run
```

**One writer, one reader.** Every command *writes* feedback entries. Only
`/feedback` *edits* context files. `/review` is not exempt: it produces
judgment about scoping and sizing, and that judgment lands in the logs like
everything else. Two writers to the same context file is how the file becomes
a landfill.

`/research` sits off this diagram deliberately. It reads the same logs and the
same `unrouted.md`, and writes nothing at all — see `FILE_CONTRACT.md` §4.11.
Adding an eighth skill that reads the evidence base costs the one-writer
invariant nothing, because it is not a writer.

---

## `/research` — cut, then restored

Recorded rather than quietly reversed, because the distinction that brings it
back is the entire justification for the skill.

**Why it was cut.** It was specified as "generates a prompt to improve the
process itself", which is `/feedback`'s output under a different name. That
reasoning was sound and it applied to a different feature that happened to share
the name.

**Why it is back.** What `/research` actually is:

> `/feedback` improves **this** repo's files from **this** repo's evidence.
> `/research` turns unresolved patterns into a question for **outside**
> investigation, and is the channel by which anvil learns from repos its author
> will never see.

That is not `/feedback` under a different name. It is the only path by which
evidence from someone else's repo reaches this one's routing table, and it costs
a GitHub issue template to build — the prompt it emits is either run by the user
on Claude research or parallel.ai, or opened as an issue here. No submission
endpoint, no telemetry, no infrastructure.

**Where it lands.** Scoped at **M3**, which is the first point `unrouted.md`
holds real content and therefore the first point the skill can be designed
against evidence rather than imagination. Built at **M7** with packaging,
because a contribution channel only matters once people can install the thing.
The kill criterion is unchanged in spirit and stated at the end of this file.

Final v1 command set, **eight**: `/setup`, `/scope`, `/kickoff`, `/build`,
`/verify`, `/review`, `/feedback`, `/research`.

---

## M0 — Hand-author the contract

**Goal.** Freeze the `.anvil/` file contract and the feedback entry format by
writing them by hand, from real corrections, with no automation whatsoever. The
schema is designed against evidence or it is designed against imagination.

**Files created.**

```
Tree A — anvil-skills/
  docs/ROADMAP.md
  docs/FILE_CONTRACT.md
  docs/PHILOSOPHY.md            rationale + link to the CLAUDE.md article
  README.md                     incl. the "we ship our own .anvil/" note
  .claude-plugin/plugin.json
  .claude-plugin/marketplace.json

Tree B — anvil-skills/.anvil/  (this repo, dogfooded)
  CONFIG.md                     hand-written for THIS repo
  CRITICAL_PATHS.md
  REVIEW_RULES.md
  process/scope.md              title line only — no seeds
                                (the only one; see below)
  feedback/human.jsonl          ≥ 4 real entries, hand-written
  feedback/self.jsonl           ≥ 1 real entry, hand-written
  feedback/unrouted.md
```

No `skills/`, no `commands/`, no `hooks/`.

**Only `process/scope.md`, deliberately.** The contract has `/setup` create all
five `process/*.md`, but there is no `/setup` at M0 and `/scope` is the only
command that exists. The other four are created by the milestone that builds
their command — `kickoff.md` and `build.md` at M4, `verify.md` at M5,
`review.md` at M6 — each already listed there. `/setup` at M7 is what first
creates all five at once, and M7's cold-install check is where that is verified.
Written down so M1 does not inherit four missing files nobody planned for.

**Definition of done.**

- Five feedback entries exist, drawn from corrections actually given in past
  sessions — not invented to fit the schema.
- `claude plugin validate .` passes on the manifests, before any skill exists.
  **This is a well-formedness check and nothing more.** Skills are discovered
  from `skills/<name>/SKILL.md` rather than declared in the manifest, so nothing
  at M0 verifies that a skill will actually load. Wiring is first checked at M1,
  by the local-marketplace install M1 already needs for the path rule — so the
  weaker M0 gate costs nothing, it just must not be read as more than it is.
- Each of the five is hand-routed to a destination file without adding a field
  to the entry format. If a field has to be added, the format was wrong and M0
  is not done.
- At least one of the five routes to `unrouted`. A routing table that catches
  everything on the first attempt is a table that has not met reality.
- At least one is a self-observed stall in `self.jsonl`, so both logs are
  exercised and the shared `fb-NNNN` sequence spans them.
- Every context file is under budget, measured with `wc -l` against the shipped
  defaults. No `BUDGETS.md` — this repo runs on defaults, and M2 needs an
  unmodified `REVIEW_RULES.md` to test its 30-line ceiling against.
- Cold-start check: a fresh session given only `.anvil/` can state what the repo
  is, where things live, and what must not break.

**Risk retired.** Designing an entry schema around imagined corrections, then
discovering at M2 that it cannot express a real one. Also the cheapest possible
place to discover that a category is missing.

---

## M1 — `/scope`

**Goal.** One command, end to end, proving the two hardest structural
properties: a cold start from files alone, and an approval gate that holds.

**Files created or changed.**

```
Tree A — anvil-skills/
  skills/scope/SKILL.md         new — disable-model-invocation: true
  evals/scope-gate/case.yaml    new — the gate-holds case
  docs/FILE_CONTRACT.md         changed if the format bends under use

Tree B — .anvil/
  process/scope.md              changed — real lines, from real runs
  feedback/human.jsonl          changed — appended during runs
```

**Definition of done.**

- Run in a session with no prior history. `/scope` reads `CONFIG.md` and
  `process/scope.md` and needs nothing else.
- **Path rule.** Every `.anvil/` reference in `SKILL.md` is
  `${CLAUDE_PROJECT_DIR}/.anvil/...`. Verified by installing the plugin from a
  local marketplace and running `/scope` in a *different* repo — it must touch
  that repo's `.anvil/`, never the copy in `~/.claude/plugins/cache/`. This is
  the test that matters most in M1 and it cannot be done by reading the file.
- **Always-on cost.** `claude plugin details anvil-skills` shows zero always-on
  tokens beyond the skill description. Establishes the baseline the remaining
  seven skills are measured against.
- Produces `.anvil/tickets/NNNN-slug/TICKET.md` with numbered, observable
  acceptance criteria, inside budget.
- **The gate holds.** It stops and asks for approval, and does not write
  `CONTEXT.md`, `PLAN.md` or anything else in the same run — verified by
  deliberately replying with something ambiguous and confirming it does not
  proceed. Then captured as an eval case so `claude plugin eval` re-checks it on
  every later change, rather than relying on remembering to test it.

  `claude plugin eval` exists and is substantial — it takes
  `evals/**/case.yaml`, runs a no-plugin ablation arm, supports LLM graders and
  a `scaffold_script`, which is what makes the cross-repo path test above
  automatable.

  **Resolved at M1: it is in the CLI, and it is early access.** An earlier draft
  of this line said it was absent from the published reference; that was wrong
  and is corrected here rather than deleted. What actually blocks it is
  entitlement — `claude plugin eval init` refuses with "currently in early
  access" on this account, so the case schema cannot be run and therefore cannot
  be checked. Writing a `case.yaml` against a schema nothing here can execute is
  a spec designed against imagination, which is the failure M0 exists to
  prevent. The fallback this milestone allows was taken: eight written checks in
  `evals/README.md`. The definition of done is "the gate holds, and the check is
  repeatable" — `eval` is the preferred mechanism, not the requirement. Convert
  checks 3 and 4 first when access opens.
- Every correction given during the run is appended to `human.jsonl` *before the
  command exits*. `/clear` between commands means an unflushed entry is a lost
  entry.
- Run once in a foreign repo of a different language to confirm nothing
  repo-specific leaked into the command.

**Risk retired.** Whether an approval gate can be made to hold against a model
that wants to be helpful, and whether a command can genuinely start cold. If
either fails, every later command inherits the failure, so it is found here for
the price of one command.

---

## M2 — `/feedback`

**Goal.** Close the loop. Raw entries in, distilled context files out, with the
admission test and the budgets enforced mechanically rather than by good
intentions.

**Files created or changed.**

```
Tree A — anvil-skills/
  skills/feedback/SKILL.md      new — admission test inlined verbatim

Tree B — .anvil/
  feedback/unrouted.md          changed — first real parked entries
  process/scope.md              changed — from M1's real corrections
  BUDGETS.md                    created only for the override and trigger 3
                                tests below, then removed
```

M2 carries two things, not one: the compounding loop, and budget resolution.
`/feedback` is the only skill that reads `BUDGETS.md` or enforces a ceiling, so
both land here or nowhere.

**Definition of done — the loop.**

- Reads **both** logs, applies the three admission questions to every `open`
  entry, and records which question killed each rejected one.
- Respects thresholds: nothing enters a `process/*.md` below two sightings, and
  `claude-md` at three sightings produces a *proposal* the human accepts or
  rejects — `/feedback` writes no `CLAUDE.md`.
- Counts a fingerprint across **both** logs. One sighting in `human.jsonl` and
  one in `self.jsonl` is two, and promotes. Tested with a real pair, because
  this is the rule the two-file split exists to protect.
- Enforces one-in-one-out: at the limit it names the evicted line, justifies
  the swap, and appends the evicted text back to `self.jsonl`. That is trigger 1
  of the staleness pass (`FILE_CONTRACT.md` §6); the other two land here too.
- **Staleness pass, trigger 2 — duplication.** Put a line in `process/scope.md`
  and the same constraint in this repo's `CLAUDE.md` — create one if absent;
  §0's note that it does not load for *users* does not affect this test, which
  is about `/feedback` reading a file, not about context loading. `/feedback` removes the
  anvil-side line automatically and appends the removed text to `self.jsonl`
  naming the trigger. It leaves `CLAUDE.md` untouched — verified by diffing it
  to nothing. Runs on a file that is nowhere near its ceiling, since the whole
  point is that this trigger is not budget-driven.
- **Staleness pass, trigger 3 — obsolescence — presents and never removes.**
  Nothing in this repo is 20 tickets old at M2, so reach the condition by
  lowering the threshold rather than by ageing anything: set `stale_after` to a
  number this repo's history can actually satisfy in the same temporary
  `BUDGETS.md` the override test uses. That exercises the new key and the
  trigger in one run. Confirm `/feedback` names the line, states how long the
  fingerprint has been quiet, and **leaves the file unchanged**. A trigger 3
  that removes anything on its own is the failure to catch, and it is cheaper to
  catch now than after twenty real tickets.

  If M2's history is too short even for `stale_after: 1`, fall back to forging —
  hand-edit a log entry's `ts` and `ticket` to fake the age — and **delete the
  forged entry the moment the test passes.** M0's definition of done says the
  entries are corrections actually given, and `FILE_CONTRACT.md` §8 ships this
  repo's logs to users as the honest evidence for anvil's central claim. A
  forged line left in place makes both of those false, and it would be
  indistinguishable from a real one to everybody who reads it afterwards.
  Prefer the threshold; it leaves nothing behind to clean up.
- **Age is read from the logs.** No context file gains a date stamp, a
  promotion marker, or a citation count. Checkable by eye and by `wc -l`: the
  line counts after a staleness pass move down or stay level, never up.
- **Idempotence test.** Two consecutive runs — the second changes nothing and
  promotes nothing, including removing nothing a staleness pass already removed.
- Prints the `unrouted.md` count on exit.

**Definition of done — budget resolution.** Three tests, because the mechanism,
the number, and who may change the number are three separate things.

1. **Hard-fail on the shipped default.** Fill `REVIEW_RULES.md` to its 30-line
   default with genuinely load-bearing checks and **no override set**, then feed
   it a strong new lesson. It must fail and report the conflict, not ship 31
   lines and not silently drop the lesson. The point is testing the default, so
   this test must not create a `BUDGETS.md`.
2. **Override resolution.** Add a `BUDGETS.md` setting `review_rules: 45`.
   `/feedback` must now enforce 45, not 30 — the same file that failed in test 1
   accepts lines up to the new ceiling and fails at it. The same file carries
   the lowered `stale_after` the trigger 3 test needs, which is the one key here
   that is not a line count. Remove the file afterwards; the repo's real state
   is defaults.
3. **No self-raise.** On the hard failure from test 1, `/feedback` presents
   *both* ways out — evict a specific named line, or raise the budget in
   `BUDGETS.md` — and does neither. Checked as an **absence assertion**, since
   test 1 has no `BUDGETS.md` to write-protect: run test 1, confirm it fails and
   reports both options, then confirm no `BUDGETS.md` exists afterward. Belt and
   braces, write-protect `.anvil/` itself and confirm the run still completes its
   report. This is the rule most likely to be quietly violated, because raising
   the number is always the locally helpful move.

**Risk retired.** Three risks. That the compounding loop is a nice diagram
rather than a working mechanism — everything after M2 depends on the loop being
real, which is why it is built second rather than last. That budgets stated in a
document are budgets in practice: that the ceiling binds, that a project can
move it, and that the tool cannot move it for itself. And that lines only ever
enter context files — a loop with promotion and no removal is an accumulator,
and it is cheapest to find that out before six milestones of promotions.

---

## M3 — Dogfood

**Goal.** Use `/scope` and `/feedback` to specify the six remaining commands.
The remaining commands are not hand-written; they are scoped by the tool.

**Files created or changed.**

```
Tree A — anvil-skills/
  docs/FILE_CONTRACT.md                     changed — from log evidence

Tree B — .anvil/
  tickets/0001-kickoff-skill/TICKET.md      via /scope
  tickets/0002-build-skill/TICKET.md        via /scope
  tickets/0003-verify-skill/TICKET.md       via /scope
  tickets/0004-review-skill/TICKET.md       via /scope
  tickets/0005-setup-skill/TICKET.md        via /scope
  tickets/0006-research-skill/TICKET.md     via /scope — built at M7
  process/scope.md                          changed via /feedback
  REVIEW_RULES.md                           changed via /feedback
  feedback/unrouted.md                      changed via /feedback
```

**Definition of done.**

- Six tickets, all produced by `/scope`, none hand-authored. A ticket rewritten
  by hand afterwards is a `/scope` bug and gets logged as one.
- At least one change to `FILE_CONTRACT.md` traceable to a specific feedback-log
  entry id. If five real runs produce no contract change, the contract was
  either exceptionally good or not actually exercised — assume the latter and
  check.
- `unrouted.md` is non-empty and has been read. Decide, explicitly, whether it
  justifies a new category or whether the entries are noise.
- **`/research` scoped, not built.** `0007-research-skill` is scoped here
  because M3 is the first point `unrouted.md` holds the evidence the skill is
  designed against — the real parked entries name what an outside question would
  have to be about. The ticket then waits for M7. Scoping it against an empty
  pile would be the mistake that got it cut the first time.

**The unrouted pile, read at M3. Decision: no new category.**

Three entries. `fb-0007` and `fb-0008` were questions about the contract and both
were answered at M2 in §4.6 — they were settled business still sitting in the
pile, and noticing that is what produced the §4.7 change below. That leaves
`fb-0003`, which argues for a `process:setup` key.

Rejected, for now. Its lesson — seed only from what the user states — is already
in §4's seeding rule, where `/setup`'s author reads it, so the entry is evidence
that was absorbed rather than evidence of a missing route. And `/setup` runs
about twice in a repo's life: a sixth `process/*.md` would be created, read and
reasoned about by everyone in order to hold lessons for the command that runs
least. Revisit after M7, when `/setup` has actually run and can produce evidence
instead of an argument.

**The contract change, and where it did not come from.** §4.7 gained a way to
close a parked entry, traceable to `fb-0007` and `fb-0008`: nothing in the
contract ever took a bullet out of `unrouted.md`, so the count printed on exit —
the one signal that the category table is wrong — could only ever go up.

It is worth being precise that this came from reading the pile, **not** from the
six scoping runs. Those six produced no contract change at all, and the
definition of done above says to assume the contract was not actually exercised
rather than that it was exceptionally good. That is the right reading here: the
six ran in one session against a context already saturated with this document,
which is close to the opposite of the cold start the format is meant to survive.
Treat the ticket format as unexercised until a run that has not just read the
spec produces one.

**Risk retired.** That the process works on toy input and collapses on real
input, and that commands specced by hand quietly diverge from what `/scope`
would actually produce. This is the milestone that catches a broken workflow
before six more commands are built on top of it.

Partly retired here. Six tickets exist and none was hand-written after the fact,
but they were produced in one saturated session, so the divergence half of this
risk is still open.

---

## M4 — `/kickoff` and `/build`

**Subject repo: `site`.** `anvil-skills` is no longer adequate from here — see
Dogfooding targets.

**Goal.** The plan-then-execute pair. Built together because `/build` exists
only to consume `/kickoff`'s output, and the interesting question is whether the
plan files are actually enough to work from.

**Files created or changed.**

```
Tree A — anvil-skills/
  skills/kickoff/SKILL.md       new
  skills/build/SKILL.md         new

Tree B — .anvil/
  process/kickoff.md            new
  process/build.md              new
  tickets/*/CONTEXT.md          new, per ticket
  tickets/*/PLAN.md             new, per ticket
  tickets/*/TEST_CASES.md       new, per ticket
  feedback/self.jsonl           changed — build stalls
```

`/build` writes no file of its own. Its stalls go into `self.jsonl` like every
other self-observed entry.

**Definition of done.**

- `/kickoff` produces all three plan files inside budget, and stops at its
  approval gate without starting implementation.
- `/build` starts cold from the ticket body + `CONTEXT.md` + `PLAN.md` +
  `TEST_CASES.md` alone, with no conversation history, on at least three real
  tickets.
- `/build` appends a `self.jsonl` entry whenever it stalls, guesses, or has to
  ask — with `correction: null`, a `fingerprint`, and a `proposed_line` naming
  what the plan should have said. The entry says what unblocked it, not just
  that it was blocked.
- **`/build`'s survival test.** Across three tickets, do entries with
  `command: "build"` exist, and do any of them carry a `category` of
  `process:kickoff`? Build entries that route back into `/kickoff` are the whole
  argument for the skill. If the logs hold no build entries after three tickets,
  `/build` is a wrapper around "read the plan" and gets deleted, with `PLAN.md`
  becoming the interface. Decide at the end of M4, not later.
- At least one build-sourced `fingerprint` reaches two sightings and is promoted
  by `/feedback` into `process/kickoff.md` — proving a stall can complete the
  round trip, which is what a separate stuck-file could never do.

**Risk retired.** That plan files are either too thin to build from or too fat
to be worth reading — the budgets in the contract are guesses until three real
tickets test them. And the risk that `/build` is a command with no reason to
exist, which is cheaper to discover before `/verify` and `/review` are wired to
it.

---

## M5 — `/verify`

**Subject repo: `site`.** The degradation test below is only meaningful in a
repo where smoke tests would otherwise work, which rules out `anvil-skills`.

**Goal.** The machine-checkable half of quality: self-review against rules,
critical paths, test cases, and smoke tests. Nothing subjective — every output
is a pass, a fail, or a "could not check".

**Files created or changed.**

```
Tree A — anvil-skills/
  skills/verify/SKILL.md        new
  docs/FILE_CONTRACT.md         changed — smoke/MCP degradation resolved

Tree B — .anvil/
  process/verify.md             new
  CRITICAL_PATHS.md             changed — format finalised against real use
  REVIEW_RULES.md               changed — first real use since M0
  tickets/*/VERIFY.md           new, per ticket
```

`/verify` is `REVIEW_RULES.md`'s first real consumer. It has existed since M0
and been budget-tested since M2, but until now nothing has read it to actually
review code, so M5 is where its 30-line default meets a real diff.

**Definition of done.**

- Emits a per-case verdict for every line in `TEST_CASES.md` and every line in
  `CRITICAL_PATHS.md`, with evidence — a command and its output, not an
  assertion.
- Applies every line in `REVIEW_RULES.md` to the diff, and keeps that separate
  from `process/verify.md`, which governs how the command runs. Lines landing in
  the wrong one of those two files is the failure mode to watch for.
- **Degradation test.** Runs in a session with no MCP server available and with
  `smoke: none` in `CONFIG.md`. It must produce a manual checklist and mark those
  cases "could not check", never fail the run and never claim a pass it did not
  observe. Portability to any repo depends on this.
- Writes feedback entries under `path` and `process:verify`, and edits no
  context file directly.
- Produces **no** judgment output: no opinions on scope, size, or whether the
  work was worth doing. Those belong to M6.

**Risk retired.** That an MCP dependency silently breaks the "works in any repo,
any language" constraint, and that `/verify` drifts into subjective territory
and starts duplicating `/review`.

---

## M6 — `/review`

**Subject repo: `site`.** Judging whether a slice held needs real PRs of real
size.

**Goal.** The judgment half: was this scoped right, is the PR too big, did the
slice hold. Output is feedback entries aimed at improving `/scope` and
`/kickoff` — never a direct context-file edit.

**Files created or changed.**

```
Tree A — anvil-skills/
  skills/review/SKILL.md        new

Tree B — .anvil/
  process/review.md             new
  process/scope.md              changed — via /feedback, from review entries
  process/kickoff.md            changed — via /feedback, from review entries
```

**Definition of done.**

- `/review` writes only to the feedback logs. Confirmed by running it with the context
  files write-protected: it must complete successfully.
- **Boundary test.** Run `/verify` and `/review` on the same finished ticket and
  compare outputs. Overlapping findings mean the split is wrong and one of the
  two skill definitions gets tightened. Repeat until they are disjoint. Frozen
  as an eval case once disjoint, so the boundary cannot silently re-blur.
- At least one `process:scope` or `process:kickoff` line in a context file
  originated from a `/review` entry — proving the loop reaches backwards into
  earlier commands, which is the whole reason `/review` exists.

**Risk retired.** The `/verify` vs `/review` overlap, tested rather than
asserted. And the "two writers" failure, where a second command starts editing
context files and the one-writer invariant quietly dies.

---

## M7 — `/setup`, `/research` and packaging

**Subject repos: `dodgeball` for fresh generation, `site` for the update path.**

**Goal.** Generate `.anvil/` for a repo that has never seen anvil, and update it
safely for one that has. Built last, deliberately: by now the format has
survived six milestones of real use and has stopped moving.

`/research` is built here for a different reason. It is not blocked on the
format — it writes nothing — but it is a contribution channel, and a
contribution channel is worthless until people can install the thing and have
something to contribute from. It ships with the package or it ships to nobody.

**If anvil is published before M7**, a crude `/setup` ships anyway — one that
only asks the `CONFIG.md`, `CRITICAL_PATHS.md` and `REVIEW_RULES.md` questions
and writes the answers verbatim, with no detection. Seven skills that all refuse to run without `.anvil/`
is not a release. M7 then upgrades that placeholder rather than introducing the
skill, and the format-freeze check below applies to the upgrade.

**Files created or changed.**

```
Tree A — anvil-skills/
  skills/setup/SKILL.md         new
  skills/research/SKILL.md      new — from M3's ticket 0007
  .github/ISSUE_TEMPLATE/       new — the research-prompt issue template
  hooks/                        new, only if a real need survived M0–M6
  .claude-plugin/marketplace.json  changed — finalised
  .claude-plugin/plugin.json    changed — version decision only, see below
  README.md                     changed — install, first run, .anvil/ note
  docs/PHILOSOPHY.md            changed — final

Tree B — created by /setup in a foreign repo
  the full .anvil/ tree, from nothing
```

`/setup`'s stack-detection half overlaps heavily with the existing skill that
scans a repo and generates a `CLAUDE.md` from targeted questions. Pull that in
rather than writing it twice; the new work is the question set for `CONFIG.md`,
`CRITICAL_PATHS.md` and `REVIEW_RULES.md`, not the scanning.

**Definition of done.**

- **Fresh generation — `dodgeball`.** `/setup` produces a complete `.anvil/`
  from nothing, in a repo with no npm, an unusual test runner and an existing
  `CLAUDE.md`. Every file passes the contract and sits inside budget.
- **`CLAUDE.md` confirmation, not decision.** The overlap question was settled
  in `FILE_CONTRACT.md` §7: `CLAUDE.md` wins and anvil does not duplicate it,
  which is why `CONFIG.md` holds only anvil's own config and pointers outside
  the repo. There is no `claude-md:` key. `dodgeball` confirms that ruling —
  its existing `CLAUDE.md` should leave `CONFIG.md` unchanged. If it does not,
  the finding is a contract bug, not a `/setup` question to reopen.
- **Unstateable paths recorded.** Any critical path in `dodgeball` that cannot
  be written as an executable or observable check is logged as a feedback entry
  against the contract, not quietly omitted. This is expected, and it is the
  main thing `dodgeball` is there to surface.
- **Idempotence and update — `site`.** Re-running `/setup` on `site`, which has
  carried a working `.anvil/` since M4, is an update rather than an overwrite.
  Human-written lines in `REVIEW_RULES.md` and `CRITICAL_PATHS.md` survive, and
  lines promoted by `/feedback` over three milestones survive. Verified by re-running
  and diffing to nothing.
- **Cold install.** In a repo with no `.anvil/`, every one of the other seven
  skills stops and points at `/setup` — `/research` included, since its inputs
  live under `.anvil/` too. None of them creates a file, guesses a
  `CONFIG.md`, or proceeds on defaults.
- **`process/*.md` ship title-only.** All five contain their title line and
  nothing else. `/setup` seeds from what the user tells it, never from what
  anvil assumes, and there is no universal `process` lesson a user can be asked
  for. Checkable with `wc -l` — one line each.
- **`/setup` does NOT create `BUDGETS.md`.** Absent means defaults, and a
  helpful `/setup` writing one pre-filled with the shipped numbers would freeze
  those numbers into every repo and turn a default into a decision nobody made.
  Verified by confirming the file is absent after a fresh run.
- Asks rather than guesses. Anything it cannot detect with confidence becomes a
  question, and anything the user does not know is omitted from `CONFIG.md`
  rather than filled with a plausible default.
- **Format-freeze check.** No change to `FILE_CONTRACT.md` during M4–M6 that
  `/setup` does not implement. Walk the contract file by file and confirm.
- **`/research` writes nothing.** Run it with the whole of `.anvil/`
  write-protected and confirm it completes and emits its prompt. It reads
  `unrouted.md` and both logs, and produces a research question quoting the
  `fb-NNNN` ids behind it — no context file, no ticket, no log entry. Same shape
  of check as M6's for `/review`, and for the same invariant.
- **The upstream path works end to end.** Take the prompt `/research` emits and
  open an issue on this repo from the template, by hand. If the template needs a
  field the prompt does not produce, that is a finding about `/research`'s output
  format, not about the template.
- Plugin installs cleanly from `marketplace.json` into a clean Claude Code
  install, and all eight skills are **discovered** from `skills/` and appear as
  `/name`. This install is the only check that skill wiring works — the manifest
  declares no skills, so `claude plugin validate` never could be.
- **Version decision.** `version` is unset from M0 through M6, so the git SHA is
  the version and users move on every commit. That is right while the format is
  still moving, and it stops being right the moment people install it and would
  rather move when you say so. M7 decides whether to pin a semver.

  One cost either way, recorded so the decision is made with it in view:
  `claude plugin validate . --strict` **cannot pass while `version` is unset** —
  the missing-version warning is its only diagnostic, and `--strict` treats
  warnings as errors. So strict validation is unavailable as a CI gate until
  this is settled. `claude plugin tag` also wants a `plugin.json` version that
  agrees with the enclosing marketplace entry, which is a second reason the
  question lands here rather than earlier.
- **Final cost gate.** `claude plugin details anvil-skills` still shows zero
  always-on tokens beyond the eight skill descriptions, with all eight shipped.
  If it has crept up, guidance leaked out of a skill body into something
  always-loaded.
- Any guidance anvil ships lives inside a skill. A `CLAUDE.md` at this repo's
  root does not load as project context for users, so it must not be relied on.
- `claude plugin eval` passes, including the M1 gate case and the M6 boundary
  case.

**Risk retired.** Writing a generator against a format that is still moving —
the single most expensive rework in the project, avoided by ordering rather than
by care.

---

## Risk ledger

| M | Risk retired |
|---|---|
| M0 | Schema designed against imagined corrections rather than real ones |
| M1 | Approval gates that do not hold; skills that cannot start cold; bare `.anvil/` paths reading the plugin cache instead of the user's repo |
| M2 | A compounding loop that is a diagram rather than a mechanism; budgets that are advisory in practice; a ceiling the tool can raise for itself; context files that only ever grow |
| M3 | A process that works on toy input and collapses on real input |
| M4 | Plan files too thin to build from or too fat to read; `/build` existing without a reason; a contract validated only against a repo with nothing to build |
| M5 | MCP dependency breaking any-repo portability; `/verify` drifting subjective |
| M6 | `/verify`–`/review` overlap; a second writer to context files |
| M7 | A generator written against a moving format, and one that only ever saw a single stack; skills that do something unhelpful when `.anvil/` is absent |

## Kill criteria

Stated up front, so they are decisions rather than sunk-cost arguments later.

- **`/build`** — deleted if the feedback logs hold no entries with
  `command: "build"` across three consecutive tickets, or if none of them ever
  routes to another skill's context file (decided end of M4).
- **`/research`** — restored to v1, but the original criterion still binds and
  is now a gate rather than a revival test: it is scoped at M3 only if
  `unrouted.md` shows a real pattern `/feedback` cannot route. An empty or
  noise-only pile at end of M3 means the ticket is not written and the command
  set returns to seven. If it is scoped and built, it is deleted if no prompt it
  emits is ever acted on — run or sent upstream — because a channel nobody uses
  is a skill description charged to every session for nothing (decided end of
  M3, reviewed at M7).
- **Any `process/*.md`** — housekeeping, not a rule. A file still empty after a
  dozen or so tickets is evidence that command does not need one; delete it
  then. It costs nothing while empty, so there is no deadline. Matches
  `FILE_CONTRACT.md` §4.5 — the contract is the authority on this, and the unit
  is tickets rather than milestones, since a user installing anvil has no
  milestones.
- **The whole system** — if by end of M3 no context-file line has changed a
  later run's behaviour in a way you can point at, the loop is decorative.
  That is the honest test, and it comes early enough to walk away from.
