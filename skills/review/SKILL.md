---
name: review
description: Judge whether the slice held, whether the change matches the plan, and whether the ticket was scoped right — and route what you learn back to the command that should change.
disable-model-invocation: true
---

# /review

The judgment half. Was this scoped right, is the PR one a person can review in
one sitting, did the slice hold, does the diff do what was planned.

**This command writes feedback entries and nothing else.** No context file, no
ticket file, no code. It is the command most likely to break the one-writer
invariant, because it produces exactly the kind of opinion that looks like it
belongs in `REVIEW_RULES.md`. It does not put it there.

## 1. Precondition

If `${CLAUDE_PROJECT_DIR}/.anvil/` does not exist, **stop**. Tell the user this
repo has no anvil setup and to run `/setup`.

## 2. Read exactly these

- `${CLAUDE_PROJECT_DIR}/.anvil/CONFIG.md`
- `${CLAUDE_PROJECT_DIR}/.anvil/process/review.md`
- `${CLAUDE_PROJECT_DIR}/.anvil/REVIEW_RULES.md` — see §3, it is read for a
  reason that is not the obvious one
- the ticket body, resolved through `CONFIG.md`'s `tracker:` mode — never by
  assuming `${CLAUDE_PROJECT_DIR}/.anvil/tickets/<id>/TICKET.md` exists
- `${CLAUDE_PROJECT_DIR}/.anvil/tickets/<id>/PLAN.md`
- the diff

Not `TEST_CASES.md`, not `CRITICAL_PATHS.md`, not `VERIFY.md`. Those are
`/verify`'s inputs and output, and reading them is how this command starts
repeating work that is already done.

The diff and the source carry no budget.

## 3. What `/verify` already did — do not do it again

`/verify` ran the cases, checked the critical paths, applied every line of
`REVIEW_RULES.md` to the diff, and wrote a verdict with evidence for each. That
work exists. Repeating it produces a second opinion on a settled question and
gives the human two overlapping documents instead of one set of facts and one
judgment.

> **You read `REVIEW_RULES.md` to know what _not_ to say.**

That is the whole reason the file is in your list. Every line in it is something
the repo already checks mechanically, so a finding that restates one is noise —
`/verify` either caught it, in which case you are repeating, or missed it, in
which case the bug is in `/verify` and the fix is a `process:verify` entry rather
than a review finding.

The division, stated once:

| `/verify`                            | `/review`                             |
| ------------------------------------ | ------------------------------------- |
| facts                                | judgment                              |
| per-case verdicts, evidence attached | whether the work was shaped right     |
| the cases, the paths, the rules      | the slice, the size, the plan, the AC |
| never an opinion                     | never a verdict                       |

If you find yourself running a test, checking a critical path, or writing PASS
next to anything, you have crossed the line.

## 4. What to judge

Four questions. Answer each one, and say plainly when the answer is "fine".

**Did the slice hold?** The ticket's `Out of scope` section is a set of promises.
Does the diff keep them? Work that crept in from the excluded list is the single
most common finding here, and it is a `process:scope` lesson far more often than
a fault in the person who wrote the code.

**Is this one PR a human can review in one sitting?** Say what makes it too big
when it is — a count of files, two unrelated concerns, a refactor riding along
with a feature. "It is large" is not a finding; "it does two things and the
second one is the rename" is.

**Does the diff do what `PLAN.md` said?** Steps skipped, steps done differently,
work done that no step describes. A plan that diverged is not automatically
wrong — but an undiscussed divergence means either the plan was thin or the
build went off-piste, and which one it was is the lesson.

**Are the acceptance criteria actually met?** Read them against the diff, not
against the tests — whether the tests pass is `/verify`'s answer, already given.
A criterion that no part of the diff addresses is the finding worth most here.

## 5. Findings

Name file and line for anything concrete. A finding a reader has to go hunting
for is a finding they will skip.

Say what would have prevented it, not just what happened. "The ticket should have
said the CSV writer is out of scope" is actionable; "this PR is doing too much"
is a mood.

Where nothing is wrong, say so briefly and stop. A review that manufactures
findings to look thorough teaches the logs a pattern that is not there.

## 6. Everything you learned goes to the logs

This is the output that outlives the review. A finding said out loud once
improves one PR; the same finding logged with a fingerprint improves every
future ticket, because `/feedback` promotes it into the file the responsible
command reads.

```json
{
  "id": "fb-NNNN",
  "ts": "ISO 8601 UTC",
  "ticket": "dir name or null",
  "command": "review",
  "category": "the command that should change",
  "fingerprint": "kebab key for the lesson, not the instance",
  "observed": "what you found, with file and line",
  "correction": "what the user said instead, or null",
  "proposed_line": "the one-line constraint this would become",
  "status": "open"
}
```

**`category` is where this command earns its place.** A finding about a slice
that did not hold is `process:scope` — the ticket should have drawn the boundary.
A finding about a divergence from a thin plan is `process:kickoff`. `process:review`
is for how _you_ run, and it is the rarest of the three.

Reaching backwards into the commands that ran before you is the entire reason
this command exists. A `/review` whose lessons all land in `process/review.md`
has learned nothing about anything except itself.

`fingerprint` is short kebab-case, no spaces. Exact-string matching means a prose
fingerprint never matches its own second sighting, so the lesson sits below the
threshold forever while the log looks healthy.

**`fb-NNNN` is one sequence across both files** — the highest id in either, plus
one. Read both before you write either.

The category set is closed: `claude-md`, `config`, `path`, `review`,
`process:scope`, `process:kickoff`, `process:build`, `process:verify`,
`process:review`, `ticket`, `unrouted`. Anything that does not fit is
`unrouted`. Never invent a category.

Corrections a person makes during the review go to
`${CLAUDE_PROJECT_DIR}/.anvil/feedback/human.jsonl`; things you noticed yourself
go to `${CLAUDE_PROJECT_DIR}/.anvil/feedback/self.jsonl` with
`"correction": null`.

**Write these before the command exits.** `/clear` runs between commands, so an
unflushed entry is a lost entry.

## 7. Write no context file

Not `REVIEW_RULES.md`, however plainly this diff proves a rule is missing. Not
`process/scope.md`, however sure you are the lesson will reach two sightings.
Not `CRITICAL_PATHS.md`, not `CONFIG.md`.

`/feedback` is the only command that edits a context file. The threshold exists
because one sighting is an anecdote, and a `/review` that writes its own
conclusion straight into a file has skipped the evidence bar every other lesson
had to clear.

This command must complete successfully with every context file write-protected.
If that would fail, it is writing something it should not be.

## 8. The report

- the four judgments from §4, including the ones that came back fine
- each finding, with file and line, and what would have prevented it
- what you logged, by fingerprint and category

Do not restate `VERIFY.md`. Do not give a verdict on the ticket. Whether it
merges is the human's call, and this command exists to make that call better
informed rather than to make it.

## 9. Start cold

Assume no conversation history. The files in §2 and the diff are the whole
input. If you needed something from earlier in the session, that is a
`self.jsonl` entry — write it down rather than working around it.
