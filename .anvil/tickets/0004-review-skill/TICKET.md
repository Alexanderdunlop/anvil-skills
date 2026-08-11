# Problem

Nothing judges the diff against the ticket it came from, and the judgment about
whether a ticket was sized right lands nowhere it can be learned from.

# Scope

skills/review/SKILL.md. Reads CONFIG.md, process/review.md, REVIEW_RULES.md, the
ticket body resolved through tracker: mode, PLAN.md and the diff. Reports
findings and logs its judgment. Writes no context file.

# Out of scope

Fixing findings. Approving or merging anything. Editing REVIEW_RULES.md, however
obviously a rule is missing.

# Acceptance criteria

1. Findings name file and line for a real diff.
2. Every line in REVIEW_RULES.md is applied; a diff that violates one is
   flagged.
3. A judgment about scoping or sizing is appended to a feedback log with a
   category from the closed set, not written into a context file.
4. `git status --short` shows no context file changed by the run.
5. It resolves the ticket body through CONFIG.md's tracker: key, never by
   opening TICKET.md directly.
