---
name: build
description: Execute an approved plan step by step from the ticket files alone, and log every point the plan was not enough.
disable-model-invocation: true
---

# /build

The plan trio in, code out. This command writes code and feedback entries. It
writes no file under `${CLAUDE_PROJECT_DIR}/.anvil/` except log lines.

Its second job matters as much as the first: **every point where the plan was
not enough gets written down**, because that is the only route by which
`/kickoff` gets better.

## 1. Precondition

If `${CLAUDE_PROJECT_DIR}/.anvil/` does not exist, **stop**. Tell the user this
repo has no anvil setup and to run `/setup`.

If the ticket has no `PLAN.md`, stop and say so. Run `/kickoff` first. Do not
plan and build in one run — that collapses the gate `/kickoff` exists to hold.

## 2. Read exactly these

- `${CLAUDE_PROJECT_DIR}/.anvil/CONFIG.md`
- `${CLAUDE_PROJECT_DIR}/.anvil/process/build.md`
- the ticket body, resolved through `CONFIG.md`'s `tracker:` mode — never by
  assuming `${CLAUDE_PROJECT_DIR}/.anvil/tickets/<id>/TICKET.md` exists
- `${CLAUDE_PROJECT_DIR}/.anvil/tickets/<id>/CONTEXT.md`
- `${CLAUDE_PROJECT_DIR}/.anvil/tickets/<id>/PLAN.md`
- `${CLAUDE_PROJECT_DIR}/.anvil/tickets/<id>/TEST_CASES.md`

Not `REVIEW_RULES.md`, not `CRITICAL_PATHS.md`, not the feedback logs, not
another ticket.

Reading the codebase is the job and carries no budget. The budget is on context
files, and the five above are the whole allowance.

## 3. Start cold

**Assume no conversation history.** The files in §2 and the repo are the entire
input. There was a `/clear` between `/kickoff` and you, and the person who
approved the plan is not necessarily in this session.

This is not a formality. If you find yourself needing something that was said
during planning, you have found a hole in `CONTEXT.md` or `PLAN.md` — that is a
`self.jsonl` entry, and writing it down is worth more than the workaround you
were about to reach for.

## 4. Execute the plan, step by step

Steps in order. After each one, check it: run the thing, read the output, look at
the state it was supposed to change. A step you cannot check is a step you cannot
report on, and §5 wants to hear about it.

**Stay inside the plan.** A step that turns out to need a change the plan does
not describe is a stall, not an invitation. Log it, do the smallest thing that
gets the step done, and say plainly in the report that you went beyond the plan
and where.

**Stop and ask when the plan is wrong**, rather than building something the
ticket did not ask for. Wrong is different from thin: a thin step you can fill in
and log, a wrong step produces the wrong software.

Run the tests your change touches. That is checking your work, not verifying the
ticket — per-case verdicts and `VERIFY.md` belong to `/verify`, and producing
them here means the same cases get judged twice by whoever wrote them.

## 5. Log every stall, before you exit

A stall is any point where you guessed, had to ask, went looking for something
the plan should have told you, or went beyond what the plan described.

Stalls are self-observed, so they go to
`${CLAUDE_PROJECT_DIR}/.anvil/feedback/self.jsonl` with `"correction": null`.
Corrections a person made during the run go to
`${CLAUDE_PROJECT_DIR}/.anvil/feedback/human.jsonl`.

```json
{
  "id": "fb-NNNN",
  "ts": "ISO 8601 UTC",
  "ticket": "dir name or null",
  "command": "build",
  "category": "the command that should change, not the one that noticed",
  "fingerprint": "kebab key for the lesson, not the instance",
  "observed": "what you hit, and what unblocked it",
  "correction": "what the user said instead, or null",
  "proposed_line": "what the plan should have said, as one line",
  "status": "open"
}
```

**`observed` says what unblocked you, not only that you were blocked.** "Stalled
at step 3" is a complaint. "Stalled at step 3 — it said persist the setting
without saying whether that meant a new column or a new table; found the answer
in the last two migrations" is evidence somebody can act on.

**`category` is usually `process:kickoff`, not `process:build`.** A hole in the
plan is a lesson for the command that wrote the plan. `process:build` is for how
_you_ run — the order you work in, what you check — and it is the rarer case.
Getting this backwards routes every lesson into the file that cannot fix it.

**`fingerprint` is short kebab-case, no spaces.** Matching is by exact string, so
a fingerprint written as prose never matches its own second sighting: the lesson
sits below the threshold forever while the log looks healthy.

**`fb-NNNN` is one sequence across both files** — the highest id in either,
plus one. Read both before you write either.

The category set is closed: `claude-md`, `config`, `path`, `review`,
`process:scope`, `process:kickoff`, `process:build`, `process:verify`,
`process:review`, `ticket`, `unrouted`. Anything that does not fit is
`unrouted`. Never invent a category.

**Write these before the command exits.** `/clear` runs between commands, so an
unflushed entry is a lost entry — and a stall you did not write down is a stall
`/kickoff` will cause again on the next ticket.

## 6. What this command does not write

- **No file under `${CLAUDE_PROJECT_DIR}/.anvil/` except log lines.** Not
  `CONTEXT.md`, not `PLAN.md`, not `TEST_CASES.md`, not `VERIFY.md`, and no
  context file. `/feedback` is the only command that edits a context file.
- **No new ticket**, however clearly the work has split in two. That is `/scope`.
- **No plan rewrite.** If `PLAN.md` is wrong, the fix is a log entry and a
  conversation, not an edit — the plan is the record of what was approved, and
  editing it makes the approval meaningless.

## 7. The report

End with:

- which steps are done, and what you checked to know it
- anything you went beyond the plan to do, and where
- every stall you logged, by fingerprint
- what is left, if the run did not finish

Say that the tests you ran passed, or that they did not. A run that reports
success without naming what it ran has reported nothing.
