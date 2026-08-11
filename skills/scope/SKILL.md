---
name: scope
description: Turn a rough idea into one sliced ticket with numbered, observable acceptance criteria, behind an approval gate.
disable-model-invocation: true
---

# /scope

A rough idea in, one sliced ticket out, behind a human approval gate. This
command writes the ticket body and the feedback logs. Nothing else.

## 1. Precondition

If `${CLAUDE_PROJECT_DIR}/.anvil/` does not exist, **stop**. Tell the user this
repo has no anvil setup and to run `/setup`.

Do not create the directory, do not write a partial `CONFIG.md`, do not infer
settings from what you can see, and do not proceed on defaults. Missing is a
clean state. Half-populated is a file nobody authored and nobody trusts.

## 2. Read exactly two files

- `${CLAUDE_PROJECT_DIR}/.anvil/CONFIG.md`
- `${CLAUDE_PROJECT_DIR}/.anvil/process/scope.md`

Not `REVIEW_RULES.md`, not `CRITICAL_PATHS.md`, not the feedback logs. A file
read here is charged to every future `/scope`.

**One exception: a prior ticket the user names.** If the idea is defined
relative to an existing ticket — "the follow-up 0001 cut", "same as 0004 but for
exports" — read that one ticket, and stay consistent with the decisions in it:
its output formats, its flag surface, what it put in `Out of scope`.

One ticket, named by the user, on the runs that refer to one. Never the newest
ticket, never every ticket touching the same code, never a scan of
`tickets/`. If the user gestures at an earlier ticket without identifying it,
ask which — do not go looking.

Reading the codebase to understand the idea is fine and expected — that is not a
context file and carries no budget.

Every line in `process/scope.md` is a constraint on how you scope this repo. It
is there because the same correction was made at least twice. Follow it as
written.

## 3. Resolve where the ticket body lives

Read `tracker:` from `CONFIG.md` first. Never assume
`${CLAUDE_PROJECT_DIR}/.anvil/tickets/<id>/TICKET.md` exists.

| `tracker:`                               | Body lives in | What `/scope` writes                     |
| ---------------------------------------- | ------------- | ---------------------------------------- |
| `local`                                  | `TICKET.md`   | `TICKET.md` — it **is** the ticket       |
| external, integration reachable this run | the tracker   | the ticket in the tracker, no local copy |
| external, no integration reachable       | the tracker   | `TICKET.md` as a labelled `MIRROR`       |

If `tracker:` names an external tracker and you cannot reach it in this session,
that is the third row. Say so before you write anything — the user needs to know
they are about to paste it in by hand.

Exactly one source of truth, and a mirror is always labelled as one.

## 4. Draft

Four sections, in this order:

```markdown
# Problem

# Scope

# Out of scope

# Acceptance criteria

1.
2.
```

**Acceptance criteria are numbered and observable.** Each one states something a
person can watch happen — a request that returns a 401, a file that downloads
with a header row. Never a file to edit or a function to add. If a criterion
names an implementation, rewrite it as the outcome that implementation produces.

**Out of scope is load-bearing.** It is what makes the slice a slice. An empty
Out of scope section usually means nothing was cut, which usually means the
ticket is not sliced.

**How to slice is not this file's decision.** This skill fixes the _shape_ of the
output. The rules for how small a ticket should be in this repo live in
`process/scope.md` and get there by being corrected twice. Do not add slicing
heuristics here — a plausible rule nobody in this repo has ever asked for is a
guess that occupies budget and gets believed.

Ask the user when the right slice is genuinely unclear. An ambiguous ticket is
more expensive than a question.

## 5. The gate

Present the draft **in the conversation**. Write no file yet. Then wait.

Proceed only on explicit approval of this draft — the user saying yes, approved,
go ahead, ship it, or equivalent, unmistakably about what you just showed them.

Not approval:

- a question about the draft
- a comment, a reaction, or a partial agreement
- "looks good, but ..." — anything with a `but` is a revision request
- "sure", "ok I guess", or anything you would have to interpret
- silence, or the user changing the subject

On anything that is not explicit approval, revise the draft and present it
again. Ambiguity is a reason to ask, never a reason to proceed. Never treat your
own summary of the draft as the user's approval.

**This run ends at the ticket.** Do not write `CONTEXT.md`, `PLAN.md`,
`TEST_CASES.md`, or any code, however obvious the next step looks. Those belong
to `/kickoff`, in a separate invocation after a `/clear`.

## 6. Write the ticket

**Id.** `NNNN-short-kebab-slug`. `NNNN` is the highest numeric prefix already in
`${CLAUDE_PROJECT_DIR}/.anvil/tickets/` plus one, zero-padded to four. There is
no counter file — read the directory. If `tracker:` is external, ask for the
external id and use `<external-id>-slug`.

**Budget.** 40 lines, checked with
`wc -l < ${CLAUDE_PROJECT_DIR}/.anvil/tickets/<id>/TICKET.md`. A ticket that will
not fit in 40 lines is a ticket that needs slicing again — do not compress the
prose to fit.

**Mirror header**, third row of §3 only. Line 1 is the literal word `MIRROR`:

```markdown
MIRROR
external: PROJ-482
updated: 2026-08-12

# Problem
```

Anvil never edits a mirror after writing it. If the tracker changes, the user
re-pastes.

## 7. Flush the feedback logs before you exit

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
  "command": "scope",
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
`unrouted`. Never invent a category — the pile of unrouted entries is the
evidence for what the table is missing, and inventing a key destroys it.

`fingerprint` identifies the lesson so the same mistake in a different ticket
matches. **Short kebab-case, no spaces** — `ac-written-as-implementation` is a
fingerprint; `wrong wording on ticket 12` is not, and neither is a prose
sentence describing what happened.

Matching is by exact string. A fingerprint written as prose never matches its
own second sighting, so the lesson never reaches the threshold and never gets
promoted — the failure is silent and looks like nothing was ever learned. Write
the key you would want the _next_ run to collide with.

**Write these before the command exits.** `/clear` runs between commands, so an
unflushed entry is a lost entry.

**Edit no context file.** Not `process/scope.md`, not `CONFIG.md`, not any of
them. `/feedback` is the only command that writes context files, and a second
writer is how they turn into a landfill.

## 8. Start cold

Assume no conversation history. `CONFIG.md`, `process/scope.md`, and what the
user just typed are the whole input. If you need something from earlier in the
session to finish, that is a `self.jsonl` entry — write it down rather than
working around it.
