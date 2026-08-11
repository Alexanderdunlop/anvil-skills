---
name: verify
description: Produce a per-case verdict for every test case, critical path and review rule, with command output as evidence and never a pass nobody observed.
disable-model-invocation: true
---

# /verify

Every case gets a verdict, and every verdict carries evidence. This command
writes `VERIFY.md` and the feedback logs. Nothing else.

**There are only three verdicts: `PASS`, `FAIL`, `COULD NOT CHECK`.** Nothing
here is a judgment about whether the work was any good — that is `/review`, and
the separation is the point.

## 1. Precondition

If `${CLAUDE_PROJECT_DIR}/.anvil/` does not exist, **stop**. Tell the user this
repo has no anvil setup and to run `/setup`.

## 2. Read exactly these

- `${CLAUDE_PROJECT_DIR}/.anvil/CONFIG.md`
- `${CLAUDE_PROJECT_DIR}/.anvil/process/verify.md`
- `${CLAUDE_PROJECT_DIR}/.anvil/REVIEW_RULES.md`
- `${CLAUDE_PROJECT_DIR}/.anvil/CRITICAL_PATHS.md`
- `${CLAUDE_PROJECT_DIR}/.anvil/tickets/<id>/TEST_CASES.md`

Not the ticket body, not `PLAN.md`, not the feedback logs. You verify against the
cases, not against the intent — reading the ticket invites re-litigating what it
should have said, which is `/review`'s job.

The diff and the source carry no budget. Read as much of them as the checks
need.

## 3. Three sources of cases, one file of verdicts

| Source                             | Produces                                     |
| ---------------------------------- | -------------------------------------------- |
| `TEST_CASES.md`                    | one verdict per case                         |
| `CRITICAL_PATHS.md`                | one verdict per path — every run, every time |
| `REVIEW_RULES.md` against the diff | one verdict per rule                         |

**Every line in all three gets a verdict.** A case you skipped is invisible in
the output, which makes a partial run look like a clean one. If you cannot check
it, that is `COULD NOT CHECK` and it belongs in the file.

Critical paths are checked whether or not the diff looks like it touches them.
That is what makes them critical: the interesting failure is the one nobody
expected the change to cause.

**`REVIEW_RULES.md` governs the diff. `process/verify.md` governs you.** If a
line still makes sense with no diff in front of you, it belongs in
`process/verify.md` and you should not be applying it to code. If it only means
anything while reading code, it is a review rule. These two files collect each
other's lines if nobody watches, so watch.

## 4. Evidence, or it is not a pass

The evidence is **a command and what it returned**, or an observation of
something that happened.

Not evidence, ever:

- "works correctly"
- "the login flow was tested"
- "this looks right"
- the code appearing to do the thing

> **No command and no observation means `COULD NOT CHECK`.** Saying so is
> always better than a `PASS` nobody witnessed.

`FAIL` is a normal outcome and does not stop the run. Record it, with what
failed and what the command returned, and continue to the remaining cases. A run
that stops at the first failure reports one problem and hides the rest.

## 5. Degrade, never fail the run

`CONFIG.md`'s `smoke:` key names a smoke path — usually an MCP server. It may be
`none`, and the server it names may be absent from this session. Both are normal.

In either case:

- verdict is `COULD NOT CHECK`
- the evidence line says why — `smoke: none`, or the server was unavailable
- **the line carries the manual steps** so a person can finish the job

Then carry on with every other case. `/verify` never fails a run for want of a
smoke driver, never converts an unchecked case into a pass, and never tells the
user to go and install something before it will work.

Portability to any repo, any language, depends on this. A command that only
works where MCP works is not a command that works anywhere.

## 6. Write `VERIFY.md`

`${CLAUDE_PROJECT_DIR}/.anvil/tickets/<id>/VERIFY.md`. One line per case:

```
<case> — PASS|FAIL|COULD NOT CHECK — <evidence>
```

```markdown
# Verify — 0009-export-csv

Empty cart exports header row only — PASS — `pnpm vitest run export` 14 passed
Column order matches the spec — FAIL — `pnpm vitest run export.order` 1 failed, got id,name,total
Login — PASS — `curl -si localhost:3000/api/session -d @seed.json` returned 200, Set-Cookie present
Checkout — COULD NOT CHECK — CONFIG.md has `smoke: none`; run manually: cart → pay → confirmation shows order id
Flag any new public function without a test — PASS — `rg '^export function' src/export.ts` 2 hits, both covered in export.test.ts
```

**No budget, and no prose.** The file grows with the number of cases and with
nothing else. A `VERIFY.md` containing paragraphs is a bug.

**Overwrite it on re-run.** It reflects the current state of the ticket, not its
history. The history is in git.

Nobody reads this file but humans. No skill loads it — not `/review`, not
`/feedback` — so it is written for a reviewer opening the PR a week later, and
that is the only audience to write for.

## 7. Produce no judgment

No opinion on whether the ticket was sized right, whether the approach was
sensible, whether the work was worth doing. Not in `VERIFY.md`, not in the
report.

Those are real and valuable and they belong to `/review`. A `/verify` that
drifts into them ends up duplicating a command that reads different files, and
the reviewer gets two overlapping opinions instead of one set of facts.

## 8. Flush the feedback logs before you exit

Append one JSON object per line — never pretty-printed.

- Corrections the user made during this run →
  `${CLAUDE_PROJECT_DIR}/.anvil/feedback/human.jsonl`
- Points where you stalled, guessed, or could not check something you should
  have been able to → `${CLAUDE_PROJECT_DIR}/.anvil/feedback/self.jsonl`, with
  `"correction": null`

```json
{
  "id": "fb-NNNN",
  "ts": "ISO 8601 UTC",
  "ticket": "dir name or null",
  "command": "verify",
  "category": "one of the closed set below",
  "fingerprint": "kebab key for the lesson, not the instance",
  "observed": "what you did",
  "correction": "what the user said instead, or null",
  "proposed_line": "the one-line constraint this would become, or null",
  "status": "open"
}
```

The two categories this command produces most:

- **`path`** — a flow that must not break and is not in `CRITICAL_PATHS.md`, or
  one whose check is not executable. Routes on first sighting, because it is a
  fact about the repo rather than a lesson about behaviour.
- **`process:verify`** — how this command should run. What to check first, what
  to skip, how much of the suite to run.

A line about what to look for **in the diff** is `review`, not
`process:verify`. Routing those two wrongly is the failure this milestone exists
to catch.

**`fb-NNNN` is one sequence across both files** — the highest id in either, plus
one. Read both before you write either.

The category set is closed: `claude-md`, `config`, `path`, `review`,
`process:scope`, `process:kickoff`, `process:build`, `process:verify`,
`process:review`, `ticket`, `unrouted`. Anything that does not fit is
`unrouted`. Never invent a category.

**Write these before the command exits.** `/clear` runs between commands, so an
unflushed entry is a lost entry.

**Edit no context file.** Not `CRITICAL_PATHS.md`, not `REVIEW_RULES.md`, not
`process/verify.md`. `/feedback` is the only command that writes them, and a
`/verify` that adds its own critical path is the second writer that turns the
file into a landfill.

## 9. The report

Say how many cases passed, failed and could not be checked, and name every
`FAIL` and every `COULD NOT CHECK` with its reason. Point at `VERIFY.md`.

Never summarise a run as verified when anything came back `COULD NOT CHECK`. Say
what was checked and what was not.

## 10. Start cold

Assume no conversation history. The files in §2, the diff and the repo are the
whole input. If you needed something from earlier in the session, that is a
`self.jsonl` entry — write it down rather than working around it.
