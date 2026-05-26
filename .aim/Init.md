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

1. **Requirements file** — locate it using the detection sequence in `.aim/PROTOCOL.md` (frontmatter tag `aim-role: requirements` takes priority, then convention fallback: `REQUIREMENTS.md` → `SPECS.md` → `PRD.md` → `RFC.md` → `docs/requirements*.md`). If not found, stop and ask.
2. `.aim/PROTOCOL.md` — AIM: Aligned Implementation Method (workflow, phases, Metadata block spec, and rules)
3. The finalized Acceptance Criteria produced at the end of Step M1, including their mapping back to source Requirements

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
- gitnexus: [available / unavailable]
- graphify: [available / unavailable]

_Tooling availability is probed at Step M1 and recorded here. The agent uses available tools automatically at the points specified in .aim/PROTOCOL.md. Unavailable tools fall back to the manual paths described in the Doubt Protocol and Step 5 Validation._

## Requirements

- REQ-01: [full text]
- REQ-02: [full text]
...

## Acceptance Criteria

- [ ] AC-01: [full text]  → REQ-01
- [ ] AC-02: [full text]  → REQ-01, REQ-02
...

## Phase 1 — Phase_1_[SlugName]

_Goal: [one sentence — what working software exists at the end of this phase]_
_Covers: AC-01, AC-02_

- [ ] 1 · Step 0 — Sentinel
- [ ] 1 · Step 1 — Blueprint: [entities, contracts, and Metadata blocks being defined]
- [ ] 1 · Step 2 — Red Loop: [test being written]
- [ ] 1 · Step 3 — Green Loop: [implementation target]
- [ ] 1 · Step 4 — Wire: [transport connection, if applicable]
- [ ] 1 · Step 5 — Validation

## Phase 2 — Phase_2_[SlugName]

_Goal: [one sentence]_
_Covers: AC-03, AC-04_

- [ ] 2 · Step 0 — Sentinel
...
```

### 3. Generate `.ai/phase-N.md` for each phase

Use this structure:

```markdown
# Phase N — Phase_N_[SlugName]

## Goal

[What working software exists at the end of this phase]

## Acceptance Criteria Covered

- AC-XX: [text]  → REQ-YY
- AC-XX: [text]  → REQ-YY

## Phase Identifier

The literal string used in all Metadata blocks referencing this phase: `Phase_N_[SlugName]`.
This identifier is **locked** once development begins — renaming invalidates every Metadata block referencing it.

## Steps

- [ ] Step 0 — Sentinel
- [ ] Step 1 — Blueprint
- [ ] Step 2 — Red Loop
- [ ] Step 3 — Green Loop
- [ ] Step 4 — Wire
- [ ] Step 5 — Validation

## Sentinel Baseline

_Filled in at Step 0. Records pre-existing artifacts (if any) already tagged with this phase's ACs, surfaced via KG query when a KG tool is available._

## Notes

[Any constraints, dependencies on previous phases, or deferred decisions]

## Status

- [ ] Not started

## Validation Result

_Filled in during Step 5._

### Coverage Evidence

_For each AC covered by this phase, record: the implementations that satisfy it, the tests that prove it, and the detection method (KG query, test output, or manual review). This section is tool-neutral — the goal is auditable evidence, not tool-specific syntax._

- **AC-XX:** ✅ / ❌
  - Implementations: [list of function/class paths]
  - Tests: [list of test paths, all passing]
  - Detection method: [KG query result / direct test execution / manual review]

### Metadata Validation

_Confirms all Metadata blocks in this phase's artifacts reference identifiers that exist in `.ai/TODO.md`. Failed validation blocks phase completion._

- Phases referenced: [list, all resolved]
- Requirements referenced: [list, all resolved]
- ACs referenced: [list, all resolved]
- Contracts referenced: [list, all resolved]
- Tests referenced: [list, all resolved]

### Gap Report

_Empty if all criteria are met. Otherwise, structured per the format in .aim/PROTOCOL.md Step 5._
```

---

## Update Rules (enforced throughout development)

- **After each step completes:** check off the corresponding item in `.ai/TODO.md` and in `.ai/phase-N.md`.
- **After Step 0 (Sentinel):** record the Sentinel Baseline in `.ai/phase-N.md`.
- **After Step 5 (Validation):** populate the Coverage Evidence and Metadata Validation sections in `.ai/phase-N.md`. If gaps exist, append a Gap Report.
- **After a phase completes:** update the `## Status` field in `.ai/phase-N.md` to `[x] Complete`.
- **If a gap report is produced in Step 5:** append it under `### Gap Report` in `.ai/phase-N.md`.
- **Never delete or rewrite past entries.** Append corrections or resolutions below the original entry.
- **The `.ai/TODO.md` is the single source of progress truth.** It must always reflect the current real state of the project.
- **Phase identifiers (Phase_N_SlugName) are locked** once development begins. Renaming a phase requires migrating every Metadata block that references it, which is a deliberate breaking change, not a routine edit.