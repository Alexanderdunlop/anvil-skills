---
name: feedback
description: Capture what went wrong as feedback log entries — drafted from the conversation you just had, or from one you talk through, and written only once the user picks.
disable-model-invocation: true
---

# /feedback

Feedback in, log entries out. This command writes to
`${CLAUDE_PROJECT_DIR}/.anvil/feedback/` and nothing else.

It exists because **the corrections that matter most are given outside a
command**. Every anvil skill flushes what it saw during its own run, but most
real feedback arrives in ordinary conversation — mid-task, after the fact, or as
a complaint about how the whole thing went. Without this, that feedback lands in
a session that ends, and nothing compounds from it.

> **`/feedback` captures. `/improve` distils.** This command never touches a
> context file, never promotes anything, and never applies a threshold.

## 1. Precondition

If `${CLAUDE_PROJECT_DIR}/.anvil/` does not exist, **stop**. Tell the user this
repo has no anvil setup and to run `/setup`.

## 2. Two ways in, and both end the same way

**Drafted from the conversation.** Look back over the session for the moments the
user corrected you, told you to stop, re-did something you produced, or expressed
that an output was wrong. Draft one entry per distinct lesson and put the list to
them.

**Dictated.** The user says what to log. Take it, shape it into the schema, and
show them what it became before writing it.

Either way, **the user picks what gets written.** That is the whole design of
this command. Silent capture would fill the logs with entries nobody authored,
and since a fingerprint reaching two sightings is what promotes a lesson, junk
capture becomes junk in the context files two runs later — with the human's
approval never once asked for.

If the session holds nothing that looks like feedback, say so and stop. Do not
manufacture entries to have something to show.

## 3. Draft, then ask

Show the entries you propose in plain terms, numbered, and say which log each
would go to:

```
1. Test cases over-generated — one per criterion plus extras nobody asked for
   → human.jsonl, process:kickoff, fingerprint `test-cases-over-generated`
2. Asked which module was live instead of reading the flag registry
   → self.jsonl, process:kickoff, fingerprint `context-omits-feature-flag`
```

Then wait. Write only the ones they pick.

**Show the fingerprint.** It is the one field the user cannot fix later and the
one most likely to be wrong — it decides whether this lesson ever meets its own
second sighting. Let them rename it.

Take "log all of them" at face value; that is a real answer. Do not take silence
or a change of subject as one.

## 4. Which log

| Log           | Holds                                              | `correction`         |
| ------------- | -------------------------------------------------- | -------------------- |
| `human.jsonl` | a person corrected something, or said it was wrong | the words, or `null` |
| `self.jsonl`  | you noticed it yourself — a stall, a guess         | always `null`        |

The split is by **who noticed**, not by who is at fault. A user pointing out that
you over-generated test cases is `human.jsonl` even though you are the one who
did it.

`correction` is nullable in `human.jsonl` because "this is wrong" is a real entry.
Do not fabricate a fix the user did not give.

## 5. The entry

```json
{
  "id": "fb-NNNN",
  "ts": "ISO 8601 UTC",
  "ticket": "dir name or null",
  "command": "the command being corrected, or \"manual\"",
  "category": "one of the closed set below",
  "fingerprint": "kebab key for the lesson, not the instance",
  "observed": "what happened",
  "correction": "what the user said instead, or null",
  "proposed_line": "the one-line constraint this would become, or null",
  "status": "open"
}
```

**`command` is the command the feedback is about**, not this one. Feedback about
a `/kickoff` run is `"kickoff"` even though `/feedback` wrote it down. Use
`"manual"` when the feedback is not about any command — a note about the repo, or
about anvil itself.

**`category` records who should change**, which is frequently not who was
running. Test cases over-generated during `/kickoff` is `process:kickoff`; a
ticket that was scoped too broadly and only became obvious during `/build` is
`process:scope`.

The category set is closed: `claude-md`, `config`, `path`, `review`,
`process:scope`, `process:kickoff`, `process:build`, `process:verify`,
`process:review`, `ticket`, `unrouted`. Anything that does not fit is
`unrouted` — never invent a key. The pile of unrouted entries is the evidence for
what the table is missing, and inventing keys destroys it.

**`fingerprint` is short kebab-case, no spaces.** Matching is by exact string.
`test-cases-over-generated` is a fingerprint; "generated too many test cases in
the kickoff for ticket 3" is not. A prose fingerprint never matches its own
second sighting, so the lesson never reaches a threshold and the loop silently
does nothing while looking healthy.

Before you write, **check whether this lesson already has a fingerprint in the
logs** and reuse it if so. A second sighting under a new name is a first sighting
twice, and it is the most common way this system fails to learn.

**`proposed_line` is the constraint the lesson would become** — one line,
imperative, no hedging. It is what `/improve` would put in a context file, so
write it as a line you would want to read there, not as a description of what
happened.

**`fb-NNNN` is one sequence across both files** — the highest id in either, plus
one. Read both before you write either.

`ticket` is the ticket directory name if the feedback is about work on one, and
`null` otherwise. Do not guess it from the branch.

## 6. Then stop

Say what was written, and where. Then stop.

Do not promote anything. Do not touch a context file. Do not apply a threshold or
tell the user a lesson is "nearly there" — counting is `/improve`'s job and it
counts across both logs in ways this command does not need to know about.

Tell them `/improve` is what turns these into context-file lines, and that it
will ask before it changes anything.

## 7. Writing to the logs

Append one JSON object per line, never pretty-printed. Append only — do not
rewrite an existing entry, do not reorder, do not reformat a line you are not
adding.

Confirm before you exit:

```bash
python3 -c "import json;[json.loads(l) for f in ['human.jsonl','self.jsonl'] for l in open('${CLAUDE_PROJECT_DIR}/.anvil/feedback/'+f)]"
```

A log that no longer parses is a corrupted evidence base, and it is corrupted in
the one file that is supposed to be the permanent record.

## 8. Start cold is not a requirement here

Every other anvil command assumes no conversation history. This one is the
exception, and deliberately: the conversation **is** the input when you are
drafting from it.

That has a cost worth stating to the user — run `/feedback` before `/clear`, or
the session it would have drafted from is gone. Dictating still works after a
clear, but the drafting does not.
