---
name: research
description: Turn the corrections anvil could not route into one research question aimed outside the repo, and write nothing at all.
disable-model-invocation: true
---

# /research

The outward channel. Every other skill improves this repo from this repo's
evidence. This one turns what this repo could **not** resolve into a question for
someone else.

> **This command writes nothing. No context file, no ticket, no log entry, not
> even an edit to `unrouted.md`.** It emits a prompt to the session and stops.

That is not a small scope, it is the invariant. `/feedback` is the only command
that edits context files, and a second command reading the same evidence base and
writing to disk would be a second writer under a different name. It must complete
successfully with the whole of `${CLAUDE_PROJECT_DIR}/.anvil/` write-protected.

## 1. Precondition

If `${CLAUDE_PROJECT_DIR}/.anvil/` does not exist, **stop**. Tell the user this
repo has no anvil setup and to run `/setup`.

Your inputs live under that directory too. Missing means there is nothing to
research, not that you should go looking elsewhere.

## 2. Read exactly these

- `${CLAUDE_PROJECT_DIR}/.anvil/feedback/unrouted.md` — the primary input
- `${CLAUDE_PROJECT_DIR}/.anvil/feedback/human.jsonl`
- `${CLAUDE_PROJECT_DIR}/.anvil/feedback/self.jsonl`

**No context file.** Not `CONFIG.md`, not `REVIEW_RULES.md`, not any
`process/*.md`. Your cost to the read budget is zero and it stays that way.

The parked pile is the question set. The logs are there so you can quote the full
entry behind a bullet, and so you can see whether the same fingerprint has been
parked more than once.

## 3. When it is worth running

Run it when the pile is past ten entries, or when the same fingerprint has been
parked across several tickets. Those are the two shapes that mean something
structural rather than something incidental.

**Against an empty or near-empty pile, say so and stop.** Do not produce a
question anyway. A research prompt with no evidence behind it is the same error
as seeding a context file from a guess: it looks like output and it is made up.

## 4. The prompt

One question. Not a list.

Pick the pattern with the most evidence behind it, and state:

- **the question**, aimed at something outside this repo — how other projects
  route this kind of correction, whether a category is missing from the table,
  whether the thing being parked is a category error rather than a gap
- **the evidence**, quoted by `fb-NNNN` id, with enough of each entry that a
  reader who has never seen this repo can judge it
- **what an answer would change** — the routing table, a budget, a skill's
  instructions. If nothing would change, this is not the question worth asking

Aim outward. "Why does this repo keep parking verify entries" is a question
`/feedback` already has the evidence for. "Is 'how much of the suite to run' a
process lesson or a config fact, and how do other projects decide" is a question
this repo cannot answer from here.

## 5. Two destinations, and name both

1. **Run it yourself.** Paste it into Claude research or a similar tool and act
   on the answer in this repo.
2. **Send it upstream.** Open an issue on `anvil-skills` from the research issue
   template.

Upstream is the interesting one: it is the only path by which evidence from a
repo anvil's author will never see reaches the routing table. It costs a GitHub
issue and nothing else.

**Nothing leaves this repo unless a human pastes it.** There is no submission
endpoint, no telemetry, and no network call in this command. Say so if the user
asks where the prompt goes — the answer is "wherever you put it".

## 6. Then stop

Emit the prompt. Do not write it to a file, do not append it to `unrouted.md`,
do not mark anything `parked` or `dropped`, and do not log the fact that you ran.

If acting on the answer produces a change, that change comes back through the
normal route — a correction during a command, logged, promoted by `/feedback` on
evidence. Not from here.
