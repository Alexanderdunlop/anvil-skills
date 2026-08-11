# Problem

PLAN.md is a plan nobody executes. It is also unproven that a build command
earns its place at all rather than being a wrapper around "read the plan".

# Scope

skills/build/SKILL.md. Starts cold from the ticket body, CONTEXT.md, PLAN.md and
TEST_CASES.md. Writes code and nothing else. Logs every stall to self.jsonl,
routed to the command that should change rather than the one that noticed.

# Out of scope

Writing CONTEXT.md, PLAN.md or TEST_CASES.md. Verifying its own work. Editing
any context file.

# Acceptance criteria

1. With no conversation history and the three plan files alone, it implements
   the plan and `git status --short` shows code changes only.
2. Every stall appends a self.jsonl entry carrying `correction: null`, a
   fingerprint, and a proposed_line naming what the plan should have said.
3. The entry says what unblocked it, not only that it was blocked.
4. At least one build-sourced entry carries `category: "process:kickoff"`.
5. After three tickets, entries with `command: "build"` exist in the logs. If
   none do, the skill is deleted and PLAN.md becomes the interface.
