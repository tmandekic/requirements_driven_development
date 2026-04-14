# Requirements-Driven Development (RDD)

A structured methodology for LLM-assisted software development that front-loads understanding before any code is written.

## The Problem

LLM-assisted development fails not because the LLM can't code, but because it codes the wrong thing. A vague description becomes fast but wrong code. Without a mechanism to verify intent before implementation, small misunderstandings compound into large structural problems.

Equally, systems that pass all tests at launch can drift, degrade, or behave incorrectly over time — especially systems with non-deterministic or AI-assisted components. Tests verify construction. Evaluations verify ongoing behavior against the original specification.

## The Core Idea

**Understand first. Build second. Verify continuously.**

Every stage produces documentation specific enough to constrain the next stage. No code is written until intent is clearly communicated, requirements are validated, architecture is chosen with tradeoffs understood, and every interaction is specified down to exact inputs, outputs, and error paths.

The specification is the single source of truth for two parallel downstream concerns: **implementation** (via TDD) and **evaluations** (via an eval plan derived from the same spec). Evals are not an afterthought — they are defined alongside the spec and run throughout the system's lifetime.

RDD is **not linear**. Each stage may surface contradictions, gaps, or new understanding that sends the process back upstream. This is expected and encouraged — it is cheaper to revise a document than to revise code.

## The Six Stages

```
┌─────────────────────────────────────────────────────────┐
│  Stage 1: Discovery & Scope                             │
│  Who, why, what matters — and what "correct" means      │
│  → Scope Doc + Assumptions Register (incl. eval intent) │
├─────────────────────────────────────────────────────────┤
│  Stage 2: Environment & Constraints                     │
│  Where, under what rules — including eval constraints   │
│  → Technical Context Document (incl. eval constraints)  │
├─────────────────────────────────────────────────────────┤
│  Stage 3: Architecture Proposal                         │
│  How it's structured — including eval harness design    │
│  → Architecture Decision Document (incl. eval arch)     │
├─────────────────────────────────────────────────────────┤
│  Stage 4: Actor-System Interactions                     │
│  Exactly what happens + how each interaction is         │
│  evaluated (type, trigger, threshold, priority)         │
│  → Actor-System Interaction Doc + Decision Log          │
├─────────────────────────────────────────────────────────┤
│  Stage 5: Implementation & Eval Plan                    │
│  Build roadmap + eval suite + execution schedule        │
│  → Implementation .md Files + 05-eval-plan.md           │
│  → Traceability Matrix (story→interaction→test→eval)    │
├─────────────────────────────────────────────────────────┤
│  Stage 6: Incremental TDD + Eval Execution              │
│  Build, test, evaluate, checkpoint, repeat              │
│  → Working, tested, evaluated, deployed code            │
│  → Post-launch: evals continue per Stage 5 schedule     │
└─────────────────────────────────────────────────────────┘
```

---

### Stage 1: Discovery & Scope

The LLM interviews the user to understand who the system serves and why it exists. It extracts personas, maps their journeys (happy paths and failure scenarios), and classifies each journey as must-have (P0), should-have (P1), or deferred (P2) — all during the same interview, not as a separate step.

The LLM is instructed to push back, challenge assumptions, ask about what the user didn't mention, and surface contradictions. Scope is established through a minimum viable scope (MVS) definition and a complexity budget.

At this stage, **eval intent** is also established: does this system have non-deterministic or AI-assisted components? What does "correct behavior" mean to the business over time, not just at launch? Are there regulatory, compliance, or SLA obligations that require continuous behavioral verification? These answers determine whether evals are lightweight quality checks or a first-class system concern, and they are captured in the Assumptions Register before architecture begins.

**Deliverables:**
- Opinionated (must haves): Intention & Scope, User Stories (01-overview.md or CLAUDE.md, 02-user-stories.md)
- Dynamically Determined (optionals): persona profiles, journey maps, prioritized feature list (P0/P1/P2), MVS, scope risks, complexity budget, assumptions register
- **[Eval]** Eval intent captured in Assumptions Register: why continuous verification matters for this system, and what classes of behavioral failure are unacceptable

**Review gate:** Structured checklist of assumptions to confirm or deny. No forward progress until resolved.

---

### Stage 2: Environment & Constraints

The LLM gathers technical context through existing documentation, structured interview, reference project analysis, or a combination of all three. It covers deployment environment, tech stack, team capabilities, performance/security requirements, CI/CD, and budget constraints.

Evals are a system with their own resource costs and operational constraints. These must be defined here — before architecture — so that the eval harness is designed within real limits, not ideal ones.

**Eval constraints to define:**
- **Spend limits**: monthly budget and per-run cost ceiling for eval executions (especially if evals invoke an LLM judge)
- **Token budgets**: max tokens per eval call and per eval suite run
- **Latency/processing constraints**: can evals block a deployment pipeline (synchronous gate) or must they be asynchronous?
- **Data constraints**: can evals use production data, or synthetic only? PII and data residency rules?
- **Frequency constraints**: how often can evals run — per commit, nightly, on-demand, continuous sampling?
- **Infrastructure constraints**: do evals run in the same environment as the application, or in isolation?
- **Threshold definitions**: what score constitutes pass vs. fail? Who sets thresholds? Who can override them?

**Deliverable:**
- Opinionated (must have): Technical Context Information (included in 03-architecture.md)
- **[Eval]** Eval Constraints section within Technical Context: spend limits, token budgets, data rules, frequency, thresholds, infrastructure isolation requirements

**Review gate:** The LLM surfaces conflicts between constraints and scoped requirements — including eval constraints vs. eval intent from Stage 1 — before they become implementation surprises.

---

### Stage 3: Architecture Proposal

The LLM proposes 2–3 architectural options (not just one), each with tradeoffs, risk profile, and mapping to P0 journeys. It recommends one but presents alternatives for informed user choice.

Evals cannot be an afterthought in architecture. An architecture that makes system outputs hard to capture, trace, or replay is a liability for eval continuity. Each architectural option must be evaluated for how well it supports the eval harness.

**Eval architecture decisions to make:**
- Does the architecture need an eval harness as a named component? (Separate service, shared library, or third-party platform such as Braintrust, LangSmith, or Promptfoo?)
- How does the system expose itself to evals? (Logging hooks, trace IDs, output capture points)
- Where do eval results go? (Monitoring dashboards, deployment gates, alerting pipelines)
- Does the eval architecture differ per environment? (Lightweight in development, full suite in staging, traffic sampling in production)

The tradeoff analysis for each architectural option should explicitly note how easy or hard it makes ongoing eval execution.

**Deliverable:**
- Opinionated (must have): Architecture Decisions (included in 03-architecture.md)
- **[Eval]** Eval Architecture section within Architecture Decision Document: harness design, system integration points, data flow from output to eval input, result routing, environment differences
- Dynamically Determined (optional): Architectural Options, Tradeoff Analysis, journey-to-architecture mapping, Risks, Domain Glossary, Data Flows, Data Designs, Constraint Catalogue, Dependency Graph, Test Infrastructure, Test Cases

**Review gate:** The LLM notes which P0/P1/P2 features the chosen architecture makes easy or hard to add later, and which architectural choice best satisfies eval constraints from Stage 2. Detect and resolve conflicts between architecture, constraints (Stage 2), and scoped requirements (Stage 1) before they become implementation surprises.

---

### Stage 4: Actor-System Interaction Specification

For each persona journey, the LLM defines: trigger, preconditions, exact inputs (field names, types, validation), system behavior, exact outputs, postconditions, and every error path. Edge cases are explicitly addressed: empty/null inputs, concurrent access, partial failures, authorization boundaries, rate limiting.

This stage is also the **primary source of eval definitions**. Every actor-system interaction has defined inputs and expected outputs — each of those becomes an eval case. The "exactly what happens at every step" definition is, by definition, the eval criteria. Eval metadata is defined here, attached to each interaction, not in a separate document.

**Eval metadata per interaction:**
- **Eval type**: deterministic (binary pass/fail) or scored (LLM judge, human review, statistical comparison)
- **Trigger**: when is this interaction evaluated — per-commit, nightly, on production traffic sampling, on-demand?
- **Threshold**: what score or result is acceptable?
- **Deployment gate priority**: is a regression here a hard deployment blocker, a warning, or a logged alert?

**Deliverable:**
- Opinionated (must have): Actor-System Interactions with eval metadata per interaction; this is the most critical review gate — both TDD tests and eval cases are generated directly from this information
- **[Eval]** Eval metadata fields (type, trigger, threshold, priority) on every interaction
- Dynamically Determined (optional): Decision Log, Detailed Specs, End-to-End Tests with Assertions

**Review gate:**
- Every actor-system interaction is correlated to 1 or more user stories && every user story participates in at least 1 actor-system interaction
- **[Eval]** Every interaction has an eval type, trigger, and threshold defined before advancing

---

### Stage 5: Implementation & Eval Plan

All artifacts are saved into `.md` files in the `.rdd` folder.
If delivery needs to be phased (MVP, must-haves, should-haves, nice-to-haves), the LLM generates phase-specific artifacts in phase-specific folders (`.rdd-MVP`, `.rdd-must-have`, `.rdd-should-have`, `.rdd-nice-to-have`).

Stage 5 now has two parallel tracks that share a traceability matrix. The traceability matrix gains a new column: every user story traces to → actor-system interaction → TDD test(s) → eval case(s).

**5a. Implementation Plan**

Following are **opinionated** (mandatory) artifacts:
- 01-overview.md (or CLAUDE.md)
- 02-user-stories.md
- 03-architecture.md
- 04-actor-system-interactions.md

All other artifacts are dynamically generated based on the purpose and scope of the system.

**5b. Eval Plan**

The following is a new **opinionated** (mandatory) artifact:
- **05-eval-plan.md**

Contents of 05-eval-plan.md:
- **Eval suite definition**: eval cases derived from Stage 4 interactions and their eval metadata
- **Eval harness setup**: infrastructure, tooling, data pipeline (synthetic vs. production, PII handling)
- **Execution schedule**: trigger mechanisms per eval case, per environment (development, staging, production)
- **Result routing**: dashboards, deployment gates, alert thresholds, escalation paths
- **Cost model**: projected spend per run and per month, validated against Stage 2 eval constraints
- **Post-launch continuation**: which evals run continuously after deployment, at what cadence, and who owns threshold reviews over time

---

### Stage 6: Incremental TDD + Eval Execution

For each increment (phase): write failing tests first, implement minimum code to pass, refactor, run the full suite, then checkpoint — the LLM reports results and waits for user review before proceeding. If the LLM hits an ambiguity, it stops and asks rather than guessing. Any user-refinements from this phase are logged and, once the phase is marked as complete, are propagated back to appropriate `.md` documents.

The execution loop now includes eval verification at each checkpoint:

1. Write failing TDD tests (from Stage 4 interaction specs)
2. Implement minimum code to pass
3. Refactor
4. Run full TDD suite
5. **[Eval]** Run eval suite for this increment
6. **[Eval]** Verify eval scores meet thresholds defined in Stage 4
7. **[Eval]** Log eval results against cost/token budget from Stage 2
8. Checkpoint — LLM reports TDD results **and** eval results before proceeding
9. User reviews both before advancing to the next increment

**Post-launch:** The eval schedule defined in Stage 5b takes over. Evals that ran during development continue running in production at the cadences and triggers defined in the spec. The spec contract is enforced continuously — not just at the moment the code ships.

**Deliverable:** Working, tested, and evaluated code deployed according to the plan.

---

## Feedback Loops

Every stage can trigger a return to an earlier stage. This is not a failure — it's the process working as intended.

```
Stage 2 (Environment)        ──→ Stage 1  when constraints make features infeasible
                                          or eval constraints conflict with eval intent
Stage 3 (Architecture)       ──→ Stage 1  when tradeoffs require re-prioritization
                                          or eval harness design reveals scope gaps
Stage 4 (Interactions)       ──→ Stage 1  when a journey is underspecified or too complex
Stage 5 (Plan)               ──→ Stage 3  when planning reveals an architectural gap
                                          or eval cost model exceeds Stage 2 constraints
-- for Stage 6 (execution), determine if the issue causes major architecture change that
-- would require user-approval, or if the coding/eval issue causes user-story change or
-- major actor-system-interaction change:
-- if no, then following triggers are executed post-execution stage (in batch)
-- if yes, then propagate back immediately
Stage 6 (Execution)          ──→ Stage 4  when ambiguity is found during coding or eval
Stage 6 (Execution)          ──→ Stage 3  when an architectural limitation is discovered
Stage 6 (Execution)          ──→ Stage 1  when a new feature is requested mid-build
```

When looping back, the LLM names the stage and reason, makes the minimum necessary change, propagates it forward through all subsequent deliverables, and gets user validation before resuming.

---

## Continuous Scope Management

Scope is managed at every stage, not just Stage 1:

- Every new idea or requirement is classified P0/P1/P2 before action.
- The LLM is authorized and expected to identify scope creep and propose deferral.
- The complexity budget is checked at every execution checkpoint.
- If cumulative P0 additions exceed ~20% since plan finalization, a full re-planning cycle is recommended.
- **[Eval]** Eval cost is tracked against the Stage 2 budget at every checkpoint. If cumulative eval spend is trending over budget, the eval plan is revised before the next increment.

---

## Review Gate Protocol

Every stage transition follows the same structure:

1. The LLM presents the deliverable.
2. The LLM generates a **structured review prompt** — specific questions, assumption checklists, or decision logs. Never just "does this look right?"
3. The user responds to each item.
4. Unresolved items are addressed before advancing.
5. The validated deliverable becomes the tracked baseline with version annotations for future changes.

---

## Retrofit Mode

For existing projects, RDD can be applied in reverse — extracting implicit requirements, personas, and explicit architecture from the codebase before establishing the artifact baselines. The LLM audits the code and interviews the user to validate what it found, and establishes "as-is" behavior.

Retrofit mode also applies to evals: the LLM audits existing test coverage to identify which behaviors are verified at build time but not monitored at runtime, and proposes an eval plan to close that gap.

---

## Artifacts Summary

| Stage | Deliverable | Key Contents |
|-------|-------------|--------------|
| 1. Discovery & Scope | 01-overview.md, 02-user-stories.md, Assumptions Register | Purpose, personas, journeys, user stories, priorities, MVS, **eval intent** |
| 2. Environment | Technical Context in 03-architecture.md | Stack, constraints, deployment specs, technical risks, **eval constraints (spend, tokens, data rules, frequency, thresholds)** |
| 3. Architecture | Architecture Decision in 03-architecture.md | Options, tradeoffs, recommendation, glossary, **eval harness design, integration points, result routing** |
| 4. Interactions | 04-actor-system-interactions.md | Input/output specs, error paths, decision log, **eval metadata per interaction (type, trigger, threshold, priority)** |
| 5. Implementation & Eval Plan | Implementation .md files + **05-eval-plan.md** | Foundation, architecture, constraints, tests, sequence, deployment, **eval suite, execution schedule, cost model, post-launch continuation** |
| 5. Traceability | Traceability Matrix | User story → interaction → TDD test → **eval case** |
| 6. Execution | Working, tested, evaluated code | Incremental, checkpoint-verified, traceable, **eval results verified at each checkpoint; post-launch evals continue per Stage 5 schedule** |
