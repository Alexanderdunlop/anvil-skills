# Problem

Nothing checks the work against TEST_CASES.md and CRITICAL_PATHS.md, so
"verified" is a claim a reviewer has to take on trust.

# Scope

skills/verify/SKILL.md. Reads CONFIG.md, process/verify.md, REVIEW_RULES.md,
CRITICAL_PATHS.md and TEST_CASES.md. Writes VERIFY.md, one line per case, and
degrades to a manual checklist when CONFIG.md says smoke: none.

# Out of scope

Fixing what it finds. Editing context files. Touching the repo's PR template —
that is an open question, not this ticket.

# Acceptance criteria

1. .anvil/tickets/<id>/VERIFY.md exists with one line per case and no paragraph
   anywhere in it.
2. Every line's evidence is a command and what it returned; "works correctly"
   appears nowhere in the file.
3. With smoke: none, browser cases read COULD NOT CHECK, carry the manual steps,
   and the run still completes.
4. A failing case is recorded FAIL and the run continues to the remaining cases.
5. Re-running overwrites VERIFY.md rather than appending to it.
6. `grep -rn VERIFY.md skills/` returns hits only in verify's own SKILL.md.
