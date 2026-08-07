# Origin Idea

The original note this repo was built from, written before any code existed.

It is reproduced **unedited**, including the parts that were later cut, the
open questions, and the naming that has since changed. An origin document that
gets tidied up to match the current design stops being evidence and becomes a
second, worse copy of the spec.

**This file is not authoritative.** `docs/FILE_CONTRACT.md` is. Read this to
understand why the project is shaped the way it is, and §"What changed" below
to see where the note and the current design diverge.

---

## The note

```
anvil-skills

A note with all SKILL.md context files is they will use context so we need to keep them lighter. Similar to CLAUDE.md (a question I often get asked is why not put this in CLAUDE.md, well that's simple the context is only relevant to the skill).

- /setup or configuration runs location setup or future updates where needed
    - What is it setting up, well the idea of this repo of skills, is in the name, skills that learn and build themselves, but it's generic enough to work with any repo and setup so that's why it needs to create a file to locate and run things faster.
    - Sets up ANVIL_SKILL.md (explains where docs live and where ticket system lives, design system like figma if there is one), asks where skill feedback should live. (Should it be a .anvil folder?)
    - Sets up ANVIL_SCOPE.md & ANVIL_SCOPE_FEEDBACK.md
    - CRITICAL_PATHS.md
    - The other thing I like but not sure other people will like is updating the pr-template that helps add a section where the skill run can explain it ran these smoke tests (or maybe that should be a comment or I'm not sure tbh).

Build so each skill, you run /clear after to save on cost.

- /scope (or spec or define) creating. You supply a rough ticket or idea or an already existing ticket. Look at ANVIL_SCOPE every time
    - Phase 1 gather information (lighter as kickoff goes in depth, this is more about setting up A/C for a ticket), ticket if it exists (checks ANVIL_SKIL), design if available (check ANVIL_SKILL), codebase orientation.
    - Phase 2 discuss (new personal version of grilling, as I have a lot of personal feedback)
    - Phase 3 (Slice it approval required)
        - Claude likes to one shot things so the idea is to make this into bite sized chunks for human review.
    - Phase 4 (draft ticket/feedback) this is where the human gives context and it updates ANVIL_SCOPE_FEEDBACK.md with a log of things the human has corrected.

Now thinking about it the feedback log should fill up whenever the skill finds it useful without going over the top. This should be for all skills.

- /kickoff (or plan) you supply a ticket look at ANVIL_KICKOFF (something for kickoff is wether you use work trees to implement) every time
    - Phase 0 get ticket from /scope
    - Phase 1 Gather implementation plan, actual files and components, look for ADRs, look at CRITICAL_PATHS.md
    - Phase 2 Information report
    - Phase 3 discuss (new personal version of grilling, as I have a lot of personal feedback)
    - Phase 4 filling in gaps
    - Phase 5 Docs approval requried
    - Phase 6 Create plans based on ANVIL_KICKOFF (default in that file, is a <ticket>/… CONTEXT.md PLAN.md TEST_CASES.md, should it make smoke tests here too?)

- /build (or implement) this is super light just points to all the files and looks at md for learnings, and then has an autonomous log where claude gets stuck a lot to help for future.

- /verify (and push) this is where claude does a lot (not sure of the order)
    - Code self review
        - Refactor comments and docs
        - Check a an anvil code rules file and CRITICAL_PATHS.md
        - Check test cases
        - Simplify
    - Verify own a/c and smoke tests with mcp (specific to a repo) if possible (needs to be setup).

- /review your own personal review and this is for giving a lot of feedback this is important because it can improve all skills overall, i.e this PR is too big well that's scoping and kickoff feedback.

- /feedback or /learning (now this is where we take all the feedback from all the files and think through what's worth improving or updating in the context files, this is the core of the package), recommend running this often at first then weekly or bi weekly after a while.

- /research skill create a prompt based on feedback or the overall skills and the overall repo this will create a prompt to improve this process as a meta, or the persons process, then recommend using parallel.ai or CLAUDE research, then if it applies to anvil-skills I recommend creating a ticket to self improve this whole flow. Or just your own flow is fine too.
```

---

## What survived unchanged

These were right first time and have not moved since:

- **Context is a tax.** The opening line of the note is still the opening
  principle of the contract. Everything about budgets, tiers and the admission
  test descends from it.
- **Why not CLAUDE.md.** The reasoning holds: the context is only relevant to
  the skill, so paying for it on every session is waste.
- **`/clear` between skills.** This is why every skill has to start cold from
  files alone, which is now a definition-of-done item in every milestone.
- **Human approval gates in `/scope` and `/kickoff`.** The note's reason —
  "Claude likes to one shot things" — is still the reason.
- **`/feedback` is the core of the package.** Named as such in the note,
  built second in the roadmap because of it.
- **`/build` is deliberately thin, and exists to record where Claude got
  stuck.** The idea survived; the separate file did not. Stalls are entries in
  `feedback/self.jsonl` rather than a per-ticket `STUCK.md`, so a stall that
  repeats across tickets can be fingerprinted and promoted like any other
  lesson — which a standalone file could never do.
- **Every skill contributes feedback**, not just `/scope`. The note reaches
  this mid-thought and it became a structural rule.

---

## What changed

| In the note                                             | Now                                                                           | Why                                                                                                                      |
| ------------------------------------------------------- | ----------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------ |
| `ANVIL_SKILL.md` at repo root                           | `.anvil/CONFIG.md`                                                            | Root prefixing couples every consuming repo to the package name; a rename becomes a migration for users. The file was briefly `MAP.md`, and the name invited repo facts — `test:`, `build:`, `entrypoint:` — that Claude can already infer. `CONFIG.md` holds only anvil's own config and pointers outside the repo |
| `ANVIL_SCOPE.md`, `ANVIL_KICKOFF.md`                    | `.anvil/process/<command>.md`                                                 | Same reason, plus one predictable path per command                                                                       |
| `ANVIL_SCOPE_FEEDBACK.md`, per-skill logs               | `.anvil/feedback/human.jsonl` + `self.jsonl`, one fingerprint space           | Per-*skill* logs are still rejected for the note's own reason: `/feedback` must spot the same lesson across different commands, and a `/build` stall routed to `/kickoff` is the case that matters. The one allowed split is by who noticed, so "show me what I actually corrected" is answerable with `cat`. `/feedback` counts sightings across both files |
| "Should it be a .anvil folder?"                         | Yes                                                                           | Answered. One dot-folder, no root clutter, trivially renamed                                                             |
| An "anvil code rules file"                              | `.anvil/REVIEW_RULES.md`, 30 lines, read by `/verify` and `/review` only       | The interesting one. The note asked for a *code* rules file; the design drifted into an always-read `RULES.md` paying for review guidance on every run of every skill, including `/scope`, which reviews nothing. That file was deleted — every line it held was already a `CLAUDE.md` fact. What remains is scoped to the two skills that read code, at a third of the cost, and is closer to what the note actually described than the intermediate design ever was |
| `CRITICAL_PATHS.md` at root                             | `.anvil/CRITICAL_PATHS.md`, absorbs testing knowledge                         | A separate `TESTING.md` would be read alongside it and double the cost for no gain                                       |
| Feedback log fills "whenever the skill finds it useful" | Fixed JSONL schema with a closed category set and promotion thresholds        | Freeform prose cannot be routed reliably; a lesson needs a destination or it is evidence that the routing table is wrong |
| `/review` improves all skills                           | `/review` writes feedback entries only; `/feedback` alone edits context files | One writer, one reader. Two writers to the same file is how it becomes a landfill                                        |
| `/research`                                             | Cut from v1                                                                   | It was `/feedback`'s output under another name. The narrow version needs `unrouted.md` to exist first. Revisit after M3  |
| `/setup` first                                          | `/setup` built last (M7)                                                      | It is a generator for a file format that has not stopped moving yet                                                      |

---

## Still open

Carried forward from the note, still undecided, parked until there is evidence:

- **PR template.** Whether `/verify` updates the repo's PR template with a
  smoke-test section, posts a comment, or does neither. The note's own
  uncertainty — "or I'm not sure tbh" — was correct. Touching a shared template
  is invasive for a tool that claims to work in any repo. Parked until M5.
- **Smoke tests in `/kickoff`.** Whether `/kickoff` generates them alongside
  `TEST_CASES.md`, or `/verify` derives them at check time.
- **Worktrees.** The note raises them as a `/kickoff` option. Unresolved, and it
  interacts with ticket id minting: `max(existing) + 1` collides across
  concurrent worktrees.
