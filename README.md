# AIM — Aligned Implementation Method

> A disciplined methodology for AI-assisted software development that keeps every implementation traceable to its original intent.

---

## Prerequisites

AIM requires a `REQUIREMENTS.md` file at the project root before any session begins. This is not optional — it is the foundation the entire methodology builds on.

A strong `REQUIREMENTS.md` must cover:

- **Problem statement** — what is being built and what does success look like
- **Technology stack** — language, framework, database, transport layer, test runner, and any key libraries
- **Functional requirements** — what the system must do
- **Non-functional requirements** — performance, security, scalability, accessibility constraints
- **Integrations** — external services, APIs, auth providers
- **Edge cases and boundaries** — what inputs are invalid, what failure states must be handled

If the file is missing or too vague, the agent will stop and ask for it before proceeding. A weak requirements file produces weak Acceptance Criteria, which breaks every downstream step.

---



AI coding agents are powerful but drift. They hallucinate APIs, make silent assumptions, build things that pass tests but miss the point, and lose track of requirements across long sessions. The gap between what you asked for and what gets built grows with every step.

AIM is a methodology designed to close that gap — not by making the AI smarter, but by constraining its freedom at every critical decision point.

---

## Core Idea

AIM treats the alignment problem as an **entropy problem**: uncertainty accumulates unless you actively suppress it. It does this through four mechanisms:

- **Acceptance Criteria** as a numbered source of truth that never changes
- **Contracts before code** so the solution space is bounded before implementation begins
- **TDD as a verification gate** so intent is encoded as a test before any code exists
- **Structural observability** so the AI reads actual system state instead of guessing

Every phase ends with a validation step that maps the implementation back to the original requirements. If they don't match, the agent stops and reports exactly what is wrong and why.

---

## Files

| File | Purpose |
|---|---|
| `REQUIREMENTS.md` | **You write this.** The project requirements, stack definition, and constraints. AIM cannot start without it. |
| `AGENT.md` | The runtime protocol. Place at project root and load as your agent's system prompt or rules file. |
| `INIT.md` | One-shot bootstrap command. Run once at project kickoff to generate the `.ai/` planning directory. |

---

## How to Use

### 1. Write your requirements

Create `REQUIREMENTS.md` at the project root. Cover the problem, the full tech stack, functional and non-functional requirements, integrations, and known edge cases. The more precise this file is, the more accurate the Acceptance Criteria and phase plan will be.

### 2. Add AIM to your project

Copy `AGENT.md` and `INIT.md` to your project root.

### 3. Load as agent rules

Point your agent at `AGENT.md` using the appropriate mechanism for your tool:

| Tool | How to load |
|---|---|
| Claude Code | Rename or symlink to `CLAUDE.md` at project root |
| Cursor | Paste content into `.cursorrules` or reference in project settings |
| Windsurf | Paste content into `.windsurfrules` |
| Any agent with system prompt | Paste `AGENT.md` content as the system prompt |

### 4. Start a project

Tell your agent:
```
Follow AGENT.md. Start with Step M1 — Requirements Interview.
```

The agent will interview you about goals, stack, constraints, and edge cases before writing a single line of code.

### 5. Bootstrap planning files

Once requirements are finalized, tell your agent:
```
Follow INIT.md. Generate the .ai/ planning directory.
```

This produces `.ai/TODO.md` and one `.ai/phase-N.md` per phase, pre-populated with the Acceptance Criteria and step checklists.

### 6. Develop phase by phase

Each phase runs the full AIM cycle autonomously:

```
Sentinel → Blueprint → Red Loop → Green Loop → Wire → Validation
```

The agent checks off tasks after each step and writes a Gap Report if the final validation reveals any mismatch between implementation and requirements.

---

## The AIM Cycle (per phase)

```
Step 0  Sentinel       Review scope, confirm no ambiguities, stop and ask if needed
Step 1  Blueprint      Define entities, contracts, and interfaces — wire observability
Step 2  Red Loop       Write a failing test — do not proceed until it fails
Step 3  Green Loop     Implement, trace, read logs on every run — fix from reality
Step 4  Wire           Connect transport layer — read interceptor logs
Step 5  Validation     Map results to Acceptance Criteria — report gaps or mark complete
```

---

## Observability

AIM treats observability as structural, not reactive. Three components are wired at definition time, not added when something breaks:

| Level | Where | Pattern |
|---|---|---|
| Entity Dump | Domain / Persistence layer | Serializable mixin / toString override |
| Logic Tracer | Service / Business logic layer | Decorator / Annotation / HOF wrapper |
| Transport Interceptor | API / RPC / Message layer | Middleware / Filter on `X-Agent-Debug: true` |

The agent captures and reads stdout/stderr on every test run — not just failures.

---

## Planning Files

```
.ai/                  ← gitignored
├── TODO.md           ← master checklist, updated after every step
├── phase-1.md        ← detailed breakdown, validation result
├── phase-2.md
└── ...
```

These files are the single source of progress truth. The agent updates them as it works — checking off steps, appending gap reports, and recording validation outcomes. They are never rewritten, only appended.

---

## Recommended Tooling

AIM works without any external dependencies. The following tools are optional enhancements that strengthen specific parts of the methodology.

### Context Hub (chub)

The most common source of hallucination in AI-assisted development is invented or outdated API signatures. An agent writing contracts against a library it doesn't actually know produces code that looks correct but fails at runtime — often silently.

[chub](https://github.com/aisuite/chub) is a CLI that fetches current, accurate API documentation for external libraries directly into the agent's context, eliminating this failure mode at the point where AIM is most vulnerable: **Step 1 (Blueprint)**, when contracts and interfaces are defined.

```bash
npm install -g @aisuite/chub

chub search [tool]           # discover available documentation
chub get [tool]/[feature]    # fetch current docs into context
chub help                    # full command reference
```

Inside AIM, chub is used at three points:
- **Step M1** — verify the declared stack actually supports the capabilities assumed
- **Step 1** — confirm real API signatures before writing any contract or interface
- **Doubt Protocol** — consult chub before escalating an unknown library behavior to the user

If chub is available, the agent uses it automatically. If not, the standard Doubt Protocol applies.

---



**One step at a time.** The agent executes one step, presents the code produced, and waits for explicit approval before continuing. The programmer reviews each output before the next step begins — this is the primary mechanism for catching drift early.

**Post-step code review is mandatory.** After every step the agent analyzes what it just built: does it align with the Acceptance Criteria? Does it introduce scope creep or unintended side effects? This analysis is presented to the programmer as part of the step output, not skipped in the interest of speed.

**Doubt is a blocker, not a footnote.** Any ambiguity mid-implementation triggers a full stop and an explicit question. The agent never guesses.

**Phases are vertical slices, not layers.** Phase 1 is always the smallest possible thing that runs end-to-end. Complexity grows incrementally on a working foundation.

**The Gap Report is first-class output.** When implementation doesn't match requirements, the agent produces a structured report: what the code does, what the requirement says, the root cause, and either a concrete fix or an open question.

**Stack agnostic.** AIM uses patterns (DTOs, Decorators, Interceptors, Mixins) not language-specific implementations. It works with any stack.

---

## Contributing

Issues and pull requests are welcome. If you adapt AIM for a specific stack or tool and want to share the implementation patterns, feel free to open a PR.

---

## License

MIT
