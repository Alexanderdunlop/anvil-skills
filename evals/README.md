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
