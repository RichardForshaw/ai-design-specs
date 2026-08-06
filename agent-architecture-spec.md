<!--
Sync Impact Report
Version change: 1.0.0 -> 2.0.0 (restructured for project-agnosticism; added iteration-scoping and iterative test-design principles; genericized coverage principle)
-->

# Agent Architecture Baseline

Companion to `engineering-quality-baseline.md`. That file defines *what counts as done* (engineering quality); this file defines *who does what, and in what order* when Claude agents execute the work, from the point a specification and plan already exist through to a verified, deployed single increment. Both are Global-tier content, meant to be referenced (not copied) by a project's `constitution.md`. Where a principle needs a project-specific instantiation (which correctness principles apply, what the deploy mechanism is, how continuation is signaled), that value is recorded in the project's own constitution or spec, not here; this file stays invariant and project-agnostic.

## Core Principles

### A. Orchestrator Activation and Scope Derivation
The Orchestrator activates once a specification and plan already exist — it is not responsible for producing them, and does not run before them. Because a spec and plan may be broadly or loosely scoped, the Orchestrator MUST derive a candidate breakdown into independently-deliverable iterations before any implementation begins, and MUST agree that breakdown with the project owner before proceeding. Where the existing plan is too broad or ambiguous to scope confidently, the Orchestrator MAY perform a re-planning pass, in coordination with the project owner, before proposing iterations.
Rationale: beginning implementation against an under-scoped plan produces iterations nobody actually agreed to; deriving and confirming scope is cheaper than discovering the mismatch mid-build.

### B. Environment and Protocol Awareness

Every agent — Orchestrator, Test-Design, and Developer alike — MUST orient itself to the current project's actual environment before acting: its toolchain reference (e.g. a PROJECT_TOOLCHAIN file) and dependency manifests, and, where the project uses a spec-management framework (e.g. SpecKit), that framework's own file layout and conventions. Agents MUST follow the framework's protocols rather than assuming a generic or self-invented structure.
Rationale: remove unstated-convention risk and conform to local project requirements and guardrails.

### C. Fixed Three-Role Separation (NON-NEGOTIABLE)
Each iteration MUST be executed by exactly three separated roles:

 1. Orchestrator: sequencing, gating, review
 2. Test-Design: writes tests from spec *before* implementation exists
 3. Developer: implements against specification and locked tests so that tests pass
 
Roles MUST NOT be collapsed into a single agent or a single context. For behaviour-level tests, the Test-Design agent MUST NOT be given visibility into the Developer's implementation, before or during authoring.
Rationale: prevents tests from being derived from — and thereby silently confirming — the very implementation they exist to check.

### D. Iterative Test-Design Engagement for Unit-Level Coverage
Test-Design authors behaviour-level tests first, with no implementation visibility, before implementation begins (Principle B). Because function- and interface-level detail is often not knowable until the Developer has sketched an implementation approach, Test-Design MAY be re-engaged during implementation specifically to author unit-level tests — but only against function/interface signatures (names, inputs, outputs, stated intent) that the Developer has proposed and not yet implemented, never against function bodies. This preserves Red-before-Green at the unit-test granularity without violating the no-implementation-visibility boundary.
Rationale: behaviour-level tests can be written from the spec alone, but unit-level tests for internal transformation logic genuinely require knowing what functions will exist — deferring that narrow slice of test authoring to the point the signatures are proposed is more honest than pretending it can happen up front.

### E. Locked Tests, No Self-Certification
Once the Orchestrator has reviewed a test suite (behaviour- or unit-level) for suitability and locked it, the Developer agent MUST NOT modify test files. It may only change the implementation.
Rationale: without this boundary, a failing test can simply be edited into a passing one, defeating the point of separating the roles.

### F. Tests Prove Spec Compliance, They Are Not a Substitute For It
The Developer implements to satisfy the specification; the locked test suite is the verification mechanism for that compliance, not an independent target to be gamed. Passing all locked tests is necessary but not sufficient — the Orchestrator's review MUST check the implementation against the spec directly, not only against test results.
Rationale: a suite that technically passes can still miss the intent of the spec if "make the tests green" becomes the goal instead of "satisfy the requirement."

### G. Two-Tier Verification: Deterministic Tests and Pre-Deploy Validation (NON-NEGOTIABLE)
Every iteration MUST be checked at two independent, categorically distinct points, per the engineering-quality baseline's Principle IV:
1. **Build-time, deterministic tests** — behaviour/integration/unit tests, run before the human-owned deploy trigger. These are the test suite proper and MUST be deterministic.
2. **Pre-deploy validation, after deployment** — config/state validation: confirms declared external state (references, dependencies, whatever the project's correctness principles specify) actually holds against live real-world conditions. This MAY depend on live networking by design, and its findings are reported to the human rather than silently folded into the deterministic suite's pass/fail.
A passing build-time check MUST NOT be treated as sufficient evidence of a working deployment — only pre-deploy validation confirms that.
Rationale: local/build success does not guarantee deployment success; this principle exists specifically to close that gap, while keeping the deterministic test suite honestly deterministic rather than quietly depending on live state.

### H. Gate Coverage Is Not Optional, Even Without Obvious Logic
Every iteration, regardless of whether it appears to contain branching logic, MUST be covered by the applicable gates (deterministic tests, pre-deployment config/reference validation, and/or post-deployment smoke testing — as defined in the engineering-quality baseline, Principle IV) derived from this project's declared correctness principles (defined in the project's own spec and/or constitution — not here). The absence of branching logic does not imply the absence of a correctness property worth checking.
Rationale: coverage should be driven by what the project has declared as correct, not by an agent's informal judgment of whether a given iteration "seems to need it."

### I. Orchestrator-Gated Ambiguity Resolution
The Orchestrator MUST halt and escalate to the project owner rather than resolve an open or ambiguous requirement on its own initiative.
(The concrete source of open items/clarify gates is instantiated per-project in that project's spec or constitution.)
Rationale: silent resolution of ambiguity is how requirements drift from what was actually intended.

### J. Bounded Retry, Then Human Escalation
On a failed check (test-suitability review, pre-push review, build-time test, pre-deploy validation), the Orchestrator MAY attempt exactly one resolution cycle, returning specific feedback to whichever agent is responsible. A second failure of the *same* check MUST halt the iteration and produce a report to the project owner. No further automatic retries.
Rationale: unbounded auto-retry hides the point at which human judgment is actually needed.

### K. Human-Owned Deploy Trigger (NON-NEGOTIABLE)
Deployment MUST remain a human-performed action. No agent may trigger it directly.
(The concrete deploy mechanism is instantiated per-project in that project's constitution.)
Rationale: keeps an irreversible, externally-visible action under direct human control regardless of how autonomous the rest of the pipeline becomes.

### L. Explicit Human Continuation Gate and Tracking
The Orchestrator MUST NOT begin implementation against a newly derived iteration plan, and MUST NOT advance from a completed iteration to the next, without an explicit confirmation or instruction from the project owner to proceed. An unacknowledged report is not consent. The owner acceptance and authorisation MUST be recorded with the appropriate iteration plan notes (this may leverage SpecKit if it is being used).
Rationale: keeps pacing and go/no-go decisions in human hands even when every automated check has passed.

## Roles

- **Orchestrator** — owns scope derivation, iteration sequencing, briefing, test-suitability review, pre-push review, retry/escalation, and requesting (never performing) deployment.
- **Test-Design** — authors behaviour-level tests before implementation, and may be re-engaged for unit-level tests against proposed signatures during implementation (Principle C). Never sees implementation bodies.
- **Developer** — implements to satisfy the spec, using the locked tests as proof of compliance; cannot modify locked tests; does not deploy.
- **External Project Owner** - resolves ambiguity escalations, performs the deploy trigger, and is the sole source of continuation confirmation (Principle K).

## SpecKit Phase Mapping

| SpecKit phase | Owner |
|---|---|
| Constitution | Standing baseline (this file + engineering-quality baseline), referenced by every role |
| Specify | Human, via SpecKit, before Orchestrator activation |
| Clarify | Human, via SpecKit, before Orchestrator activation (ambiguity surfacing later is handled by the Orchestrator's own gate, Principle H) |
| Plan | Human, via SpecKit, before Orchestrator activation |
| (Iteration/deliverable scoping — not a named SpecKit phase) | Orchestrator, once Plan exists, agreed with the human (Principle A) |
| Tasks | Orchestrator, generated fresh per iteration, not all up front |
| Analyze | Orchestrator's pre-push cross-artifact consistency check |
| (Test authoring — not a named SpecKit phase) | Test-Design — behaviour-level before implementation, optionally unit-level during implementation (Principle D) |
| Implement | Developer |

## Workflow

**Activation and scoping** (once, or whenever re-planning is triggered):
1. Orchestrator reviews the existing spec and plan and derives a candidate iteration breakdown (Principle A)
2. Orchestrator proposes the breakdown to the human; does not proceed without confirmation or adjustment (Principle L)
3. If the plan proves too broad/ambiguous to scope confidently, Orchestrator performs a re-planning pass with the human before re-proposing iterations

**Per iteration:**
1. Orchestrator checks open items against this iteration's scope — halts for human input if blocked (Principle I)
2. Orchestrator drafts a scoped task brief for this iteration only
3. Test-Design orients to the project's test tooling/framework (Principle B), then writes behaviour-level tests from the brief and the project's declared correctness principles, without seeing any implementation (Principle C)
4. Orchestrator reviews tests for suitability, locks them (Principle E)
5. Developer orients to the project's toolchain (Principle B), then proposes an implementation approach (function/interface signatures) sufficient to satisfy the spec and the locked behaviour tests
6. Where transformation/unit-level logic is involved, Test-Design may be re-engaged against those signatures — not bodies — to write unit tests before they're implemented (Principle D)
7. Developer implements against the spec and all locked tests, commits locally, reports back — passing tests is evidence of spec compliance, not the goal itself (Principle F)
8. Orchestrator runs the deterministic test gate and reviews the result against both the tests and the underlying spec — one retry on failure, then halt (Principle J)
9. Orchestrator runs the pre-deployment config/reference validation gate — one retry & revision allowed on failure, then halt (Principle J); a failure here blocks deployment authorization, so step 10 does not happen until this passes
10. Human reviews and performs the deploy trigger (Principle K)
11. Orchestrator runs the post-deployment smoke test, confirming both the deployment mechanism succeeded and that real (unmocked) integration points function and presenting the results (Principle J)
12. Orchestrator reports the iteration's outcome and waits for explicit human confirmation before advancing to the next iteration (Principle L)

## Governance

This file is amended independently of any single project: propose with rationale, get approval, update dependent agent definition files in the same change, record the change in the Sync Impact Report at the top of this file. Projects referencing this file pick up amendments the next time their container pulls current standards — no per-project re-composition step required.

**Version**: 1.0.0 | **Ratified**: [RATIFICATION_DATE] | **Last Amended**: [LAST_AMENDED_DATE]