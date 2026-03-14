# Requirements-Driven Development (RDD)

A structured methodology for LLM-assisted software development that front-loads understanding before any code is written.

## The Problem

LLM-assisted development fails not because the LLM can't code, but because it codes the wrong thing. A vague description becomes fast but wrong code. Without a mechanism to verify intent before implementation, small misunderstandings compound into large structural problems.

## The Core Idea

**Understand first. Build second.**

Every stage produces documentation specific enough to constrain the next stage. No code is written until intent is clearly communicated, requirements are validated, architecture is chosen with tradeoffs understood, and every interaction is specified down to exact inputs, outputs, and error paths.

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

The LLM interviews the user to understand who the system serves and why it exists. It extracts personas, maps their journeys (happy paths and failure scenarios), and classifies each journey as must-have (P0), should-have (P1), or deferred (P2) — all during the same interview, not as a separate step.

The LLM is instructed to push back, challenge assumptions, ask about what the user didn't mention, and surface contradictions. Scope is established through a minimum viable scope (MVS) definition and a complexity budget.

**Deliverables:** 
Opinionated (must haves): Intention & Scope, User Stories (01-overview.md or CLAUDE.md, 02-user-stories.md)
Dynamically Determined (optionals): persona profiles, journey maps, prioritized feature list (P0/P1/P2), MVS, scope risks, complexity budget, assumptions register

**Review gate:** Structured checklist of assumptions to confirm or deny. No forward progress until resolved.

### Stage 2: Environment & Constraints

The LLM gathers technical context through existing documentation, structured interview, reference project analysis, or a combination of all three. It covers deployment environment, tech stack, team capabilities, performance/security requirements, CI/CD, and budget constraints.

**Deliverable:** 
- Opinionated (must have): Technical Context Information (included in 03-architecture.md)

**Review gate:** The LLM surfaces conflicts between constraints and scoped requirements before they become implementation surprises.

### Stage 3: Architecture Proposal

The LLM proposes 2–3 architectural options (not just one), each with tradeoffs, risk profile, and mapping to P0 journeys. It recommends one but presents alternatives for informed user choice.

**Deliverable:** 
- Opinionated (must have): Architecture Decisions (included in 03-architecture.md)
- Dynamically Determined (optional):
    - Architecture could also include considered Architectural Options, Tradeoff Analysis, journey-to-architecture mapping, Risks, Domain Glossary, Data Flows, Data Designs, Constraint Catalogue, Dependency Graph, Test Infrastructure, Test Cases

**Review gate:** The LLM notes which P0/P1/P2 features the chosen architecture makes easy or hard to add later.
- Detect and resolve conflicts between architecture, constraints (stage 2) and scoped requirements (stage 1) before they become implementation surprises.

### Stage 4: Actor-System Interaction Specification

For each persona journey, the LLM defines: trigger, preconditions, exact inputs (field names, types, validation), system behavior, exact outputs, postconditions, and every error path. Edge cases are explicitly addressed: empty/null inputs, concurrent access, partial failures, authorization boundaries, rate limiting.

**Deliverable:** 
  - Opinionated (must have): Actor-System Interactions; this is the most critical review gate — tests are generated directly from this information.
  - Dynamically Determined (optional): Decision Log, Detailed Specs, End-to-End Tests with Assertions

**Review gate:** Every actor-system interaction is correlated to 1 or more user stories && every user story participates in at least 1 actor-user interaction

### Stage 5: Implementation Plan

All artifacts are saved into `.md` files in .rdd folder. 
If delivery needs to be phased (MVP, must-haves, should-haves, nice-to-haves), LLM can generate phase-specific set of artifacts and save them into phase-specific .rdd folder (.rdd-MVP, .rdd-must-have, .rdd-should-have, .rdd-nice-to-have).
Following are **opinionated** (mandatory) artifacts, and are prefixed with 00-based prefix: 
- 01-overview.md (or CLAUDE.md)
- 02-user-stories.md
- 03-architecture.md
- 04-actor-system-interactions.md

All other artifacts are dynamically generated based on purpose and scope of the system, 

### Stage 6: Incremental (Phased) TDD Implementation

For each increment (phase): write failing tests first, implement minimum code to pass, refactor, run the full suite, then checkpoint — the LLM reports results and waits for user review before proceeding. If the LLM hits an ambiguity, it stops and asks rather than guessing. Any user-refinements from this phase are logged and, once the phase is marked as complete, are propagated back to appropriate .md documents.

**Deliverable:** Working, tested code deployed according to the plan.

## Feedback Loops

Every stage can trigger a return to an earlier stage. This is not a failure — it's the process working as intended.

```
Stage 2 (Environment)        ──→ Stage 1  when constraints make features infeasible
Stage 3 (Architecture)       ──→ Stage 1  when tradeoffs require re-prioritization
Stage 4 (Interactions)       ──→ Stage 1  when a journey is underspecified or too complex
Stage 5 (Plan)               ──→ Stage 3  when planning reveals an architectural gap
--for Stage 6 (execution), determine if the issue causes major architecture change that would require user-approval, or if the coding issue causes user-story change or major actor-system-interaction change:
-- if no, then following triggers are executed post-execution stage (in batch)
-- if yes, then propagate back immediatelly 
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

For existing projects, RDD can be applied in reverse — extracting implicit requirements, personas, and explicit architecture from the codebase before establishing the artifact baselines. The LLM audits the code and interviews the user to validate what it found, and establishes "as-is" behavior.


## Artifacts Summary

| Stage | Deliverable | Key Contents |
|-------|-------------|--------------|
| 1. Discovery & Scope | Discovery & Scope Document: 01-overview.md (or CLAUDE.md), User Stories: 02-user-stories.md | Purpose, personas, journeys, user stories, priorities, MVS, assumptions register |
| 2. Environment | Technical Context in 03-architecture.md | Stack, constraints, deployment specs, technical risks |
| 3. Architecture | Architecture Decision in 03-architecture.md | Options, tradeoffs, recommendation, glossary |
| 4. Interactions | Actor-System Interaction: 04-actor-system-interactions.md | Input/output specs, error paths, decision log |
| 5. Plan | Implementation Plan (.md files) in single or phased .rdd directories | Foundation, architecture, constraints, tests, sequence, deployment |
| 6. Execution | Working tested code | Incremental, checkpoint-verified, traceable |
