# Problem

Seven skills stop and ask for /setup, and nothing creates what they ask for. It
is built last because it generates the file format, and a generator written
against a moving format gets rewritten.

# Scope

skills/setup/SKILL.md. Asks the user questions and writes CONFIG.md,
CRITICAL_PATHS.md, REVIEW_RULES.md, five empty process/*.md, both feedback logs
and unrouted.md. Handles the update path over an existing .anvil/.

# Out of scope

Seeding process/*.md. Creating BUDGETS.md. Inferring config by reading the repo.

# Acceptance criteria

1. In a repo with no .anvil/, the full tree appears and every seeded line traces
   to an answer the user gave in the run.
2. All five process/*.md contain their title line and nothing else.
3. No BUDGETS.md is created.
4. CONFIG.md holds no build, test, lint or entrypoint key.
5. Re-run over an existing .anvil/, `git diff` shows no human-written line
   changed or removed.
6. In a repo that already has a CLAUDE.md, CONFIG.md gains nothing that restates
   it.
