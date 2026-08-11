# Philosophy

Why anvil is shaped the way it is. `FILE_CONTRACT.md` says **what** every file
holds and what it costs; this file says **why** anyone would accept those
constraints. If the two disagree, the contract is right and this file is out of
date.

---

## Context is a tax

The claim the whole project rests on:

> A line in a context file is paid every time the command that reads it runs —
> forever.

Nothing else here is surprising once that is taken seriously. It reframes every
"should we write this down?" question as a purchase rather than a note, and the
default answer to a purchase with no stated benefit is no.

The reason it needs stating at all is that the cost is invisible at the moment
you pay it. Adding a line feels free — it is one line, you are already in the
file, and it is obviously true. The bill arrives distributed across every future
run, charged to someone who was not in the room and cannot see what the line was
for. Files that grow this way are not badly maintained; they are maintained by
people making an individually reasonable decision every single time.

So the mechanism has to be structural. A convention that says "keep it small"
loses to a person with a good reason, every time, and the person is usually
right about their line and wrong about the total.

## Why not `CLAUDE.md`

The most common question, and the answer has not moved since the origin note:
**the context is only relevant to the skill.**

`CLAUDE.md` is read in every session, whether or not you are doing the thing the
line is about. A note about how to slice a ticket is charged to a session spent
debugging CSS. Put it in `process/scope.md` instead and it is charged to `/scope`
and nothing else.

This cuts both ways, and the second direction matters more than it sounds.
Anvil deleted the one file it owned that was read on every run, because no line
was ever found that belonged there rather than in `CLAUDE.md` — see
`FILE_CONTRACT.md` §4.10. Anvil now owns **no** always-read file. When a lesson
really does apply to every session, the right destination is your `CLAUDE.md`,
and anvil proposes the line rather than writing it. Duplicating a constraint in
two files is how the two drift apart, and the drift is silent.

The precedent is worth looking at, because it is the admission test applied by
someone with every excuse not to. Apple shipped a `CLAUDE.md` that documented an
enormous codebase in eight lines: what streams use, which classes are actors,
what a protocol abstracts. Nothing the code could tell you.
[What Apple's leaked CLAUDE.md teaches us](https://medium.com/vibe-coding/what-apples-leaked-claude-md-teaches-us-b8269e2ace51).

## Most corrections are not lessons

Three questions, and a correction has to pass all three:

1. Could Claude have inferred this from the code?
2. Is it stated as a direct constraint rather than a suggestion?
3. Does it fit on one line?

Most real corrections fail the first one. That is the test working. A correction
is evidence that something went wrong in **this run**; a lesson is a claim that
it will go wrong again in a way reading the code would not prevent. Those are
different claims, and only the second one is worth paying for forever.

The failed ones are not discarded. They stay in the logs, where they cost
nothing and remain available if the same shape turns up enough times to change
the verdict.

## The budget is the feature

Every context file has a hard line ceiling. At the ceiling, adding a line means
removing one, and `/improve` has to name the line it is evicting and say why the
new line beats it. If nothing existing is weaker, it fails and asks you.

The failure is not a rough edge to be smoothed out later. It is the only moment
in the system where someone is made to say which of two constraints actually
matters. Without a ceiling every addition is an append, ranking never happens,
and pruning waits for a cleanup nobody schedules.

Which is why `/improve` may not raise a budget itself, even though it knows the
number and the file is right there. Raising a ceiling is a human decision by
definition — the entire meaning of the act is that someone looked at the trade
and chose to pay more. A tool that raises its own ceiling when the ceiling is
inconvenient has not made that decision, it has deleted it, at exactly the moment
the question was about to be asked.

The **number** is a different thing from the **mechanism**, and only the number is
negotiable. The shipped budgets are guesses about a repo anvil has never seen, and
a project raising one in `BUDGETS.md` is making an informed decision, not failing.
A budget that only ever goes up is the signal worth acting on: it means lessons
have stopped being distilled and started being accumulated, and the fix is
upstream in how they are written.

The same reasoning is why every context-file change is a recommendation rather
than an action. The mechanism was always there to force a person to rank two
constraints; confining that to the moment a file happens to be full was the
arbitrary part.

## Losing lines is the point

Two halves, and they have opposite rules.

**The logs are permanent.** `feedback/human.jsonl` and `feedback/self.jsonl` are
append-only and have no budget. They are the evidence base, and throwing evidence
away is how a system stops being able to learn. Anything ever evicted can be
found, re-argued and re-promoted from the log it was written to.

**The context files are lossy on purpose.** They are a cache over the logs, held
small enough to be worth reading on every run. A line leaves when something beats
it, when the same constraint turns up in your `CLAUDE.md`, or when it has gone
quiet long enough that you agree to drop it.

"Nothing is lost" is true of the logs and false of the context files, and
collapsing the two is how you end up with either a landfill or a system that
forgets.

## Nothing is seeded

`process/*.md` ship containing a title line and nothing else, and the temptation
to prefill them is strong and wrong.

"Slice so one ticket is one PR a human can review in one sitting" is good advice.
It is exactly the kind of line that belongs there. But until this repo has
actually produced an oversized PR, it is anvil assuming rather than anvil
knowing — and nothing in the file distinguishes a shipped guess from a line that
was earned through two independent sightings.

The asymmetry decides it. An empty file costs nothing and misleads nobody. A
wrong seeded line costs budget **and** gets believed. The cost of waiting is a few
tickets; the cost of guessing wrong is a line that shapes every run of that
command until someone notices.

The same reasoning governs `/setup`: it seeds from what you tell it, never from
what it infers. A half-guessed config file is worse than a missing one, because
missing is a clean state and half-populated is a file nobody authored that
everyone trusts.

## One writer, and it asks

Every command writes feedback entries. Exactly one command — `/improve` — edits
context files, and it edits nothing the user has not approved in that run.

Those are two separate protections against two separate failures. One writer
stops the files filling from several directions at once. Asking first stops the
one writer filling them with plausible lines nobody agreed to — the same
landfill, arriving politely and by a single door.

`/feedback` is the other half of that split: it puts entries into the logs and
touches no context file, so capture stays cheap and frequent while distillation
stays deliberate and rare.

`/review` is the interesting exclusion, because it produces the most opinionated
output in the system and has the best case for an exception. It does not get one.
Its judgment lands in the logs like everything else and reaches the context files
only by clearing the same threshold as everything else. Two writers to one file
is how the file becomes a landfill, and the second writer always arrives with a
good reason.

## What would make this wrong

The claim is falsifiable and it is worth naming the test up front: **if no line
in a context file ever changes a later run's behaviour in a way you can point
at, the loop is decorative.**

Not "hard to measure" — decorative. A compounding-feedback diagram is easy to
draw and easy to believe, and the failure mode is a system that faithfully logs,
routes and promotes lessons that were never going to change anything. Every
component here carries a kill criterion for the same reason; see the end of
`ROADMAP.md`.

---

## Further reading

- [FILE_CONTRACT.md](FILE_CONTRACT.md) — the authority. Every file, its budget,
  its exact format
- [ROADMAP.md](ROADMAP.md) — the milestones, and the risk each one retires
- [ORIGIN_IDEA.md](ORIGIN_IDEA.md) — the unedited note this was built from, and
  where the design has since diverged from it
