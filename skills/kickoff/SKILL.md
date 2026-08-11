---
name: kickoff
description: Turn an approved ticket into a context file, a numbered plan and a test-case list, behind an approval gate.
disable-model-invocation: true
---

# /kickoff

One approved ticket in, three files out, behind a human approval gate. This
command writes the plan trio and the feedback logs. It writes no code.

## 1. Precondition

If `${CLAUDE_PROJECT_DIR}/.anvil/` does not exist, **stop**. Tell the user this
repo has no anvil setup and to run `/setup`.

Do not create the directory, do not write a partial `CONFIG.md`, do not infer
settings from what you can see. Missing is a clean state. Half-populated is not.

## 2. Read exactly three things

- `${CLAUDE_PROJECT_DIR}/.anvil/CONFIG.md`
- `${CLAUDE_PROJECT_DIR}/.anvil/process/kickoff.md`
- the ticket body, resolved through §3

Not `REVIEW_RULES.md`, not `CRITICAL_PATHS.md`, not the feedback logs, not
another ticket. A file read here is charged to every future `/kickoff`.

Reading the codebase is not only allowed, it is the job. Source files carry no
budget — the budget is on context files.

Every line in `process/kickoff.md` is a constraint on how you plan in this repo.
It is there because the same correction was made in two distinct tickets. Follow
it as written.

## 3. Resolve where the ticket body lives

Read `tracker:` from `CONFIG.md` **first**. Never assume
`${CLAUDE_PROJECT_DIR}/.anvil/tickets/<id>/TICKET.md` exists.

| `tracker:`                    | Body lives in | `TICKET.md`                       |
| ----------------------------- | ------------- | --------------------------------- |
| `local`                       | `TICKET.md`   | is the ticket — read it           |
| external, integration working | the tracker   | absent — fetch from the tracker   |
| external, no integration      | the tracker   | a labelled `MIRROR`, may be stale |

A `TICKET.md` whose first line is `MIRROR` is a copy, not the source. Say so
before you plan from it, because the tracker may have moved since it was
written.

If you cannot reach an external tracker and no mirror exists, **stop and ask**.
Planning from a ticket you have not read is the most expensive guess available
here.

## 4. Write three files, and nothing else

| File            | Budget   | Holds                                     |
| --------------- | -------- | ----------------------------------------- |
| `CONTEXT.md`    | 40 lines | the files that matter, and why            |
| `PLAN.md`       | 60 lines | numbered steps, each a checkable change   |
| `TEST_CASES.md` | 40 lines | one case per line, `<given> → <expected>` |

All three under `${CLAUDE_PROJECT_DIR}/.anvil/tickets/<id>/`. Check each with
`wc -l` before you finish. A trio that will not fit is a ticket that needs
slicing again — do not compress the prose to fit.

### `CONTEXT.md`

The files this change touches and what a reader needs to know about each that
**the code does not already say**. That last clause is the whole test:

> Could Claude have inferred this by opening the file? If yes, leave it out.

A list of file paths with no reason attached is inferable from the diff that has
not been written yet. What is not inferable: which of two similar modules is the
live one, what a name means when it does not mean what it says, a constraint
imposed from somewhere outside the code.

Ticket-tier lessons go here too — a correction that applies to this ticket and
no other. Log it with `"category": "ticket"` in the same run, because this file
is where that category lands and `/improve` will not write it later.

### `PLAN.md`

Numbered steps. **Each step is one checkable change**: after doing it, there is
something you can run or look at that tells you whether it worked. A step you
cannot check is a step you cannot know you finished.

State what a step changes, not how to type it. If a step changes stored data,
name what it changes — a step that says "persist the setting" without saying
where is the shape that stalls `/build`.

Do not write code in the plan. A plan containing the implementation is an
implementation with line numbers, and the gate below stops meaning anything.

### `TEST_CASES.md`

One case per line, `<given> → <expected>`. Observable both sides. These are the
cases `/verify` will produce verdicts for, so a case nobody can check is a case
that will come back as `COULD NOT CHECK`.

Every acceptance criterion in the ticket appears here as at least one case. If a
criterion cannot be turned into one, that is a defect in the ticket — say so
rather than inventing a case that dodges it.

## 5. The gate

Present all three files **in the conversation**. Write nothing yet. Then wait.

Proceed only on explicit approval — the user saying yes, approved, go ahead, or
equivalent, unmistakably about what you just showed them.

Not approval:

- a question about the plan
- a comment, a reaction, or a partial agreement
- "looks good, but ..." — anything with a `but` is a revision request
- "sure", "ok I guess", or anything you would have to interpret
- silence, or the user changing the subject

On anything that is not explicit approval, revise and present again. Ambiguity is
a reason to ask, never a reason to proceed. Never treat your own summary of the
plan as the user's approval.

**This run ends at the three files.** Do not start step 1, do not write code,
however obvious it looks. That is `/build`, in a separate invocation after a
`/clear`.

## 6. Flush the feedback logs before you exit

Append one JSON object per line — never pretty-printed.

- Corrections the user made during this run →
  `${CLAUDE_PROJECT_DIR}/.anvil/feedback/human.jsonl`
- Points where you stalled, guessed, or had to ask →
  `${CLAUDE_PROJECT_DIR}/.anvil/feedback/self.jsonl`, with `"correction": null`

```json
{
  "id": "fb-NNNN",
  "ts": "ISO 8601 UTC",
  "ticket": "dir name or null",
  "command": "kickoff",
  "category": "one of the closed set below",
  "fingerprint": "kebab key for the lesson, not the instance",
  "observed": "what you did",
  "correction": "what the user said instead, or null",
  "proposed_line": "the one-line constraint this would become, or null",
  "status": "open"
}
```

**`fb-NNNN` is one sequence across both files** — the next id is the highest in
either file plus one. Read both before you write either.

**The category set is closed:** `claude-md`, `config`, `path`, `review`,
`process:scope`, `process:kickoff`, `process:build`, `process:verify`,
`process:review`, `ticket`, `unrouted`. Anything that does not fit is
`unrouted`. Never invent a category.

`command` records who saw it. `category` records who should change. They are
frequently not the same skill — a gap you hit while planning is often a
`process:scope` lesson, because the ticket should have said it.

`fingerprint` is **short kebab-case, no spaces** —
`plan-omits-schema-change` is a fingerprint; a prose sentence is not. Matching is
by exact string, so a prose fingerprint never matches its own second sighting and
the lesson is never promoted. The failure is silent and looks like nothing was
ever learned.

**Write these before the command exits.** `/clear` runs between commands, so an
unflushed entry is a lost entry.

**Edit no context file.** Not `process/kickoff.md`, not `CONFIG.md`, not any of
them. `/improve` is the only command that writes context files, and a second
writer is how they turn into a landfill.

## 7. Start cold

Assume no conversation history. `CONFIG.md`, `process/kickoff.md` and the ticket
body are the whole input. If you need something from earlier in the session to
finish, that is a `self.jsonl` entry — write it down rather than working around
it.
