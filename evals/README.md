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
