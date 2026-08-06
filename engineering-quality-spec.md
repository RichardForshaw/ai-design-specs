<!--
Sync Impact Report
Version change: N/A (baseline) -> 1.0.0
-->

# Engineering Quality Baseline

## Description
Project-independent engineering quality directives. Meant to be referenced (not copied) by a project's `constitution.md`. Where a principle needs a project-specific instantiation, that value is recorded in the project's own constitution, not here; this file stays invariant. This file must *not* be modified by an agent unless specifically instructed by a human user.

## Core Principles

### I. Test-Driven Changes With Human Review (NON-NEGOTIABLE)
Code changes MUST start with a test that captures the expected behavior (either BDD or unit-level) and fails for the intended reason. Tests MUST be connected to a specification or User Story for the project. Tests MUST be reviewed by a human before implementation begins. Implementation proceeds only after Red is confirmed, then follows Red-Green-Refactor to Green.
(The concrete scenario/test format — e.g. Gherkin under a features directory, or a plain test framework — is instantiated per-project in that project's constitution.)
Rationale: prevents speculative coding, reduces regressions, and keeps scope focused on agreed behavior rather than incidental implementation detail.

### II. Functional-First, Declarative Design
Prefer pure, deterministic functions for new code and refactors. Avoid hidden mutable state; PREFER that data transformations return new values rather than mutate inputs. Side effects (network, filesystem, environment, time, randomness, external providers) MUST be isolated behind explicit boundary/adapter functions.
Object-oriented design is permitted only for an agreed reason. Any departure from functional-first MUST be raised before implementation, explicitly agreed with the user, and recorded in the plan or review notes. If using OO classes, PREFER a declarative implementation style.
Rationale: functional-style code improves testability, predictability, and refactoring safety. Declarative style improves readability and highlights key behaviours.

### III. Explicit Toolchain Reference and Execution Boundary

Three distinct requirements, kept together because they all guard against the same failure mode — an unstated assumption about how or where code runs:

 (a) *Toolchain reference*. A repository-level PROJECT_TOOLCHAIN file MUST describe the development tools in use and the preferred command for each common action (install, build, test, run any ingestion/generation step, etc.), so a command's correct invocation is never left to memory or convention.

 (b) *Dependency reproducibility*. Dependencies MUST be pinned via the ecosystem's standard manifest/lockfile (e.g. requirements.txt/lockfile, package.json/lockfile, go.mod/go.sum), committed to the repository.

 (c) *Execution boundary*. By default, assume the development environment is in a container and all project/toolchain commands (install, build, test, generation, implementation) run inside the container. This default holds unless a project's constitution explicitly states a different split and why.

Rationale: (a) and (b) keep operational knowledge and dependency versions reproducible; (c) removes ambiguity about where commands execute by fixing a sane default instead of requiring a bespoke host/container decision on every project.

### IV. Deterministic, Explicit Testing Standards

 (a) *Behaviour tests are mandatory*. Every feature is treated as a behaviour and MUST have at least one behaviour-level test covering its observable contract. This also serves as a legible record of what the system does and a way to track project progress (see Principle V).

 (b) *Integration tests, scoped by risk*. Integration tests SHOULD be written when a feature interfaces with another system. Where that system is a remote/external service, tests MUST demonstrate that our system satisfies the required contract/API shape. Otherwise, a behaviour test covering the integration is an acceptable substitute for a full integration suite.

 (c) *Unit tests, mandatory for transformation logic*. Unit tests are preferred generally but not required everywhere; they ARE mandatory for transformation/pure-function logic, consistent with Principle II's functional-first preference. For other examples, assess the value depending on the overlap with the associated behaviour tests from (a).

 (d) *Determinism applies to the test suite proper*. Behaviour, integration, and unit tests MUST be deterministic and isolated: no dependency on live networking, live third-party state, or ambient/carry-over state between tests.

 (e) *Pre-deployment config/reference validation is a distinct gate, not a test*. Confirming that a declared external reference (a link, an address, a config value pointing outside the system) currently resolves is config validation, not a behaviour/integration/unit test — its entire purpose is to check live real-world state (which may be external), so it MAY depend on live networking. It sits in a separate category from (d). It runs before deployment and gates whether deployment mey then be requested at all: a failing reference blocks a deployment request rather than being discovered after the fact. Findings are reported to the project owner, not folded into the deterministic suite's pass/fail.

 (f) *Post-deployment smoke testing is a distinct gate, and must prove more than reachability*. Once a deployment has occurred, a smoke test MUST confirm the deployment itself is reachable and responding, AND provide minimal proof that basic functionality is operational in the production environment. Production differs from the tested environment precisely where it matters most — external systems are real there, not mocked — so reachability alone does not prove those integrations work. This is deliberately shallow relative to (d): a minimal proof each real integration functions, not a full re-test of its behavior. It does not re-validate what the config/reference gate in (e) already confirmed, and it is a third distinct category from (d) and (e).

 (g) *Real-world fixtures, captured and dated*. Fixtures representing real-world input MUST be sourced from real captured examples, not synthetic data invented for convenience, and versioned with a capture timestamp so tooling can flag a fixture that has grown older than an agreed staleness threshold for refresh. Exception: error-response/failure-mode fixtures (malformed feed, timeout, 4xx/5xx from a remote system, etc.) MAY be synthetic, since real examples of a remote system's error states are often impractical to capture on demand. Synthetic error fixtures SHOULD still be plausible/representative of the real system's actual error shape where that shape is known (e.g. from documentation or an observed one-off failure), not arbitrary.

The three-gate sequence. Together, (d), (e), and (f) define three sequential, categorically distinct gates: deterministic tests gate progression toward deployment; pre-deployment config/reference validation gates authorization to actually deploy; post-deployment smoke testing verifies the deployment mechanism succeeded. This principle defines what each gate checks and why they're distinct. It does not define who executes them or in what procedural context — that is a process/orchestration concern, specified in the Agent Architecture baseline, not here.

Rationale: behaviour-level tests keep an accurate, living record of what the system does; scoping integration/unit requirements by actual risk (remote contract surfaces, transformation logic) avoids ceremony that doesn't earn its cost; splitting reference validation, smoke testing, and the deterministic suite into three distinct gates resolves the contradiction between "tests must be deterministic" and "external/deployment state must be checked live," rather than exempting live checks from determinism inside one category; dated fixtures prevent a captured example from silently drifting out of sync with the real system it represents; keeping gate definitions here and gate execution in the Agent Architecture baseline keeps each document answering one question, not both.

### V. Traceability and Test Tagging
Task ordering MUST place test tasks before the implementation tasks that satisfy them (Red before Green). Tests SHOULD be tagged (if the chosen framework allows it) so they can be (a) traced back to the scenario/requirement/feature they cover, and (b) run selectively by feature-group tag, rather than relying on file location or naming convention alone to find "the tests for X." Code review MUST verify test-before-implementation ordering and tag correctness, along with compliance with the other Core Principles, before approval.
Rationale: explicit traceability enables reliable audits, allows for confident progress monitoring and keeps engineering quality consistent as the codebase and contributor set grow.

### VI. Phase Deliverability and Default-First Slicing
Each delivery phase beyond initial setup/foundational work MUST conclude with a functional, independently usable increment that can be deployed and validated against the feature's intended value. A phase may be a full user story or a narrower slice, but it must remain independently usable at that boundary.
Rationale: deliverable phase boundaries keep plans legible, preserve useful increments, allow for earlier refinement or re-planning and prevent configuration or deployment work from outrunning the behavior it modifies.

## Governance

This file is amended independently of any single project: propose with rationale, get approval, record the change in the Sync Impact Report at the top of this file. Projects referencing this file pick up amendments the next time their container pulls current standards — no per-project re-composition step required, since projects reference rather than copy.

**Version**: 1.0.0 | **Ratified**: [2026-08-05] | **Last Amended**: [2026-08-05]