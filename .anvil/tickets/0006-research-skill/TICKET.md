# Problem

Parked entries are evidence about where the routing table is wrong, and that
evidence is invisible to anvil's author. Nothing turns the pile into a question
aimed outside the repo.

# Scope

skills/research/SKILL.md. Reads unrouted.md and both feedback logs. Emits one
research question citing the fb-NNNN entries behind it, says what an answer
would change, and names both destinations — run it yourself, or open an issue
upstream. Scoped now against a real pile; built at M7.

# Out of scope

Writing any file, including unrouted.md. Any submission endpoint or telemetry.
Acting on the answer.

# Acceptance criteria

1. `git status --short` is clean after a run — no file is written at all.
2. The prompt cites at least one fb-NNNN id and states what an answer would
   change.
3. The run opens no context file.
4. Against an empty unrouted.md, it says there is no evidence to draw on and
   stops rather than inventing a question.
