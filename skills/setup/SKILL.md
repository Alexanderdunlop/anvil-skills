---
name: setup
description: Create the .anvil directory for a repo that has never seen anvil, or update one that has, from what the user tells you rather than from what you can guess.
disable-model-invocation: true
---

# /setup

The generator. Every other skill stops and points here when
`${CLAUDE_PROJECT_DIR}/.anvil/` is missing, so this is the one command that may
create it.

> **The governing rule: seed a file from what the user tells you. Never from
> what you assume.**

Everything below is an application of that. A file anvil populated by guessing
is a file nobody authored and nobody trusts, and it costs budget on every run
while being believed.

## 1. Which mode

- **`${CLAUDE_PROJECT_DIR}/.anvil/` absent** → fresh generation, §3–§6.
- **Present** → update, §7. Never an overwrite.

Check before anything else. Getting this backwards destroys work that took
milestones to accumulate.

## 2. Scan the repo, then ask

Read the repo first: manifests, test config, CI, directory shape, and the
`CLAUDE.md` if there is one. That reading exists to make your **questions**
better, not to fill in **answers**.

The distinction is the whole skill. Detecting `vitest` tells you to ask "is the
export test the check for the checkout path, or is there a smoke script?" It does
not tell you to write a line about vitest — that line would fail the admission
test, because anyone can read the manifest.

**If the repo has a `CLAUDE.md`, read it and duplicate nothing from it.** It is
read by every tool in the repo already. A constraint stated there and restated in
an anvil file is one constraint in two places, which is how the two drift apart.
Anvil never writes `CLAUDE.md`.

**Ask rather than guess. Omit rather than fill.** Anything you cannot detect with
confidence is a question. Anything the user does not know is left out of
`CONFIG.md` entirely — not written blank, not filled with a plausible default.

## 3. `CONFIG.md` — anvil's own config, and pointers outside the repo

Ask for these four, and only these:

| Key       | Ask                                                             |
| --------- | --------------------------------------------------------------- |
| `tracker` | `local`, or the name and project key of an external tracker     |
| `tickets` | where ticket directories live, if not the default               |
| `design`  | a design source anvil cannot read from here — a Figma file, say |
| `smoke`   | an MCP server that can drive the app, or `none`                 |

`smoke: none` is a **valid and common answer**, and offering it is not a failure
to configure something. `/verify` degrades to a manual checklist when it is set,
and never fails a run for want of a smoke driver. Do not push the user toward
installing an MCP server.

**Budget 10 lines.** Four keys is typical.

**These do not belong here, ever:** build, test and lint commands, entry points,
directory layouts, framework names. All of them are readable from the repo, which
is the "could Claude have inferred this" case — and worse, a stale lookup gets
re-run while a stale _line_ gets trusted. If a command is genuinely
undiscoverable, that is a `CLAUDE.md` fact the user writes, not a `CONFIG.md`
one.

```markdown
# Config

tracker: local
tickets: .anvil/tickets/
smoke: none
```

## 4. `CRITICAL_PATHS.md` — what must not break

Ask what breaks worst if it silently stops working, and for each one, the
cheapest way to tell. **The check must be executable or observable.** "Make sure
it works" is not a check.

Format, one per line: `<name> — <check>`. **Budget 40 lines.**

```markdown
# Critical paths

Login — POST /api/session with seed creds returns 200 and sets cookie
Checkout — cart → pay → confirmation page shows order id
```

**When the user names something real that cannot be written as a check** — "the
throw arc feels right" — do not quietly drop it and do not invent a check that
does not measure it. Write a feedback entry recording it, tell the user it is
recorded, and move on. That gap is a finding about the contract, and the entries
are the evidence for fixing it.

## 5. `REVIEW_RULES.md` — what to look for in a diff

Ask what they always end up saying in code review. One check per line,
imperative, stated so a reader can tell whether a diff passes or fails it.

**Budget 30 lines.** Read by `/verify` and `/review` only.

```markdown
# Review rules

Flag any new public function without a test.
Flag error paths that swallow the error and return a default.
```

If a line would still make sense with no diff in front of you, it is not a review
rule. It is a note about how a command runs, and it has no home here — that file
fills from corrections, not from setup.

## 6. Write the rest, and write it empty

```
${CLAUDE_PROJECT_DIR}/.anvil/
  CONFIG.md              from §3
  CRITICAL_PATHS.md      from §4
  REVIEW_RULES.md        from §5
  process/
    scope.md             "# Process: scope" and nothing else
    kickoff.md           "# Process: kickoff"
    build.md             "# Process: build"
    verify.md            "# Process: verify"
    review.md            "# Process: review"
  feedback/
    human.jsonl          empty
    self.jsonl           empty
    unrouted.md          "# Unrouted" and nothing else
```

**All five `process/*.md` ship with their title line and nothing else.** One line
each, checkable with `wc -l`. There is no question you can ask that fills them,
because the answer would be a prediction about what this repo will get wrong.

Seeding them is a natural idea and it is wrong. A universal lesson that has not
been observed _here_ is still a guess, however plausible — and nothing in the
file distinguishes a shipped guess from a line the repo earned through two
independent sightings. An empty file costs nothing and misleads nobody.

**Do not create `BUDGETS.md`.** Absent means defaults, and that is the expected
state. Writing one pre-filled with the shipped numbers would freeze a default
into a decision nobody made, in every repo anvil touches. A project writes it
when it wants to override something.

**Do not create `tickets/`.** `/scope` makes it when there is a first ticket, and
an empty directory does not survive git anyway.

Tell the user `${CLAUDE_PROJECT_DIR}/.anvil/` is committed to git. Feedback
compounds across a team, and a bloated context file showing up as PR noise is a
feature.

## 7. Update mode — never an overwrite

A repo with an existing `${CLAUDE_PROJECT_DIR}/.anvil/` has history in it: lines
a human wrote, and lines `/improve` promoted after two independent sightings.
Both are more valuable than anything you can generate.

- **Change no existing line.** Not to reword it, not to reformat it, not to sort
  it.
- **Remove nothing.** Removal is `/improve`'s job and it has rules about it.
- **Add only what the user asks for in this run.**
- **Create only what is missing** — a `process/*.md` that does not exist yet,
  written title-only like any other.

Say what you propose to change and wait for the user to confirm before writing.
Then a re-run with no new answers changes nothing at all, which is the property
worth having: `git diff` prints nothing.

## 8. Budgets bind here too

Check every file you write with `wc -l` against its budget — 10, 40, 30, and one
line each for `process/*.md`.

If the user's answers do not fit, **do not silently truncate and do not quietly
extend**. Say which ceiling was hit, show what is competing, and let them cut.
If they would rather raise the number, tell them the key to put in
`BUDGETS.md` — `critical_paths: 60`, say — and let them write it. `/setup` does
not create that file, and it does not decide a budget on a project's behalf.

## 9. Log what you learn, before you exit

Append one JSON object per line to
`${CLAUDE_PROJECT_DIR}/.anvil/feedback/human.jsonl` for corrections the user
made, or `${CLAUDE_PROJECT_DIR}/.anvil/feedback/self.jsonl` with
`"correction": null` for things you noticed yourself. `"command": "setup"`.

The category set is closed: `claude-md`, `config`, `path`, `review`,
`process:scope`, `process:kickoff`, `process:build`, `process:verify`,
`process:review`, `ticket`, `unrouted`.

**There is no `process:setup` key.** A lesson about how this command runs is
`unrouted` until the table gains one, and the pile of those entries is the
evidence for whether it should. Never invent the key.

`fingerprint` is short kebab-case, no spaces — exact-string matching means a
prose fingerprint never matches its own second sighting. `fb-NNNN` is one
sequence across both files.

## 10. Then stop

Tell the user what was created, what was left empty on purpose, and that
`process/*.md` fill up from corrections rather than from setup.

Do not scope a ticket, do not plan anything, do not write code. The next command
is `/scope`, in a fresh session.
