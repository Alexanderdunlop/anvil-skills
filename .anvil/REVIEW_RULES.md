# Review rules

Flag any `.anvil/` path in a skill not written `${CLAUDE_PROJECT_DIR}/.anvil/`.
Flag a skill that opens a ticket body without resolving `tracker:` from CONFIG.md first.
Flag a skill that writes any file when `${CLAUDE_PROJECT_DIR}/.anvil/` is absent.
Flag any skill other than `/feedback` that edits a context file.
Flag a `process/*.md` line with no fingerprint at two sightings in the logs.
Flag a date stamp, promotion marker or citation count added to a context file.
Flag a `CONFIG.md` key naming a build, test, lint or entrypoint command.
Flag `/feedback` writing `BUDGETS.md` or `CLAUDE.md`.
Flag a number stated in two documents where neither points at the other.
