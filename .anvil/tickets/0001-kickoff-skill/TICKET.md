# Problem

/scope produces a ticket body and stops. Nothing turns it into files a cold
session can build from, so /build has no input and every implementation starts
by re-deriving what the ticket already decided.

# Scope

skills/kickoff/SKILL.md. Reads CONFIG.md, process/kickoff.md and the ticket body
resolved through tracker: mode. Produces CONTEXT.md, PLAN.md and TEST_CASES.md
for one ticket, behind an approval gate. Appends to both feedback logs before
exit.

# Out of scope

Writing code. Editing any context file. Seeding process/kickoff.md. Running
tests.

# Acceptance criteria

1. In a repo with no .anvil/, it says to run /setup and `git status --short` is
   clean.
2. Against one ticket, three files appear under .anvil/tickets/<id>/ and `wc -l`
   shows CONTEXT ≤ 40, PLAN ≤ 60, TEST_CASES ≤ 40.
3. The plan is shown in the conversation first; replying "sure, I guess" leaves
   `git status --short` clean.
4. Under an external tracker with a reachable integration, no TICKET.md is read
   or written.
5. Every stall appends a self.jsonl entry with `correction: null` and a kebab
   fingerprint before the command exits.
6. The run completes having opened only CONFIG.md, process/kickoff.md and the
   ticket body.
