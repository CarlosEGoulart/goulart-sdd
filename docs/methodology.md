# Goulart SDD Methodology

An opinionated Spec-Driven Development methodology built on the official OpenSpec engine. It adds disciplined lifecycle sequencing, explicit human gates, TDD, independent review, focused implementation tasks, and final verification while keeping the upstream engine unmodified.

## Purpose

Goulart SDD exists because AI coding assistants are powerful but unpredictable when requirements live only in chat history. OpenSpec adds a spec layer, but its default workflow lacks engineering discipline: no adversarial review, no TDD enforcement, no independent verification, and no bounded review loops. Goulart SDD adds these practices into a coherent workflow suitable for personal, academic, portfolio, and long-lived side projects.

## Architecture and Ownership

```
OpenSpec engine (upstream, unmodified)
      |
      v
Goulart methodology (schema + docs + config)
      |
      v
coding-agent adapter (goulart-* skills/commands)
```

**OpenSpec engine** owns:

- CLI and change lifecycle
- Schema engine and artifact dependency graph
- Delta-spec mechanics and validation
- Generated `opsx-*` commands and `openspec-*` skills
- Archive mechanics

**Goulart methodology** owns:

- Custom `goulart-sdd` schema (`openspec/schemas/goulart-sdd/`)
- Methodology documentation (`docs/`)
- Project configuration (`openspec/config.yaml`)
- Methodology sequencing and gate semantics

**coding-agent adapter** owns:

- `goulart-plan`, `goulart-review`, `goulart-apply`, `goulart-verify`, `goulart-archive` entry points
- Skills and commands that implement methodology sequencing

OpenCode is the first reference adapter. The methodology and schema are agent-agnostic. A user can create a new adapter for a different coding agent without modifying the methodology or schema.

## Compliance Boundary

**Goulart-compliant execution** uses `goulart-*` lifecycle entry points and honors their STOPs, review gates, human decisions, verification prerequisites, and archive rules. Disclosed degraded reviews and explicit overrides retain their disclosed limitations; they do not supply the guarantees they waive.

**Raw OpenSpec execution** (direct `/opsx-propose`, `/opsx-apply`, `/opsx-archive`, `openspec ...`) remains available as an escape hatch. Raw execution may bypass Goulart sequencing, review handoff, or human gates. The result must not be represented as Goulart-compliant. In OpenSpec 1.13, raw `/opsx-propose` creates a change and may generate its entire schema-required planning set, bypassing Goulart sequencing, review handoff, or human gates.

Do not attempt to modify or disable upstream OpenSpec commands.

## Roles

| Role | Responsibility |
|---|---|
| **Author** | Develops and revises proposal, specs, and design. After accepted plan review, may continue test-plan and tasks through `goulart-plan`. |
| **Reviewer** | Performs plan or code review. Uses a context separate from author/implementer when possible. Writes only the selected review artifact. Reports findings rather than silently repairing reviewed artifacts. |
| **Implementer** | Executes one eligible implementation task per `goulart-apply` invocation. Follows approved specs/design/test-plan/task. Stops on spec drift instead of silently changing requirements. |
| **Verifier** | Independently consumes final implementation/review/test evidence. Checks verification prerequisites. Performs the final audit. Writes the verify result. Does not repair implementation, review, or planning artifacts. |
| **Human** | Retains final authority to accept, request revision, triage findings, defer, override permitted gates, and dispose archive warnings. Human decisions are explicit and never fabricated by the agent. |

## Artifact and Lifecycle Overview

The conceptual artifact flow is:

```
proposal → specs → design → plan-review → test-plan → tasks → code-review → verify → archive
```

This is NOT one uninterrupted command execution. There are mandatory lifecycle STOPs and later invocations. The full lifecycle requires multiple separate `goulart-*` invocations with human decisions between them.

## Planning Phase

### First invocation — initial planning

`goulart-plan` creates or continues:

1. proposal
2. specs
3. design

Then **STOPs** and instructs the human to run:

```
goulart-review plan
```

in a fresh or separate review context when possible. Initial planning must not automatically continue through plan-review, test-plan, tasks, or implementation.

### Plan review and human decision

Plan-review evaluates proposal/spec/design consistency, compliance, quality, feasibility, scope, ambiguity, testability, and material omissions. The reviewer posture is adversarial and read-only toward reviewed artifacts.

**Reviewer verdicts:**

| Verdict | Meaning |
|---|---|
| APPROVE | Plan is acceptable as-is. |
| APPROVE_WITH_CHANGES | Required changes must be applied before progression. |
| REVISE | Normal progression is blocked. Artifacts must be revised and re-reviewed. |

**Human disposition after plan-review:**

| Disposition | Effect |
|---|---|
| ACCEPTED | Permits downstream planning, provided the review is current and all required changes are applied. |
| REVISE | Artifacts revised and re-reviewed. |
| OVERRIDDEN | Explicit exception with mandatory reason. Does not make a stale review fresh. |

The reviewer verdict alone does not always authorize continuation. An explicit human disposition is mandatory.

Key combinations:
- APPROVE + human ACCEPTED + current review → downstream planning may proceed.
- APPROVE_WITH_CHANGES + unapplied Required Changes → BLOCKED.
- APPROVE_WITH_CHANGES + material corrections applied → re-review normally required, or explicit OVERRIDDEN with mandatory reason.
- APPROVE_WITH_CHANGES + demonstrably non-material corrections → human may record ACCEPTED with materiality rationale without re-review.
- REVISE or human REVISE → normally revise and re-review.

### Second invocation — planning continuation

After a permitting plan-review and human gate, a later `goulart-plan` invocation checks:

- verdict and disposition
- Required Changes and materiality
- review freshness
- degraded-review acknowledgement if applicable
- override evidence if applicable
- round and escalation state

Only when permitted does planning continue to:

1. test-plan
2. tasks

When planning completes, `goulart-plan` reports planning complete, **STOPs**, and identifies `goulart-apply` as the next Goulart-compliant step. No additional planning command is needed.

## Test Planning

The test-plan maps every spec scenario to explicit validation entries. Each entry uses exactly one of three validation types:

| Type | When Used | Required Record |
|---|---|---|
| AUTOMATED | Scenario verifiable by automated test (unit, integration, E2E). | Test file path and test function/method name. |
| MECHANICAL | Scenario verifiable by deterministic tool/command (lint, typecheck, schema validation, build). | Exact validation command. |
| SEMANTIC | When automated or mechanical validation is not reasonably suitable. | Rationale, evaluator, evaluation description. |

SEMANTIC is not an escape hatch for inconvenient tests. Every scenario in every spec file must be mapped to at least one validation entry. Unmapped scenarios are blocking.

Planning evidence is not execution proof. Test-plan status fields (RED, GREEN, etc.) track visibility; they do not prove historical TDD chronology.

## Task Generation and Traceability

Implementation tasks are small and focused, each suitable for one coding-agent session. Each focused task maps to one primary spec scenario/acceptance criterion or test-plan entry. Broad integration verification tasks are allowed only when genuinely cross-cutting. Tasks are ordered by dependency.

Fake task splitting merely to create RED/GREEN/REFACTOR checkboxes is not the methodology. One focused implementation task internally follows RED → GREEN → REFACTOR where automated testing is appropriate.

## Implementation and TDD

### One task per invocation

`goulart-apply`:

1. Resolves the first eligible pending task
2. Loads only relevant context plus allowed repository exploration
3. Implements exactly one task
4. Validates it
5. Marks only that task complete
6. Reports the result
7. **STOPs**

A later invocation handles the next task. `goulart-apply` must not run the entire task list in one accumulated conversational context by default.

When all tasks are complete, `goulart-apply` **STOPs** and instructs the human to run:

```
goulart-review code
```

in a fresh or separate context where possible.

### TDD execution

For AUTOMATED behavioral validation where automated testing is appropriate:

- **RED**: write or update the test first; execute it; confirm failure for the expected reason before implementation.
- **GREEN**: implement the minimum required behavior to make the relevant test pass.
- **REFACTOR**: improve structure where useful; keep relevant test and applicable suite green.

For MECHANICAL or SEMANTIC work: do not invent automated tests merely to claim TDD.

### Bounded retry

If the same test fails twice consecutively for the same root cause within a single task, the implementer stops and reports the issue rather than attempting a third approach.

### Spec drift

If implementation reveals that a requirement or scenario is wrong, incomplete, contradictory, materially ambiguous, or untestable, the implementer must STOP. Do not silently rewrite the spec, weaken the test, or reinterpret the requirement merely to pass. The planning/spec lifecycle must be corrected before implementation resumes.

### Truthfulness

- Do not claim a test already exists or already passes unless current repository evidence establishes that fact.
- Do not fabricate evidence at any stage.

## Code Review

Code review happens after all implementation tasks are complete. Code-review examines implementation against specs, design, test-plan, and code quality concerns. The reviewer does not silently fix implementation.

**Verdicts:**

| Verdict | Meaning |
|---|---|
| APPROVE | No blocking implementation changes required. Non-blocking findings still require human triage. |
| APPROVE_WITH_CHANGES | At least one implementation change required before verification. |
| REVISE | Normal progression to verify is blocked. Implementation must be revised and re-reviewed. |

Where findings exist, human per-finding triage uses:

| Triage | Effect |
|---|---|
| ACCEPT | Finding addressed. Justification optional. |
| REJECT | Finding rejected. Justification required. |
| DEFER | Finding deferred. Justification required. |

A clean review with zero findings does not require artificial finding triage; the verdict stands as-is.

REJECT or DEFER without justification is invalid triage. All mandatory triage must be complete before normal progression.

Material fixes after code review make that review stale and normally require re-review. Non-material corrections may avoid re-review only with the required explicit rationale and evidence.

## Review Independence and Degraded Mode

Independent review uses a context or session separate from the author or implementer context. The fallback hierarchy is:

1. **Harness-supplied separate/fresh context** — preferred.
2. **User-managed separate clean session** — fallback, still independent.
3. **Degraded review** in the same or non-separate context — last resort only.

Only when neither separate-context option is available may review proceed without a separate context in degraded mode.

Degraded review must:

- disclose the limitation;
- never claim to be independent;
- require actual human acknowledgement before any permitting downstream gate where the contract requires that acknowledgement.

Cross-model review is optional. A different model alone does not establish independence. Separate context is a process practice, not mechanical proof of cognitive isolation.

## Review Rounds, Material Changes, and Staleness

### Two-round limit

Both review stages (plan-review and code-review) have a maximum of two review-revision rounds. Each artifact records `ROUND: 1 | 2` and a concise previous-round disposition or history.

After round 2, if unresolved blocking conditions, unresolved required changes, or another material change requiring further review remain: **STOP and escalate to the human**. Do not automatically start round 3, reset round to 1, or erase previous-round history.

The limit includes re-reviews triggered by REVISE, APPROVE_WITH_CHANGES material corrections, or other material post-review changes.

### Two staleness lineages

**Plan-review staleness:** material changes to proposal, specs, or design after plan-review make the plan-review stale. Normal progression requires re-review unless an explicit human override is recorded. The override is an exception, not evidence of freshness.

**Code-review staleness:** material changes to source code, tests, or implementation-relevant config after code-review make the code-review stale. Re-review is required before normal progression to verify; non-material corrections may avoid re-review only with a clear recorded justification.

Staleness detection uses repository revision or diff information where available, with conservative semantic comparison fallback. Filesystem timestamps must not be used as reliable staleness evidence. Timestamps are explicitly excluded due to instability across common Git operations.

An override does not make a stale review fresh. It records an exception; it does not imply the changed implementation was reviewed.

## Verification

### Prerequisites

Before producing a final DECISION, `goulart-verify` independently checks prerequisites:

- code-review exists and its verdict or state permits progression
- mandatory human finding triage is complete
- all tasks in tasks.md are complete
- every required test-plan entry has a final evaluable state (completed AUTOMATED or MECHANICAL check with recorded result, or documented SEMANTIC evaluation)
- degraded-review disclosure and human acknowledgement are satisfied where applicable
- review freshness for both plan-review and code-review lineages is assessed, with any explicitly permitted override recorded

All-task completion is non-waivable. A review override does not automatically waive these separate prerequisites. If any prerequisite is not satisfied: verification audit does not run; the blocking prerequisite and gap are recorded; STOP. Do not emit DECISION: FAIL.

### Final audit

After prerequisites permit verification, the audit checks:

- required spec scenarios against implementation
- AUTOMATED, MECHANICAL, and SEMANTIC validation evidence
- behavioral coverage integrity
- full test suite results
- scope drift
- review and evidence consistency

### Decision semantics

| Decision | Meaning | Archive proceeds? |
|---|---|---|
| PASS | All checks pass with no blocking findings and no warnings. | Yes. |
| PASS_WITH_WARNINGS | No blocking findings but genuine warnings remain. | Only after human explicitly accepts or defers every warning. Must not silently mean PASS. |
| FAIL | Blocking issues found. | No. Human override must be explicit and recorded with reason. |

## Archive

`goulart-archive` consumes an already completed verify result. It never performs verification itself. It never automatically invokes `goulart-verify`.

Eligibility routing:

| Verify result | Archive behavior |
|---|---|
| Valid and current PASS | Delegate to upstream OpenSpec archive. |
| Valid and current PASS_WITH_WARNINGS | Require actual human disposition (ACCEPT or DEFER) for every warning. Then delegation may occur. |
| Valid and current FAIL | Block normally. Proceed only under explicit archive-stage human override with mandatory reason. |
| STALE, UNKNOWN, malformed, or incomplete verify evidence | STOP. No automatic verification. Human must run `goulart-verify` when new verification is needed. |

Archive warning dispositions and FAIL overrides affect archive eligibility; they do not rewrite the verify decision.

Actual archive mechanics remain upstream OpenSpec responsibility.

## Gate Categories

### Artifact gates

Provided by OpenSpec's schema dependency graph (`requires:` field). They enforce artifact file existence and ordering. They do NOT prove semantic quality, implementation completion, independent review, or historical TDD chronology.

### Execution gates

Provided by Goulart adapter commands and skills. They enforce methodology sequencing at the adapter level. These are not necessarily provable from Git history.

### Repository and tool gates

Provided by automated tests, lint, build, typecheck, OpenSpec validation, schema validation, and deterministic structural checks. They validate repository state but generally cannot prove agent cognition or process chronology.

### Human decisions

Human acceptance, triage, override, and warning disposition are explicit human authority. They are not mechanically inferred. One stage's override does not automatically override another gate.

## Evidence and Truthfulness Boundaries

Goulart SDD records and validates evidence, but it must not claim guarantees that the available evidence cannot prove:

- timestamps do not prove review freshness
- test-plan status fields do not prove TDD chronology
- using another model does not prove independence
- an override does not make stale evidence fresh
- a partial or prerequisite-blocked verify artifact is not a completed PASS or FAIL
- agent-generated text cannot fabricate a human decision

## Deferred Scope

The following are deferred and not currently implemented:

- light workflow profile
- adapters beyond the current OpenCode reference adapter
- broader CI enforcement beyond basic structural checks
- per-task code review (only per-change review in v0.1)
- enterprise governance and elaborate audit or evidence infrastructure
- OpenSpec CLI fork or vendor modifications
- mandatory LLM vendor or model

Do not describe deferred functionality as implemented.

## Lifecycle Summary

```
goulart-plan (1st invocation)
    proposal → specs → design
    STOP → instruct: goulart-review plan

goulart-review plan
    plan-review artifact
    findings + VERDICT

HUMAN DECISION
    ACCEPTED | REVISE | OVERRIDDEN

goulart-plan (2nd invocation)
    check verdict, freshness, gates
    if permitted:
        test-plan → tasks
        STOP → instruct: goulart-apply

goulart-apply (per task)
    one task: RED → GREEN → REFACTOR
    STOP (repeat per invocation)
    when all complete: STOP → instruct: goulart-review code

goulart-review code
    code-review artifact
    findings + VERDICT

HUMAN TRIAGE (where findings exist)
    ACCEPT | REJECT | DEFER per finding

goulart-verify
    prerequisites → audit → DECISION

goulart-archive
    check DECISION → delegate or block
```
