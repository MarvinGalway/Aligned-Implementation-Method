# INIT — Planning File Generator

This file contains a one-shot instruction for bootstrapping the `.ai/` planning directory.
Execute this **once**, at project kickoff, after Step M1 (Requirements Interview) is complete.

---

## Trigger Condition
Run this when:
- A new project is starting, OR
- A significant new feature set is being added to an existing project
- The Acceptance Criteria list from Step M1 is finalized

---

## Instructions

Read the following inputs before generating any files:
1. **Requirements file** — locate it using the detection sequence in `AGENT.md` (frontmatter tag `aim-role: requirements` takes priority, then convention fallback: `REQUIREMENTS.md` → `SPECS.md` → `PRD.md` → `RFC.md` → `docs/requirements*.md`). If not found, stop and ask.
2. `AGENT.md` — AIM: Aligned Implementation Method (workflow, phases, and rules)
3. The finalized Acceptance Criteria produced at the end of Step M1

Then execute the following:

### 1. Create the `.ai/` directory
Add `.ai/` to `.gitignore` if not already present.

### 2. Generate `.ai/TODO.md`

Use this structure:

```markdown
# Project TODO
_Last updated: [date]_

## Environment
- chub: [available / unavailable]

## Acceptance Criteria
- [ ] AC-01: [full text]
- [ ] AC-02: [full text]
...

## Phase 1 — [Title]
_Goal: [one sentence — what working software exists at the end of this phase]_
- [ ] 1 · Step 0 — Sentinel
- [ ] 1 · Step 1 — Blueprint: [entities and contracts being defined]
- [ ] 1 · Step 2 — Red Loop: [test being written]
- [ ] 1 · Step 3 — Green Loop: [implementation target]
- [ ] 1 · Step 4 — Wire: [transport connection, if applicable]
- [ ] 1 · Step 5 — Validation

## Phase 2 — [Title]
_Goal: [one sentence]_
- [ ] 2 · Step 0 — Sentinel
...
```

### 3. Generate `.ai/phase-N.md` for each phase

Use this structure:

```markdown
# Phase N — [Title]

## Goal
[What working software exists at the end of this phase]

## Acceptance Criteria Covered
- AC-XX: [text]
- AC-XX: [text]

## Steps
- [ ] Step 0 — Sentinel
- [ ] Step 1 — Blueprint
- [ ] Step 2 — Red Loop
- [ ] Step 3 — Green Loop
- [ ] Step 4 — Wire
- [ ] Step 5 — Validation

## Notes
[Any constraints, dependencies on previous phases, or deferred decisions]

## Status
- [ ] Not started

## Validation Result
[To be filled in during Step 5]
```

---

## Update Rules (enforced throughout development)

- **After each step completes:** check off the corresponding item in `.ai/TODO.md` and in `.ai/phase-N.md`.
- **After a phase completes:** update the `## Status` field in `.ai/phase-N.md` to `[x] Complete` and write the `## Validation Result`.
- **If a gap report is produced in Step 5:** append it under `## Validation Result` in `.ai/phase-N.md`.
- **Never delete or rewrite past entries.** Append corrections or resolutions below the original entry.
- **The `.ai/TODO.md` is the single source of progress truth.** It must always reflect the current real state of the project.