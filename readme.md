# Requirements-Driven Development (RDD)

A structured methodology for LLM-assisted software development that front-loads understanding before any code is written.

## The Problem

LLM-assisted development fails not because the LLM can't code, but because it codes the wrong thing. A vague description becomes fast but wrong code. Without a mechanism to verify intent before implementation, small misunderstandings compound into large structural problems.

## The Core Idea

**Understand first. Build second.**

Every stage produces a document specific enough to constrain the next stage. No code is written until requirements are validated, architecture is chosen with tradeoffs understood, and every interaction is specified down to exact inputs, outputs, and error paths.

RDD is **not linear**. Each stage may surface contradictions, gaps, or new understanding that sends the process back upstream. This is expected and encouraged — it is cheaper to revise a document than to revise code.

## The Six Stages

```
┌─────────────────────────────────────────────────────────┐
│  Stage 1: Discovery & Scope                             │
│  Who, why, and what matters most                        │
│  → Discovery & Scope Document + Assumptions Register    │
├─────────────────────────────────────────────────────────┤
│  Stage 2: Environment & Constraints                     │
│  Where and under what rules                             │
│  → Technical Context Document                           │
├─────────────────────────────────────────────────────────┤
│  Stage 3: Architecture Proposal                         │
│  How it's structured (2–3 options with tradeoffs)       │
│  → Architecture Decision Document                       │
├─────────────────────────────────────────────────────────┤
│  Stage 4: Actor-System Interactions                     │
│  Exactly what happens at every step                     │
│  → Actor-System Interaction Doc + Decision Log          │
├─────────────────────────────────────────────────────────┤
│  Stage 5: Implementation Plan                           │
│  The build roadmap                                      │
│  → 6 Implementation .md Files + Traceability Matrix     │
├─────────────────────────────────────────────────────────┤
│  Stage 6: Incremental TDD Execution                     │
│  Build, test, checkpoint, repeat                        │
│  → Working, tested, deployed code                       │
└─────────────────────────────────────────────────────────┘
```

### Stage 1: Discovery & Scope

The LLM interviews the user to understand who the system serves and why it exists. It extracts personas, maps their journeys (happy paths and failure scenarios), and classifies each journey as P0 (must-have), P1 (should-have), or P2 (deferred) — all during the same interview, not as a separate step.

The LLM is instructed to push back, challenge assumptions, ask about what the user didn't mention, and surface contradictions. Scope is established through a minimum viable scope (MVS) definition and a complexity budget.

**Deliverable:** Discovery & Scope Document containing purpose statement, persona profiles, journey maps, prioritized feature list (P0/P1/P2), MVS, scope risks, complexity budget, and an Assumptions Register requiring explicit user validation.

**Review gate:** Structured checklist of assumptions to confirm or deny. No forward progress until resolved.

### Stage 2: Environment & Constraints

The LLM gathers technical context through existing documentation, structured interview, reference project analysis, or a combination of all three. It covers deployment environment, tech stack, team capabilities, performance/security requirements, CI/CD, and budget constraints.

**Deliverable:** Technical Context Document.

**Review gate:** The LLM surfaces conflicts between constraints and scoped requirements before they become implementation surprises.

### Stage 3: Architecture Proposal

The LLM proposes 2–3 architectural options (not just one), each with tradeoffs, risk profile, and mapping to P0 journeys. It recommends one but presents alternatives for informed user choice.

**Deliverable:** Architecture Decision Document with options, tradeoff analysis, journey-to-architecture mapping, risks, and glossary.

**Review gate:** The LLM notes which P1/P2 features the chosen architecture makes easy or hard to add later.

### Stage 4: Actor-System Interaction Specification

For each P0 journey, the LLM defines: trigger, preconditions, exact inputs (field names, types, validation), system behavior, exact outputs, postconditions, and every error path. Edge cases are explicitly addressed: empty/null inputs, concurrent access, partial failures, authorization boundaries, rate limiting.

**Deliverable:** Actor-System Interaction Document + Decision Log. This is the most critical review gate — tests and implementation are generated directly from this document.

**Review gate:** Every design decision in the Decision Log requires explicit confirmation.

### Stage 5: Implementation Plan

The LLM synthesizes all prior artifacts into six `.md` files:

| File | Contents |
|------|----------|
| `01-technical-foundation.md` | Pinned versions, project structure, config, dependencies |
| `02-architecture.md` | Components, data models, API contracts, state management |
| `03-constraints-and-reference.md` | Non-functional requirements, glossary, dependency graph, conventions |
| `04-test-infrastructure.md` | Testing framework, categories, data strategy, coverage, CI pipeline |
| `05-implementation-sequence.md` | Ordered increments with TDD, acceptance criteria, checkpoints |
| `06-deployment.md` | Setup, deployment steps, rollback, smoke tests, monitoring |

**Review gate:** Traceability Matrix mapping each increment → interaction → journey → persona.

### Stage 6: Incremental TDD Execution

For each increment: write failing tests first, implement minimum code to pass, refactor, run the full suite, then checkpoint — the LLM reports results and waits for user review before proceeding. If the LLM hits an ambiguity, it stops and asks rather than guessing.

**Deliverable:** Working, tested code deployed according to the plan.

## Feedback Loops

Every stage can trigger a return to an earlier stage. This is not a failure — it's the process working as intended.

```
Stage 2 (Environment)        ──→ Stage 1  when constraints make features infeasible
Stage 3 (Architecture)       ──→ Stage 1  when tradeoffs require re-prioritization
Stage 4 (Interactions)       ──→ Stage 1  when a journey is underspecified or too complex
Stage 5 (Plan)               ──→ Stage 3  when planning reveals an architectural gap
Stage 6 (Execution)          ──→ Stage 4  when ambiguity is found during coding
Stage 6 (Execution)          ──→ Stage 3  when an architectural limitation is discovered
Stage 6 (Execution)          ──→ Stage 1  when a new feature is requested mid-build
```

When looping back, the LLM names the stage and reason, makes the minimum necessary change, propagates it forward through all subsequent deliverables, and gets user validation before resuming.

## Continuous Scope Management

Scope is managed at every stage, not just Stage 1:

- Every new idea or requirement is classified P0/P1/P2 before action.
- The LLM is authorized and expected to identify scope creep and propose deferral.
- The complexity budget is checked at every execution checkpoint.
- If cumulative P0 additions exceed ~20% since plan finalization, a full re-planning cycle is recommended.

## Review Gate Protocol

Every stage transition follows the same structure:

1. The LLM presents the deliverable.
2. The LLM generates a **structured review prompt** — specific questions, assumption checklists, or decision logs. Never just "does this look right?"
3. The user responds to each item.
4. Unresolved items are addressed before advancing.
5. The validated deliverable becomes the tracked baseline with version annotations for future changes.

## Retrofit Mode

For existing projects, RDD can be applied in reverse — extracting implicit requirements, personas, and architecture from the codebase before establishing the artifact baselines. The LLM audits the code, interviews the user to validate what it found, and distinguishes "as-is" behavior from "to-be" requirements. A characterization test suite locks in P0 behavior before any changes are proposed.

## Getting Started

Copy the contents of [`rdd-system-prompt.md`](rdd-system-prompt.md) into the system prompt (or first user message) when starting a new project with an LLM. The prompt contains the full stage definitions, interview discipline rules, feedback loop map, scope management protocol, and review gate standards.

## Artifacts Summary

| Stage | Deliverable | Key Contents |
|-------|-------------|--------------|
| 1. Discovery & Scope | Discovery & Scope Document | Purpose, personas, journeys, priorities, MVS, assumptions register |
| 2. Environment | Technical Context Document | Stack, constraints, deployment specs, technical risks |
| 3. Architecture | Architecture Decision Document | Options, tradeoffs, recommendation, glossary |
| 4. Interactions | Actor-System Interaction Document | Input/output specs, error paths, decision log |
| 5. Plan | Implementation Plan (.md files) | Foundation, architecture, constraints, tests, sequence, deployment |
| 6. Execution | Working tested code | Incremental, checkpoint-verified, traceable |
