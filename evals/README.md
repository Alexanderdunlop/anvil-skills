# Evals

## Status: manual, and why

`claude plugin eval` **exists** — it takes `evals/**/case.yaml`, runs a
no-plugin ablation arm, supports LLM graders and a `scaffold_script`, and
`--scaffold` is what would make the cross-repo path test below automatable.
ROADMAP M1 recorded it as absent from the published CLI reference; that was
wrong in a way worth correcting. It is in the CLI.

It is also **early access**, and not enabled on this account:

```
$ claude plugin eval init --bare scope-gate
`plugin eval` is currently in early access
```

So the case schema cannot be run, which means it cannot be checked. Writing a
`case.yaml` against a schema nothing here can execute would be a spec designed
against imagination — the exact failure M0 exists to prevent, one directory
over. The scripts below are the fallback M1's definition of done allows: the
gate holds, and the check is repeatable.

**When eval access opens**, `claude plugin eval init --bare scope-gate` scaffolds
against the real schema, and checks 3 and 4 below are the two worth converting
first.

---

## Manual re-test — `/scope`

Run all seven after any change to `skills/scope/SKILL.md`. Checks 1 and 2 are
static and take seconds. Checks 3 to 7 each need a fresh session, because a
command that only starts cold in theory does not start cold.

### 1. Path rule, static

```bash
grep -rn '\.anvil/' skills/ | grep -v CLAUDE_PROJECT_DIR
```

Must print nothing. Any hit is a review-blocking defect: a bare `.anvil/` path
resolves against the plugin cache copy of this repo rather than the user's.

### 2. Always-on cost

```bash
claude plugin details anvil-skills
```

Always-on must be zero beyond the skill description. Anything else means
guidance leaked out of the skill body into something loaded every session.
Record the number — it is the baseline the remaining seven skills are measured
against.

### 3. Path rule, live — the one that matters

Reading the file cannot prove this. Install from a local marketplace and run in
a **different** repo.

```bash
claude plugin marketplace add /path/to/anvil-skills
claude plugin install anvil-skills
cd /path/to/some-other-repo    # must have its own .anvil/
```

Run `/scope` there. Then:

```bash
git -C /path/to/some-other-repo status --short   # the ticket is here
diff -r ~/.claude/plugins/cache/anvil-skills/anvil-skills/*/.anvil .anvil
```

The ticket, and any feedback entry, must land in the other repo. The `diff` must
print nothing: the cached copy of this repo's own `.anvil/` is still byte-for-byte
what was installed. Any difference is the bug this whole rule exists for.

**Do not check this with `find -newermt`.** The installer copies the whole tree,
so every cached file carries an mtime from install time and a recency window
catches all of them whether or not anything wrote to them. The first run of this
check reported fourteen false hits that way. Compare content, not timestamps.

### Iterating between runs

The install is a **copy pinned to a commit**, not a symlink —
`~/.claude/plugins/cache/anvil-skills/anvil-skills/<sha>/`. Editing the working
tree changes nothing for the installed plugin, so every re-test needs:

```bash
git commit ...
claude plugin update anvil-skills@anvil-skills   # NOT the bare name
```

The bare `claude plugin update anvil-skills` fails with
`Plugin "anvil-skills" not found`. It wants `<plugin>@<marketplace>`, and both
are called `anvil-skills` here. Confirm the sha moved before spending a fresh
session on a stale copy:

```bash
ls ~/.claude/plugins/cache/anvil-skills/anvil-skills/   # newest sha == HEAD
```

### 4. The gate holds

Fresh session. Run `/scope` with a real idea. When the draft is presented, reply
with something **deliberately ambiguous** — "sure, I guess" or "looks good, but
what about mobile?".

Pass conditions, all of them:

- It does not write `TICKET.md`
- It does not write `CONTEXT.md`, `PLAN.md`, `TEST_CASES.md`, or any code
- It revises and presents again, or asks

```bash
git status --short   # must be clean
```

Then approve explicitly and confirm the ticket appears and the run stops there.

### 5. Cold start

Fresh session, no history. `/scope` must complete having read only
`CONFIG.md` and `process/scope.md`. If it opens `REVIEW_RULES.md`, the feedback
logs, or another ticket, it is over its 30-line budget.

### 6. The ticket

```bash
wc -l .anvil/tickets/<id>/TICKET.md          # ≤ 40
```

Four sections in order — Problem, Scope, Out of scope, Acceptance criteria.
Criteria numbered, and each one observable: something a person can watch happen,
never a file to edit. An empty Out of scope section usually means nothing was
cut.

### 7. Corrections flushed before exit

Correct something deliberately during the run — reject a slice, reword a
criterion. Then, **in the same session**, before running anything else:

```bash
tail -2 .anvil/feedback/human.jsonl
python3 -c "import json,sys; [json.loads(l) for l in open('.anvil/feedback/human.jsonl')]"
```

The correction must already be on disk, one object per line, `command: "scope"`,
`status: "open"`, and its `fb-NNNN` must not collide with anything in
`self.jsonl`. `/clear` runs between commands, so an entry written "later" is an
entry lost.

### 8. Foreign repo

Repeat check 4 in a repo of a different language. Anything repo-specific that
leaked into the skill shows up here and nowhere else.

---

## Manual re-test — `/feedback`

Run after any change to `skills/feedback/SKILL.md`. Checks 1 and 2 are static.
The rest each need a fresh session against the **installed** plugin — see
"Iterating between runs" above, because an edit in the working tree is invisible
to the thing you are testing.

**These checks mutate `.anvil/`, which is committed.** Every one of them ends
with a restore, and the milestone is not done until `git status --short` is clean
apart from the intended changes.

> **Never forge a log entry to reach a condition.** No hand-edited `ts`, no
> hand-edited `ticket`. `FILE_CONTRACT.md` §8 ships these logs as the honest
> evidence for anvil's central claim, and a forged line is indistinguishable
> from a real one to everyone who reads it afterwards. Every check below reaches
> its condition by lowering a threshold in a temporary `BUDGETS.md` instead.

### 1. Path rule and always-on cost, static

```bash
grep -rn '\.anvil/' skills/ | grep -v CLAUDE_PROJECT_DIR   # must print nothing
claude plugin details anvil-skills                          # always-on unchanged
```

Check 1 of the `/scope` list already covers every skill. The second command is
the one worth re-reading: adding a second skill must move the always-on number
by its description only. Anything else means guidance leaked out of the body.

### 2. It writes nothing it does not own

```bash
grep -n 'BUDGETS' skills/feedback/SKILL.md
```

Every hit must be a read or a prohibition. A skill that describes writing
`BUDGETS.md` or `CLAUDE.md` fails before it is ever run.

### 3. Cross-file counting — the rule the split exists for

Needs a real pair: the same `fingerprint` in `human.jsonl` and in `self.jsonl`,
across two distinct tickets — where the null-ticket bucket counts as one ticket
(`FILE_CONTRACT.md` §4.6). Do not manufacture the pair. It arrives from real
runs, and until it does this check is **pending**, not passed.

Run `/feedback`. The fingerprint must promote at two sightings, and **both**
entries must flip to `promoted`, in both files.

```bash
grep -c promoted .anvil/feedback/human.jsonl .anvil/feedback/self.jsonl
```

Counted per file it would be one and one and would sit below the threshold
forever. That is the failure this check exists to catch.

### 4. One in, one out

Fill the target file to its resolved ceiling, then feed it a stronger lesson.

Pass conditions, all of them:

- it names the evicted line and says why the new line beats it
- the evicted text lands in `self.jsonl` with `"command": "feedback"` and
  `"category": "unrouted"`
- `wc -l < <file>` is unchanged, not ceiling + 1

### 5. Hard fail on the shipped default

**No `BUDGETS.md` for this check** — the point is testing the default.

Fill `REVIEW_RULES.md` to 30 lines with genuinely load-bearing checks, then feed
it a strong new lesson.

```bash
wc -l < .anvil/REVIEW_RULES.md    # 30 before, 30 after
ls .anvil/BUDGETS.md              # must not exist, before or after
```

It must fail and report the conflict. Not 31 lines. Not a silently dropped
lesson.

### 6. Override resolution

Same file, same lesson as check 5. Now add:

```markdown
# Budgets

review_rules: 45
stale_after: 1
```

`/feedback` must enforce 45, not 30 — the file that failed in check 5 now accepts
lines, and fails again at 45. The lowered `stale_after` is what makes check 8
reachable in a repo this young, which is why both keys live in one temporary
file.

```bash
rm .anvil/BUDGETS.md    # the repo's real state is defaults
```

### 7. No self-raise

Run check 5 again. Then:

```bash
ls .anvil/BUDGETS.md    # must still not exist
```

It must present **both** ways out — evict a named line, or raise the budget in
`BUDGETS.md` — and do neither. The absence assertion is the test: check 5 has no
`BUDGETS.md` to write-protect, so "it did not write one" is the only observable.

Belt and braces, `chmod -R a-w .anvil/` and confirm the run still completes its
report rather than crashing. Restore with `chmod -R u+w .anvil/`.

This is the rule most likely to be quietly violated, because raising the number
is always the locally helpful move.

### 8. Trigger 2 — duplication removes, and only on the anvil side

Put a line in `process/scope.md` and the same constraint in this repo's
`CLAUDE.md`. Create `CLAUDE.md` if it is absent — §0's note that it does not load
as project context for users is about context loading, not about `/feedback`
reading a file.

Run on a file **nowhere near its ceiling**. This trigger is not budget-driven and
testing it under budget pressure tests the wrong thing.

```bash
git diff --stat CLAUDE.md          # must print nothing
tail -1 .anvil/feedback/self.jsonl # the removed text, naming trigger 2
wc -l .anvil/process/scope.md      # down by one
```

If the removed line's fingerprint has an open `claude-md` proposal, that proposal
must flip to `promoted` in the same pass.

### 9. Trigger 3 — presents, and removes nothing

With `stale_after: 1` from check 6 still in place, run `/feedback`.

It must name the line, say when it was promoted and how long the fingerprint has
been quiet, and stop.

```bash
git diff --stat .anvil/           # no context file changed by this trigger
```

**A trigger 3 that removes anything on its own is the failure to catch**, and it
is cheaper to catch now than after twenty real tickets have been promoted on top
of it.

### 10. Line counts never go up

```bash
wc -l .anvil/*.md .anvil/process/*.md
```

Before and after a staleness pass. Down or level. Never up.

### 11. Idempotence

Two consecutive runs, second in a fresh session.

```bash
git status --short    # must be clean after the second run
```

It promotes nothing, removes nothing, appends no bullet and no log entry —
including re-removing something a staleness pass already removed. Re-presenting a
trigger-3 line the user kept is allowed; changing a file is not.

### 12. The logs survive being written to

```bash
git diff .anvil/feedback/*.jsonl | grep '^[-+]' | grep -v status
python3 -c "import json;[json.loads(l) for f in ['human','self'] for l in open('.anvil/feedback/'+f+'.jsonl')]"
```

The only field that may differ on an existing line is `status`. Any other diff —
reordered fields, reflowed JSON, a changed `ts` — is a corrupted evidence base,
and it is corrupted in the file that is supposed to be the permanent record.

### 13. The count on exit

Every run ends by printing the `unrouted.md` count, including when it is zero.
Above ten, it says the category table needs review.

---

## Manual re-test — `/kickoff`

Needs a repo with something to plan against. `anvil-skills` has no test runner
and no build, so these run in `site` from M4 onward.

### 1. Precondition and gate

Fresh session, repo with no `.anvil/`: it must say to run `/setup` and write
nothing.

Then in a repo that has one, run against a real ticket and reply to the
presented trio with **"sure, I guess"**.

```bash
git status --short    # clean — no CONTEXT.md, no PLAN.md, no TEST_CASES.md
```

It revises and presents again, or asks. Then approve explicitly and confirm all
three appear and the run stops there — no code, no step 1.

### 2. Budgets

```bash
wc -l .anvil/tickets/<id>/{CONTEXT,PLAN,TEST_CASES}.md    # ≤ 40, ≤ 60, ≤ 40
```

A trio that does not fit is a ticket that needs slicing, not prose that needs
compressing.

### 3. Cold start

Fresh session, no history. It must finish having opened `CONFIG.md`,
`process/kickoff.md` and the ticket body. Opening `REVIEW_RULES.md`, the logs or
another ticket puts it over its 70-line budget.

### 4. The ticket body is resolved, not assumed

Set `tracker:` to an external value with no reachable integration. It must
either read a `MIRROR`-labelled `TICKET.md` and say the tracker wins, or stop and
ask. Opening `TICKET.md` without checking `tracker:` first is the defect.

### 5. Coverage

Every numbered acceptance criterion in the ticket appears in `TEST_CASES.md` as
at least one case, both sides observable. A criterion that produced no case must
have been called out as a ticket defect, not quietly dropped.

### 6. `CONTEXT.md` earns its lines

Read it against the files it names. Any line you could have got by opening the
file fails the admission test and should not be there. A bare list of paths with
no reasons is the common failure.

---

## Manual re-test — `/build`

### 1. Cold start from the files alone

Fresh session, no history, three plan files and the ticket body.

```bash
git status --short    # code changes only, nothing under .anvil/ but log lines
```

If it needed something only the planning conversation held, that is a hole in
`CONTEXT.md` — and check 3 says it must be in the log rather than worked around.

### 2. It rewrites no plan

```bash
git diff .anvil/tickets/<id>/PLAN.md    # must print nothing
```

The plan is the record of what was approved. Editing it makes the approval
meaningless, and "the plan was wrong" is a log entry and a conversation.

### 3. Stalls are logged, and logged usefully

```bash
tail -3 .anvil/feedback/self.jsonl
```

Each entry: `correction: null`, a kebab fingerprint, a `proposed_line` naming
what the plan should have said, and an `observed` that says **what unblocked
it** — not only that it was blocked.

### 4. The survival test — decide at the end of M4, not later

```bash
grep -c '"command":"build"' .anvil/feedback/*.jsonl
grep '"command":"build"' .anvil/feedback/*.jsonl | grep -c 'process:kickoff'
```

Across three real tickets. Build entries that route back into `/kickoff` are the
whole argument for the skill. If the logs hold no build entries after three
tickets, `/build` is a wrapper around "read the plan": delete it and let
`PLAN.md` be the interface.

### 5. The round trip closes

At least one build-sourced fingerprint reaches two sightings and is promoted by
`/feedback` into `process/kickoff.md`.

```bash
wc -l .anvil/process/kickoff.md
```

This is the check the whole system exists for — a stall observed by one command
changing the behaviour of a different one. A separate stuck-file could never do
it, which is why there is not one.

---

## Manual re-test — `/verify`

Runs in `site`. The degradation check is only meaningful in a repo where smoke
tests would otherwise work, which rules out this one.

### 1. Every line gets a verdict

```bash
wc -l < .anvil/tickets/<id>/TEST_CASES.md
grep -vc '^#\|^$' .anvil/CRITICAL_PATHS.md
grep -vc '^#\|^$' .anvil/REVIEW_RULES.md
```

The verdict count in `VERIFY.md` covers all three sets. A skipped case is
invisible in the output, which is what makes a partial run look like a clean one.

### 2. Evidence, not assertion

```bash
grep -n 'PASS' .anvil/tickets/<id>/VERIFY.md
```

Every `PASS` names a command and what it returned. If "works correctly", "was
tested" or "looks right" appears anywhere in the file, the check has failed —
those are the exact phrases a run reaches for when it did not actually look.

### 3. Degradation — the portability check

Fresh session with **no MCP server available** and `smoke: none` in `CONFIG.md`.

Pass conditions, all of them:

- the run completes and writes `VERIFY.md`
- smoke cases read `COULD NOT CHECK`, and each says which cause it was
- each of those lines carries the manual steps
- no case became a `PASS`
- it did not ask the user to install anything

Then repeat with `smoke:` naming a server that is genuinely unavailable. Same
result, different reason on the line.

### 4. It does not stop at the first failure

Break something a test case covers. Every remaining case must still get a
verdict, and the `FAIL` must name what the command returned.

### 5. Re-run overwrites

```bash
wc -l .anvil/tickets/<id>/VERIFY.md    # before and after a second run
```

Same case set, same line count. A file that grew is appending history that
belongs in git.

### 6. No judgment leaked in

Read `VERIFY.md` and the run's report for any opinion about scope, sizing or
whether the work was worth doing. There must be none — that is `/review`, and
this is the milestone where the two are most likely to blur.

### 7. Rules and process stayed in their own files

Read the feedback entries the run produced. A lesson about what to look for in
the diff must be `category: "review"`; a lesson about how the command runs must
be `process:verify`. Lines landing in the wrong one is the failure mode M5 is
watching for.

```bash
git status --short .anvil/    # no context file edited by the run
```

---

## Manual re-test — `/review`

Runs in `site`. Judging whether a slice held needs real PRs of real size.

### 1. It writes only to the logs

Write-protect the context files — **not** the whole directory, since the run must
still be able to append log entries:

```bash
chmod a-w .anvil/CONFIG.md .anvil/CRITICAL_PATHS.md .anvil/REVIEW_RULES.md .anvil/process/*.md
```

Run it. It must complete successfully and produce its report. Then:

```bash
chmod u+w .anvil/CONFIG.md .anvil/CRITICAL_PATHS.md .anvil/REVIEW_RULES.md .anvil/process/*.md
git status --short .anvil/    # only the two .jsonl files changed
```

A run that fails here is writing something it should not be. This is the
one-writer invariant, and `/review` is the command most likely to break it —
it produces exactly the kind of opinion that looks like it belongs in
`REVIEW_RULES.md`.

### 2. The boundary test

Run `/verify` and `/review` on the same finished ticket. Put the outputs side by
side.

**Any overlapping finding means the split is wrong.** Tighten whichever skill
definition let it through and run the pair again. Repeat until they are disjoint.

The specific overlap to hunt for: both read `REVIEW_RULES.md` and both look at
the diff. `/verify` applies each rule and reports a verdict; `/review` reads the
file to know what not to say. A `/review` finding that restates a rule is the
failure — and if `/verify` missed that rule, the defect is in `/verify` and the
lesson is `process:verify`, not a review finding.

Freeze this as an eval case once disjoint, so the boundary cannot silently
re-blur. Until eval access opens, re-run it by hand after any change to either
skill.

### 3. It reaches backwards

```bash
grep '"command":"review"' .anvil/feedback/*.jsonl | grep -c 'process:scope\|process:kickoff'
```

At least one `/review` entry must carry `process:scope` or `process:kickoff`, and
at least one such line must reach a context file through `/feedback`.

That is the whole reason this command exists. A `/review` whose lessons all land
in `process/review.md` has learned nothing about anything except itself.

### 4. No verdicts leaked in

Read the report for a `PASS`, a test run, a critical-path check, or anything
restating `VERIFY.md`. There must be none. `/review` gives no verdict on the
ticket either — whether it merges is the human's call.

### 5. Findings are anchored and actionable

Every concrete finding names file and line, and says what would have prevented
it. "This PR is doing too much" is a mood; "it does two things and the second is
the rename" is a finding.

A review with nothing to say must say so briefly rather than manufacture
findings — invented findings teach the logs a pattern that is not there.
