# Requirements-Driven Development (RDD)

A structured methodology for LLM-assisted software development that front-loads understanding before any code is written.

## The Problems

LLM-assisted development fails not because the LLM can't code, but because it codes the wrong thing. A vague description becomes fast but wrong code. Without a mechanism to verify intent before implementation, small misunderstandings compound into large structural problems.

Equally, systems that pass all tests at launch can still drift, degrade, or behave incorrectly over time. Code that didn't change can break because the environment around it changed — a dependency updated, a data shape shifted, a configuration drifted, an integration contract was quietly violated. Tests verify construction under controlled conditions. Evaluations verify ongoing behavior against the original specification under real conditions. Every system needs evals; the type and complexity of evals scales with the system's non-determinism. AI-assisted systems need richer evals because their outputs are probabilistic. Deterministic systems need simpler evals — often scheduled contract checks or synthetic monitors — but they need them too.

A third problem is **dark code**: AI-generated codebases that work but that no human fully understands. When AI writes code at speed, the humans responsible for that code can no longer answer basic questions about it — why was this dependency called here? why is caching structured this way? how was separation of concerns reasoned about? This is *epistemic debt*: not bugs, not performance problems, but a loss of understanding that accumulates silently and becomes a liability at the worst possible moments. RDD addresses this by requiring that code comprehension — the reasoning behind structural decisions — be specified before implementation begins and embedded into the codebase as it is written.

## The Core Idea

**Understand first. Build second. Evaluate continuously.**

Every stage produces documentation specific enough to constrain the next stage. No code is written until intent is clearly communicated, requirements are validated, architecture is chosen with tradeoffs understood, and every interaction is specified down to exact inputs, outputs, and error paths.

The specification is the single source of truth for three parallel downstream concerns: **implementation** (via TDD), **evaluations** (via an eval plan derived from the same spec), and **code transparency** (via comprehension requirements embedded into every module). Evals are not an afterthought — they are defined alongside the spec and run throughout the system's lifetime. Code transparency is not documentation added after the fact — it is a requirement collected from the human in Stages 1–2, cemented in Stages 3–4, and implemented as a structural layer in Stages 5–6.

RDD is **not linear**. Each stage may surface contradictions, gaps, or new understanding that sends the process back upstream. This is expected and encouraged — it is cheaper to revise a document than to revise code.

## The Six Stages

```
┌─────────────────────────────────────────────────────────┐
│  Stage 1: Discovery & Scope                             │
│  Who, why, what matters — and what "correct" means      │
│  → Scope Doc + Assumptions Register (incl. eval intent, │
│    code transparency intent)                            │
├─────────────────────────────────────────────────────────┤
│  Stage 2: Environment & Constraints                     │
│  Where, under what rules — including eval constraints   │
│  and legibility requirements                            │
│  → Technical Context Document (incl. eval constraints,  │
│    Code Comprehension Requirements)                     │
├─────────────────────────────────────────────────────────┤
│  Stage 3: Architecture Proposal                         │
│  How it's structured — including eval harness design    │
│  and comprehension standard                             │
│  → Architecture Decision Document (incl. eval arch,     │
│    Comprehension Standard)                              │
├─────────────────────────────────────────────────────────┤
│  Stage 4: Actor-System Interactions                     │
│  Exactly what happens + how each interaction is         │
│  evaluated + structural reasoning captured              │
│  → Actor-System Interaction Doc + Decision Log          │
│    (incl. eval metadata, design rationale)              │
├─────────────────────────────────────────────────────────┤
│  Stage 5: Implementation & Eval Plan                    │
│  Build roadmap + eval suite + execution schedule        │
│  + comprehension layer specification                    │
│  → Implementation .md Files + 05-eval-plan.md           │
│  → Traceability Matrix (story→interaction→test→eval)    │
├─────────────────────────────────────────────────────────┤
│  Stage 6: Incremental TDD + Eval Execution              │
│  Build, test, evaluate, checkpoint, repeat              │
│  Every module ships with manifest + semantic context    │
│  → Working, tested, evaluated, legible, deployed code   │
│  → Post-launch: evals continue per Stage 5 schedule     │
└─────────────────────────────────────────────────────────┘
```

---

### Stage 1: Discovery & Scope

The LLM interviews the user to understand who the system serves and why it exists. It extracts personas, maps their journeys (happy paths and failure scenarios), and classifies each journey as must-have (P0), should-have (P1), or deferred (P2) — all during the same interview, not as a separate step.

The LLM is instructed to push back, challenge assumptions, ask about what the user didn't mention, and surface contradictions. Scope is established through a minimum viable scope (MVS) definition and a complexity budget.

At this stage, **eval intent** is also established. Every system needs evals — the question is what kind and how complex. The LLM interviews the human to determine: What does "correct behavior" mean to the business over time, not just at launch? Are there SLA obligations, performance contracts, or integration agreements that must be verified continuously? Does this system have non-deterministic or AI-assisted components that make output quality unpredictable over time? These answers determine the nature of the eval plan: deterministic systems typically need scheduled contract checks and synthetic monitors (binary, low cost); AI-assisted systems need scored evals with LLM judges and sampling strategies (probabilistic, higher cost). In both cases, evals are defined and planned — they do not get invented post-launch by an ops team.

**Code transparency intent** is also captured here: who are the humans responsible for this code, and what is their relationship to the codebase? Are they the original authors or inheritors? Are they expected to read every module or audit at the interface level? What does a responsible engineer need to understand about a module without reading its implementation? These questions determine the depth and granularity of comprehension requirements that will be carried forward into Stages 2–6.

**Deliverables:**
- Opinionated (must haves):
  - **01-overview.md** — the human-readable record of system intent, scope, personas, priorities, and validated assumptions. Written for humans to review, baseline, and version. This is the Stage 1 review gate artifact.
  - **agent.md / Claude.md** — the agent-facing instruction file. Always produced alongside 01-overview.md. Contains a compressed summary of intent and scope from 01-overview.md (the parts that affect coding decisions), plus all coding conventions, project structure, tooling, constraints, and rules of engagement for working in this codebase. Written for the agent; consulted on every interaction throughout Stages 5–6. The two files are complementary, not interchangeable: 01-overview.md is the record of what was decided and why; agent.md is the standing instruction set for how to act on it.
  - **02-user-stories.md** — the shared vocabulary of the system, written for humans and the agent equally. Answers: who are the actors and personas? What does each actor need to accomplish? What is the priority of each story (P0/P1/P2)? What are the acceptance criteria? What is explicitly out of scope and why? Every story here must be traceable to at least one interaction in 04-actor-system-interactions.md — this bidirectional traceability is enforced at the Stage 4 review gate. Stories are the top of the traceability chain: story → interaction → test → eval → comprehension artifact.
- Dynamically Determined (optionals): persona profiles, journey maps, prioritized feature list (P0/P1/P2), MVS, scope risks, complexity budget, assumptions register
- **[Eval]** Eval intent captured in Assumptions Register: what "correct over time" means for this system, what classes of behavioral failure are unacceptable, and whether the system is deterministic (simpler evals) or non-deterministic/AI-assisted (richer evals) — this determines eval plan complexity, not whether an eval plan exists
- **[Transparency]** Code transparency intent captured in Assumptions Register: who owns the code, what level of legibility is required, and what questions a responsible engineer must be able to answer without reading the full implementation

**Review gate:** Structured checklist of assumptions to confirm or deny. No forward progress until resolved.

---

### Stage 2: Environment & Constraints

The LLM gathers technical context through existing documentation, structured interview, reference project analysis, or a combination of all three. It covers deployment environment, tech stack, team capabilities, performance/security requirements, CI/CD, and budget constraints.

Evals are a system with their own resource costs and operational constraints. These must be defined here — before architecture — so that the eval harness is designed within real limits, not ideal ones.

**Code Comprehension Requirements** are also defined at this stage. These are the legibility standards the human is establishing for the codebase before any code is written. The LLM interviews the human to establish:
- What must a developer understand about a module without reading its implementation?
- What standards apply to documenting dependencies — both what a module calls (outbound) and what calls it (inbound)?
- What behavioral contracts are mandatory at every interface: performance expectations, failure modes, retry semantics?
- Are there existing internal standards (documentation format, tooling, naming conventions) this codebase must conform to?
- What telemetry or observability tooling will consume this metadata, and what format does it expect?

These requirements become a named section in the Technical Context Document and flow directly into the Comprehension Standard defined in Stage 3.

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
- **[Transparency]** Code Comprehension Requirements section within Technical Context: legibility standards, dependency documentation rules, mandatory interface contracts, internal standards to conform to, tooling/telemetry format requirements

**Review gate:** The LLM surfaces conflicts between constraints and scoped requirements — including eval constraints vs. eval intent from Stage 1, and legibility requirements vs. team capabilities — before they become implementation surprises.

---

### Stage 3: Architecture Proposal

The LLM proposes 2–3 architectural options (not just one), each with tradeoffs, risk profile, and mapping to P0 journeys. It recommends one but presents alternatives for informed user choice.

Evals cannot be an afterthought in architecture. An architecture that makes system outputs hard to capture, trace, or replay is a liability for eval continuity. Each architectural option must be evaluated for how well it supports the eval harness.

The architecture must also define the **Comprehension Standard** — the uniform structure that every module in the codebase will follow for transparency. This is an architectural decision, not a per-developer choice, because it must be consistent across the entire system to be useful. The Comprehension Standard defines three layers, derived from the Code Comprehension Requirements in Stage 2:

**1. Module Manifest** — structural visibility. Every module must declare:
- What this module does (single-responsibility statement)
- What it depends on (outbound): services, libraries, data stores it calls
- What depends on it (inbound): which modules or actors call this one
- Which user story and actor-system interaction it implements (traceability back to Stage 4)

**2. Semantic Context** — behavioral visibility. Every interface must declare its rules-of-engagement, not just its data shape:
- Performance expectations (latency targets, throughput limits)
- Failure modes (what can go wrong, expected error types)
- Retry semantics (is this safe to retry? idempotent? what backoff applies?)
- Behavioral contracts (preconditions, postconditions, invariants)
- Caching rationale (if applicable: what is cached, where, why, and what can and cannot read it)
- Separation of concerns reasoning (why this boundary exists here and not elsewhere)

**3. Decision Residual** — anything structural not covered above:
- Why this pattern was chosen over alternatives considered
- What the agent was instructed to optimize for
- Known limitations or future evolution points

The architecture should also define how these comprehension artifacts are consumed: read by humans directly in the code, indexed by a tool, fed to an agent for onboarding, or surfaced in a dashboard. The answer affects format requirements.

**Eval architecture decisions to make:**
- Does the architecture need an eval harness as a named component? (Separate service, shared library, or third-party platform such as Braintrust, LangSmith, or Promptfoo?)
- How does the system expose itself to evals? (Logging hooks, trace IDs, output capture points)
- Where do eval results go? (Monitoring dashboards, deployment gates, alerting pipelines)
- Does the eval architecture differ per environment? (Lightweight in development, full suite in staging, traffic sampling in production)

The tradeoff analysis for each architectural option should explicitly note how easy or hard it makes ongoing eval execution.

**Deliverable:**
- Opinionated (must have): **03-architecture.md** — written primarily for humans, but consulted by the agent throughout Stages 5–6 to keep implementation decisions consistent with the chosen architecture. A single document that incorporates both Stage 2 (constraints and environment) and Stage 3 (architectural decisions), because constraints and architecture are inseparable — the constraints inform and bound the choices. Answers: what technical constraints and environment must this system operate within? What architectural options were considered and what are their tradeoffs? Which architecture was chosen and why? What does the system look like structurally — components, boundaries, data flows, dependencies? What are the performance, security, and scalability decisions? What is the domain glossary? What is the eval harness design? What is the Comprehension Standard for all modules?
- **[Eval]** Eval Architecture section within 03-architecture.md: harness design, system integration points, data flow from output to eval input, result routing, environment differences
- **[Transparency]** Comprehension Standard section within 03-architecture.md: mandatory structure for Module Manifest, Semantic Context, and Decision Residual; consumption format; tooling integration
- Dynamically Determined (optional): Architectural Options, Tradeoff Analysis, journey-to-architecture mapping, Risks, Domain Glossary, Data Flows, Data Designs, Constraint Catalogue, Dependency Graph, Test Infrastructure, Test Cases

**Review gate:** The LLM notes which P0/P1/P2 features the chosen architecture makes easy or hard to add later, and which architectural choice best satisfies eval constraints from Stage 2. Detect and resolve conflicts between architecture, constraints (Stage 2), and scoped requirements (Stage 1) before they become implementation surprises.

---

### Stage 4: Actor-System Interaction Specification

For each persona journey, the LLM defines: trigger, preconditions, exact inputs (field names, types, validation), system behavior, exact outputs, postconditions, and every error path. Edge cases are explicitly addressed: empty/null inputs, concurrent access, partial failures, authorization boundaries, rate limiting.

This stage is also the **primary source of eval definitions** and the place where **structural reasoning is cemented**. Every actor-system interaction has defined inputs and expected outputs — each becomes an eval case. And every interaction carries the design rationale that will flow into the Semantic Context and Decision Residual of the modules that implement it. The questions that need to be answerable by a responsible engineer — *why did you call that dependency here? why is caching structured this way? how was separation of concerns reasoned about?* — are answered here, in the spec, before any code is written.

**Eval metadata per interaction:**
- **Eval type**: deterministic (binary pass/fail) or scored (LLM judge, human review, statistical comparison)
- **Trigger**: when is this interaction evaluated — per-commit, nightly, on production traffic sampling, on-demand?
- **Threshold**: what score or result is acceptable?
- **Deployment gate priority**: is a regression here a hard deployment blocker, a warning, or a logged alert?

**Deliverable:**
- Opinionated (must have): **04-actor-system-interactions.md** — the most operationally dense artifact in RDD and the most critical review gate. Written for humans to sign off and for the agent as the primary generation source for tests, evals, and code. Answers, for every persona journey: what triggers this interaction? What are the preconditions? What are the exact inputs — field names, types, validation rules? What does the system do in response? What are the exact outputs? What are the postconditions? What happens in every error path? What are the edge cases — null inputs, concurrent access, partial failures, rate limiting, authorization boundaries? What eval type, trigger, threshold, and deployment gate priority applies? What is the design rationale — why this dependency, why this boundary, why this caching approach? Everything downstream — TDD tests, eval cases, module Semantic Context, and Decision Residuals — is generated directly from this file.
- **[Eval]** Eval metadata fields (type, trigger, threshold, priority) on every interaction
- **[Transparency]** Design rationale captured per interaction: dependency choices, boundary decisions, caching rationale, separation of concerns reasoning — this becomes the source material for Semantic Context and Decision Residual in Stage 6
- Dynamically Determined (optional): Decision Log, Detailed Specs, End-to-End Tests with Assertions

**Review gate:**
- Every actor-system interaction is correlated to 1 or more user stories && every user story participates in at least 1 actor-system interaction
- **[Eval]** Every interaction has an eval type, trigger, and threshold defined before advancing
- **[Transparency]** Every interaction has design rationale sufficient to answer: why this dependency, why this boundary, why this caching or concurrency approach

---

### Stage 5: Implementation & Eval Plan

All artifacts are saved into `.md` files in the `.rdd` folder.
If delivery needs to be phased (MVP, must-haves, should-haves, nice-to-haves), the LLM generates phase-specific artifacts in phase-specific folders (`.rdd-MVP`, `.rdd-must-have`, `.rdd-should-have`, `.rdd-nice-to-have`).

Stage 5 now has three parallel tracks that share a traceability matrix. The traceability matrix gains two new columns: every user story traces to → actor-system interaction → TDD test(s) → eval case(s) → module comprehension artifact(s).

**5a. Implementation Plan**

Following are **opinionated** (mandatory) artifacts:
- **01-overview.md** — human-readable record of system intent, scope, personas, priorities, and validated assumptions. Answers: why does this system exist, who does it serve, what is in/out of scope, what was assumed and validated? (produced in Stage 1, referenced throughout)
- **CLAUDE.md / agent.md** — agent-facing instruction file. Answers: what are the coding rules, conventions, project structure, tooling, constraints, and escalation triggers for this codebase? Contains a compressed summary of intent and scope from 01-overview.md plus all rules of engagement. (produced in Stage 1, updated as conventions evolve)
- **02-user-stories.md** — shared vocabulary of the system for humans and agent alike. Answers: who are the actors, what does each need to accomplish, what is the priority and acceptance criteria of each story, what is explicitly out of scope? Top of the traceability chain. (produced in Stage 1)
- **03-architecture.md** — structural blueprint of the system, incorporating both environment constraints (Stage 2) and architectural decisions (Stage 3). Answers: what constraints govern this system, what architecture was chosen and why, what does the system look like structurally, what is the domain glossary, what is the eval harness design, what is the Comprehension Standard? (produced in Stages 2–3, consulted by agent throughout Stages 5–6)
- **04-actor-system-interactions.md** — precise behavioral specification for every persona journey. Answers: what exactly happens at every step, what are the exact inputs and outputs, what are all error paths and edge cases, what eval metadata applies, what is the design rationale for every structural decision? Primary generation source for TDD tests, eval cases, and module comprehension artifacts. (produced in Stage 4, the most critical review gate)

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

**5c. Comprehension Layer Specification**

The Comprehension Standard defined in Stage 3 is now translated into explicit generation instructions for Stage 6. For every module to be built, the LLM is given:
- The Module Manifest template (what to declare, required fields, traceability links back to Stage 4)
- The Semantic Context template (required interface annotations: performance, failure modes, retry, contracts, caching, separation of concerns)
- The Decision Residual template (pattern rationale, optimization targets, known limitations)
- The consumption format (inline code annotations, sidecar `.md` files, or structured metadata — as defined in Stage 3)

This ensures the comprehension layer is generated consistently, not improvised per module.

---

### Stage 6: Incremental TDD + Eval Execution

For each increment (phase): write failing tests first, implement minimum code to pass, refactor, run the full suite, then checkpoint — the LLM reports results and waits for user review before proceeding. If the LLM hits an ambiguity, it stops and asks rather than guessing. Any user-refinements from this phase are logged and, once the phase is marked as complete, are propagated back to appropriate `.md` documents.

The execution loop now includes eval verification and comprehension layer generation at each checkpoint:

1. Write failing TDD tests (from Stage 4 interaction specs)
2. Implement minimum code to pass
3. Generate Module Manifest, Semantic Context, and Decision Residual for each new module (from Stage 5c templates and Stage 4 design rationale)
4. Refactor
5. Run full TDD suite
6. **[Eval]** Run eval suite for this increment
7. **[Eval]** Verify eval scores meet thresholds defined in Stage 4
8. **[Eval]** Log eval results against cost/token budget from Stage 2
9. **[Transparency]** Verify every module in this increment has a complete Manifest, Semantic Context, and Decision Residual — a missing or thin comprehension artifact is treated the same as a failing test: the increment is not complete
10. Checkpoint — LLM reports TDD results, eval results, and comprehension coverage before proceeding
11. User reviews all three before advancing to the next increment

**Post-launch:** The eval schedule defined in Stage 5b takes over. Evals that ran during development continue running in production at the cadences and triggers defined in the spec. The spec contract is enforced continuously — not just at the moment the code ships. The comprehension layer is permanent: it travels with the code and is updated whenever a module is modified, ensuring the codebase remains legible to humans and agents over its full lifetime.

**Deliverable:** Working, tested, evaluated, and legible code deployed according to the plan.

---

## Feedback Loops

Every stage can trigger a return to an earlier stage. This is not a failure — it's the process working as intended.

```
Stage 2 (Environment)        ──→ Stage 1  when constraints make features infeasible
                                          or eval constraints conflict with eval intent
                                          or legibility requirements exceed team capability
Stage 3 (Architecture)       ──→ Stage 1  when tradeoffs require re-prioritization
                                          or eval harness design reveals scope gaps
Stage 4 (Interactions)       ──→ Stage 1  when a journey is underspecified or too complex
Stage 5 (Plan)               ──→ Stage 3  when planning reveals an architectural gap
                                          or eval cost model exceeds Stage 2 constraints
                                          or comprehension layer format needs revision
-- for Stage 6 (execution), determine if the issue causes major architecture change that
-- would require user-approval, or if the coding/eval issue causes user-story change or
-- major actor-system-interaction change:
-- if no, then following triggers are executed post-execution stage (in batch)
-- if yes, then propagate back immediately
Stage 6 (Execution)          ──→ Stage 4  when ambiguity is found during coding or eval
                                          or design rationale is missing for a structural decision
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

Retrofit mode also applies to code transparency: the LLM audits the existing codebase for dark code — modules without manifests, interfaces without behavioral contracts, structural decisions without documented rationale — and generates a backlog of comprehension artifacts to be added. The human reviews and validates the LLM's inferences about design intent before they are committed as the baseline.

---

## Relationship to Spec-Driven Development (SDD)

RDD and the emerging practice of Spec-Driven Development (SDD) share the same foundational insight: specifications must *drive* development, not merely document it. The specification is the source of truth — code is its expression. Both methodologies were motivated by the same problem: AI coding agents produce code that works but codes the wrong thing when given vague instructions, and the solution is to front-load precision rather than attempt to recover it after the fact.

The convergence is not coincidental. The conditions that made SDD necessary — AI agents capable of generating large amounts of code quickly, rising system complexity, and accelerating pace of change — are the same conditions that motivated RDD. Neither methodology is derived from the other; they arrived at the same core principle independently.

Where they differ is in scope and depth. SDD, as practiced by tools like GitHub Spec Kit and Kiro, defines the philosophy and a lightweight four-phase workflow (Specify → Plan → Tasks → Implement) optimized for individual developers and small teams working with AI coding agents. It is deliberately accessible and tool-first. RDD operationalizes the same philosophy for enterprise-scale development, where the stakes of a wrong architectural decision are higher, the number of stakeholders is larger, and the system must remain correct and understandable long after the original team has moved on. Specifically, RDD extends SDD with:

- **Structured eval planning** — continuous behavioral verification derived from the same spec that drives implementation, applicable to every system regardless of whether it uses AI components
- **Code transparency requirements** — comprehension embedded into every module at write time, eliminating epistemic debt in AI-generated codebases
- **Actor-System Interaction Specification** — a more rigorous specification layer that defines exact inputs, outputs, error paths, and behavioral contracts for every persona journey, becoming the source for both TDD tests and eval cases
- **Multiple architectural options** — Stage 3 requires 2–3 options with explicit tradeoffs rather than a single recommendation, ensuring informed human choice over AI-defaulted decisions
- **Phased delivery support** — explicit support for MVP, must-have, should-have, and nice-to-have delivery phases with phase-specific artifact sets
- **Continuous scope management** — P0/P1/P2 classification and complexity budget enforced at every stage, not just at the start
- **Retrofit mode** — applying the methodology in reverse to existing codebases, extracting implicit requirements and establishing artifact baselines for systems that predate RDD

For teams already familiar with SDD, RDD is a compatible and more rigorous elaboration of the same philosophy. The core vocabulary maps directly: what SDD calls a specification, RDD calls a Discovery & Scope Document and Actor-System Interaction Spec. What SDD calls a plan, RDD calls Stages 3–5. What SDD calls implementation, RDD calls Stage 6 — with the addition of evals and the comprehension layer running alongside.



| Stage | Deliverable | Answers | Key Contents |
|-------|-------------|---------|--------------|
| 1. Discovery & Scope | **01-overview.md** | WHY — intent, scope, assumptions | Purpose, personas, journeys, priorities, MVS, **eval intent**, **code transparency intent** |
| 1. Discovery & Scope | **CLAUDE.md / agent.md** | HOW TO WORK HERE — coding rules, conventions, constraints | Compressed intent summary + project structure, tooling, rules of engagement, escalation triggers |
| 1. Discovery & Scope | **02-user-stories.md** | WHO + WHAT — actors and discrete units of value | Personas, stories, acceptance criteria, P0/P1/P2 priorities, explicit out-of-scope |
| 2–3. Environment & Architecture | **03-architecture.md** | WHERE + HOW — environment, structure, decisions | Stack, constraints, architectural options and tradeoffs, chosen architecture, domain glossary, **eval harness design**, **Comprehension Standard** |
| 4. Interactions | **04-actor-system-interactions.md** | EXACTLY WHAT — precise behavior, every path, every edge case | Exact inputs/outputs, error paths, edge cases, postconditions, **eval metadata**, **design rationale** |
| 5b. Eval Plan | **05-eval-plan.md** | HOW TO VERIFY CONTINUOUSLY | Eval suite, execution schedule, cost model, post-launch continuation |
| 5c. Comprehension Layer | Part of Implementation Plan | HOW TO STAY LEGIBLE | Module Manifest, Semantic Context, Decision Residual generation instructions per module |
| 5. Traceability | Traceability Matrix | WHAT CONNECTS TO WHAT | User story → interaction → TDD test → eval case → comprehension artifact |
| 6. Execution | Working, tested, evaluated, legible code | THE SYSTEM ITSELF | Incremental, checkpoint-verified, traceable, eval results verified at each checkpoint, comprehension artifacts verified at each checkpoint, post-launch evals continue per Stage 5 schedule |
