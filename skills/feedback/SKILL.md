---
name: feedback
description: Distil the feedback logs into the .anvil context files — promote on evidence, evict at the ceiling, and present what has gone stale.
disable-model-invocation: true
---

# /feedback

Raw log entries in, distilled context files out. This is the only command that
edits a context file. Every other command appends to the logs and edits nothing.

## 1. Precondition

If `${CLAUDE_PROJECT_DIR}/.anvil/` does not exist, **stop**. Tell the user this
repo has no anvil setup and to run `/setup`.

Do not create the directory, do not write a partial file, do not infer settings
from what you can see. Missing is a clean state. Half-populated is not.

## 2. What you read, and what you may write

Read, in this order:

- `${CLAUDE_PROJECT_DIR}/.anvil/BUDGETS.md` — if absent, defaults. Not an error.
- `${CLAUDE_PROJECT_DIR}/.anvil/feedback/human.jsonl`
- `${CLAUDE_PROJECT_DIR}/.anvil/feedback/self.jsonl`
- `${CLAUDE_PROJECT_DIR}/.anvil/feedback/unrouted.md`
- `${CLAUDE_PROJECT_DIR}/.anvil/CONFIG.md`
- `${CLAUDE_PROJECT_DIR}/.anvil/CRITICAL_PATHS.md`
- `${CLAUDE_PROJECT_DIR}/.anvil/REVIEW_RULES.md`
- all five of `${CLAUDE_PROJECT_DIR}/.anvil/process/*.md`
- `${CLAUDE_PROJECT_DIR}/CLAUDE.md` — **for the duplication check only** (§9)

You may write `CONFIG.md`, `CRITICAL_PATHS.md`, `REVIEW_RULES.md`, the five
`process/*.md`, `feedback/unrouted.md`, the `status` field of existing log
entries, and new entries appended to `feedback/self.jsonl`.

You may **never** write `BUDGETS.md`, `CLAUDE.md`, any ticket file, or any code.

## 3. Resolve every budget before you write anything

A budget checked after the fact is a file already over its ceiling.

| File                | Key              | Default |
| ------------------- | ---------------- | ------- |
| `CONFIG.md`         | `config`         | 10      |
| `CRITICAL_PATHS.md` | `critical_paths` | 40      |
| `REVIEW_RULES.md`   | `review_rules`   | 30      |
| `process/<cmd>.md`  | see below        | 20      |
| — (not a count)     | `stale_after`    | 20      |

`BUDGETS.md` is `key: number`, one per line. Any key it does not name keeps its
default, and no `BUDGETS.md` at all means every default above. That is the
expected case, not a missing file.

For a `process/*.md` the resolution order is `process:<command>`, then
`process`, then 20. `process:scope: 40` beats `process: 30` for `scope.md` only.

**A line is any line, including the title and blanks.** No exemptions —
exemptions get gamed. Check with `wc -l < <file>`, never by eye.

## 4. Count sightings

- A sighting is matched by **exact `fingerprint` string**. Nothing fuzzy. A
  fingerprint written as prose never matches its own second sighting; if you see
  one, say so in the report — it is a bug in the command that wrote it.
- **Count across both files.** One sighting in `human.jsonl` and one in
  `self.jsonl` is **two**, not one each. This is the rule the two-file split
  exists to protect: a lesson a person flagged and the model later hit on its
  own is the best-evidenced kind in the system, and per-file counting is how it
  would score lowest.
- Thresholds count **distinct tickets**, not entries. Five entries from one
  ticket are one sighting.
- **`ticket: null` is one bucket.** Every null-ticket entry, in either file,
  counts as a single distinct ticket however many there are. Two entries written
  outside any ticket are one sighting; a null and a real ticket are two. Entries
  from one sitting must not promote a line by themselves.
- Only `open` entries are routed. `promoted`, `parked` and `dropped` are
  history: read them for age (§9), never re-route them.

## 5. The admission test

Every open entry faces all three questions before it goes anywhere.

1. **Could Claude have inferred this from the code?** If yes, drop it.
2. **Is it stated as a direct constraint rather than a suggestion?** If not,
   rewrite it until it is.
3. **Does it fit on one line?** If not, it is two lessons or it is not a lesson.

Question 1 failing sets `status: "dropped"`. Question 2 failing is a rewrite of
`proposed_line`, not a rejection — "prefer", "consider" and "try to" all fail it.
Question 3 failing leaves the entry `open` and names it in the report as
carrying two lessons; a human splits it, because guessing the split is how one
correction becomes two wrong lines.

**Record which question killed every entry you dropped.** A rejection with no
reason is indistinguishable from a routing bug.

Most corrections fail question 1. They die in the logs. That is the system
working, not failing.

## 6. Route

The category set is **closed**. An entry carrying a category not in this table is
a defect in the command that wrote it: leave it `open`, report it, and do not
invent a destination.

| `category`        | Destination                | Threshold          |
| ----------------- | -------------------------- | ------------------ |
| `claude-md`       | proposed to `CLAUDE.md`    | 3 distinct tickets |
| `config`          | `CONFIG.md`                | 1 — a fact         |
| `path`            | `CRITICAL_PATHS.md`        | 1 — a fact         |
| `review`          | `REVIEW_RULES.md`          | 2 distinct tickets |
| `process:scope`   | `process/scope.md`         | 2 distinct tickets |
| `process:kickoff` | `process/kickoff.md`       | 2 distinct tickets |
| `process:build`   | `process/build.md`         | 2 distinct tickets |
| `process:verify`  | `process/verify.md`        | 2 distinct tickets |
| `process:review`  | `process/review.md`        | 2 distinct tickets |
| `ticket`          | that ticket's `CONTEXT.md` | 1, at capture time |
| `unrouted`        | `feedback/unrouted.md`     | n/a                |

`category: "ticket"` is **not yours to write**. The capturing command already
wrote that line into the ticket's `CONTEXT.md`; flip the entry to `promoted` and
touch no file.

`category: "unrouted"` appends one bullet to `unrouted.md` carrying the
`fb-NNNN` id and the date, and sets the entry to `parked`.

`review` versus `process:verify`, which otherwise collect each other's lines: if
the line still makes sense with no diff in front of you it is `process/verify.md`;
if it only means anything while reading code it is `REVIEW_RULES.md`.

## 7. Write the line, and pay for it

A promoted line is the entry's `proposed_line`: one line, imperative, no
rationale, no URL, no date, no promotion marker, no citation count. The file is
constraint only — the history is in the logs.

**Under budget.** Append it.

**At budget — one in, one out.** Name the line you are evicting, state why the
new line is worth more, and append the evicted text to `self.jsonl` as an entry
with `"command": "feedback"`, `"category": "unrouted"`, `"status": "parked"`,
and the trigger that fired. Nothing leaves a context file without landing in a
log.

**A removal entry is a record, not a lesson.** It is born `parked` and is never
routed on a later run — an `open` one would become an `unrouted.md` bullet on
the next invocation, which breaks idempotence and buries the signal that pile
exists to carry.

**Nothing weaker than the new line — fail hard.** Report the conflict and
present **both** ways out:

- evict a specific named line, or
- raise this file's budget in `BUDGETS.md`.

Then do neither. Do not ship a file one line over. Do not silently drop the
lesson. Do not lower the bar until something looks evictable.

> **You never write `BUDGETS.md`, and you never edit a budget key — including
> when the user says raising it is obviously right.** Give them the exact
> `key: number` line to add and let them add it.

Raising a ceiling is a human decision by definition: the whole meaning of the act
is that someone looked at the trade and chose to pay more. A tool that raises its
own ceiling at the moment the ceiling binds has not made that decision, it has
deleted the only point in the system where someone is forced to say which of two
constraints actually matters. The hard failure **is** the mechanism.

When a file is full, look for a stale line before you reach for the ceiling.

## 8. `claude-md` proposes and never writes

At three distinct tickets, print the proposed one-line addition and the reason,
and stop. Write nothing. The entries stay `open`.

Never mark your own proposal `promoted`. You cannot see whether the human
accepted it, and the one thing that closes it is the line turning up in
`CLAUDE.md` — which trigger 2 sees on a later run.

## 9. The staleness pass — every run

Run this on every invocation, not only when a file is at its ceiling. A line that
has quietly stopped earning its place is charged to every run of its command
forever, and nothing about a full file is what makes that wrong.

Three triggers. They do not have the same authority.

**Trigger 1 — budget pressure.** §7. One in, one out, the evicted line named and
justified, hard fail when nothing existing is weaker.

**Trigger 2 — duplication. Removes automatically.** Read the repo's `CLAUDE.md`
and check whether the substance of any context-file line now appears there. If it
does, remove the anvil-side line, and append the removed text to `self.jsonl`
naming trigger 2 — same shape as §7, born `parked` and never re-routed. Nothing is lost: the constraint still holds, stated once
instead of twice, and two statements of one constraint is how the two drift
apart.

> **If the removed line's fingerprint has an open `claude-md` proposal, flip that
> proposal to `promoted` in the same pass.** The line appearing in `CLAUDE.md`
> **is** the human's acceptance, observed rather than assumed. Left un-flipped,
> you re-propose it forever against a `CLAUDE.md` that already contains it.

This trigger is not budget-driven. It fires on a file nowhere near its ceiling.
**`CLAUDE.md` is read here and never written.**

**Trigger 3 — obsolescence. Presents, and never removes.** A line promoted more
than `stale_after` tickets ago whose fingerprint has not been seen since: name
the line, say when it was promoted and how long its fingerprint has been quiet,
and **stop there**. The user drops it or keeps it. Removing on this evidence
alone is the failure to avoid.

A fingerprint going quiet has two readings that look identical in the logs — the
lesson is dead weight because Claude is now good enough at that pattern, or the
line is working and the silence is the line doing its job. The second is common
and you cannot tell it from the first.

Log a trigger-3 removal when the user accepts it, never when you present it.

**Age comes from the logs.** Every promoted line has at least one entry behind it
carrying `ts`, `ticket` and `status: "promoted"`. "Tickets since" is the count of
**distinct non-null `ticket` values** appearing in entries with a later `ts` than
that promotion. Do not date-stamp context files and do not build citation
tracking to answer a question the logs already answer.

**Line counts after a staleness pass go down or stay level. Never up.**

## 10. Statuses, rewritten in place

The logs are append-only in their **entries**. The `status` field is the one
thing you may rewrite on an existing line.

- `open` → `promoted` — the line landed in a context file, or trigger 2 saw the
  accepted `CLAUDE.md` line
- `open` → `parked` — routed to `unrouted.md`
- `open` → `dropped` — failed admission question 1
- `open` stays `open` — below threshold, or a `claude-md` proposal awaiting a
  human

When you rewrite a line, change **only** `status`. Same field order, every other
value byte-for-byte, same line order, one object per line, never pretty-printed.
Confirm before you exit:

```bash
wc -l ${CLAUDE_PROJECT_DIR}/.anvil/feedback/*.jsonl
python3 -c "import json;[json.loads(l) for f in ['human.jsonl','self.jsonl'] for l in open('${CLAUDE_PROJECT_DIR}/.anvil/feedback/'+f)]"
```

Line counts must be unchanged except for entries you appended, and both files
must still parse. A reformatted log is a corrupted evidence base.

Your own appended entries use `"command": "feedback"` and the shared `fb-NNNN`
sequence — the next id is the highest in **either** file plus one.

## 11. Idempotence

**Two consecutive runs: the second changes nothing.** It promotes nothing,
removes nothing, appends no bullet and no log entry.

Before writing any line, check whether it is already in the file. A promoted
entry whose `status` you failed to flip gets re-promoted on every future run, and
the duplicate line looks exactly like a line the file earned.

Re-presenting a trigger-3 line the user chose to keep is fine — presenting is not
changing. Changing a file twice for one lesson is not.

## 12. The report

End every run with:

- promoted — file, and the line
- evicted — the line, and why the new one beat it
- removed by trigger 2 — the line, and the `CLAUDE.md` line that duplicates it
- presented by trigger 3 — the line, promoted when, quiet how long
- proposed for `CLAUDE.md` — the line, and why
- rejected — each one, and **which admission question killed it**
- parked — count
- **the `unrouted.md` count**, every run, even when it is zero

Above ten unrouted entries, say that the category table needs review. That pile
is the evidence for what the table is missing, not a rubbish bin.

## 13. Never

- write `BUDGETS.md` or `CLAUDE.md`, or edit any budget key
- ship a context file over its resolved budget
- remove a line under trigger 3 without the user accepting it
- add a date stamp, promotion marker or citation count to a context file
- promote a line nobody logged, or seed one from a plausible guess
- edit any field of a log entry other than `status`, and never a `ts` or a
  `ticket` — a forged entry is indistinguishable from a real one afterwards, and
  the logs are the evidence for anvil's central claim
- invent a category

## 14. Start cold

Assume no conversation history. The files in §2 are the whole input. If you need
something from earlier in the session to finish, that is a `self.jsonl` entry —
write it down rather than working around it.
