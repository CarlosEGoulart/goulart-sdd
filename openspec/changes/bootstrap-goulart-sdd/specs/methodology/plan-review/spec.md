## Purpose

Defines the pre-implementation adversarial review that gates downstream artifacts and implementation, including explicit human acceptance.

## ADDED Requirements

### Requirement: Plan review gates implementation
A plan-review artifact SHALL exist with an acceptable verdict and human acceptance before the test-plan, tasks, or apply may proceed.

#### Scenario: Apply blocked without review
- **WHEN** the apply operation is invoked and plan-review.md does not exist
- **THEN** the workflow SHALL refuse to proceed

#### Scenario: Apply blocked on REVISE
- **WHEN** the apply operation is invoked and plan-review.md verdict is REVISE
- **THEN** the workflow SHALL refuse to proceed until artifacts are revised and re-reviewed

#### Scenario: Apply blocked without human acceptance
- **WHEN** the plan-review verdict is APPROVE or APPROVE_WITH_CHANGES but no human decision has been recorded
- **THEN** the workflow SHALL refuse to proceed until the human records a disposition

### Requirement: Plan-review verdict semantics
The plan-review SHALL emit exactly one verdict: APPROVE, APPROVE_WITH_CHANGES, or REVISE. Each verdict has specific human disposition semantics.

#### Verdict: APPROVE

The plan is acceptable as-is.

- Human may record `STATUS: ACCEPTED` with optional reason.
- Downstream planning may proceed.

#### Verdict: APPROVE_WITH_CHANGES

Required changes MUST be applied before proceeding.

- Human SHALL NOT record `STATUS: ACCEPTED` if required changes remain unapplied.
- After material required changes are applied: run plan-review again before normal progression; OR human MAY explicitly record `STATUS: OVERRIDDEN` with mandatory reason to proceed without re-review.
- `ACCEPTED` MUST NOT silently mean that unapplied required changes are acceptable.

#### Verdict: REVISE

Normal progression is blocked.

- Artifacts must be revised and reviewed again.
- Human may explicitly `STATUS: OVERRIDDEN` only with mandatory justification.
- Maximum 2 review/revision rounds per review stage.
- After the second unresolved REVISE/required-changes round: STOP and escalate to the human with a clear summary of unresolved issues.

#### Scenario: Human accepts plan
- **WHEN** the plan-review verdict is APPROVE
- **THEN** the human SHALL record STATUS: ACCEPTED with optional reason
- **THEN** downstream planning may proceed

#### Scenario: Human accepts with changes applied
- **WHEN** the plan-review verdict is APPROVE_WITH_CHANGES
- **THEN** the human SHALL ensure required changes are applied
- **THEN** the human SHALL record STATUS: ACCEPTED
- **THEN** downstream planning may proceed

#### Scenario: Human proceeds without re-review
- **WHEN** the plan-review verdict is APPROVE_WITH_CHANGES
- **THEN** the human MAY record STATUS: OVERRIDDEN with mandatory reason to proceed without re-review

#### Scenario: Human requests revision
- **WHEN** the human is unsatisfied with the plan
- **THEN** the human SHALL record STATUS: REVISE with reason
- **THEN** artifacts SHALL be revised and re-reviewed

#### Scenario: Human overrides reviewer
- **WHEN** the plan-review verdict is REVISE but the human chooses to proceed
- **THEN** the human SHALL record STATUS: OVERRIDDEN with mandatory reason
- **THEN** downstream planning may proceed

#### Scenario: Review escalation
- **WHEN** the second review round produces a REVISE verdict or unapplied APPROVE_WITH_CHANGES
- **THEN** the workflow SHALL STOP and escalate to the human

### Requirement: Fresh context for review with fallback hierarchy
The plan-reviewer SHOULD use a fresh context. The reviewer SHALL NOT be the same context that authored the proposal, specs, or design.

Fresh-context independence SHALL be prioritized over cross-model review. The fallback hierarchy is:

1. **Harness-created fresh context** (preferred) — the coding-agent harness spawns a provably isolated context.
2. **User-managed fresh session** (fallback) — the user starts a separate clean session manually.
3. **Degraded review mode** (last resort) — when neither 1 nor 2 is possible:
   - The review MAY still be performed.
   - The limitation MUST be explicitly disclosed.
   - The human MUST acknowledge the degraded independence.
   - The result MUST NOT be described as an independent review.

Cross-model review remains optional and is NOT required for independence.

#### Scenario: Harness-created fresh context
- **WHEN** the harness can spawn a fresh context
- **THEN** the plan-reviewer SHALL use a fresh context

#### Scenario: User-managed fresh session
- **WHEN** the harness cannot spawn fresh contexts but the user can start a separate session
- **THEN** the user SHALL start a separate clean session for the review
- **THEN** the review result SHALL be described as independently reviewed

#### Scenario: Degraded review mode
- **WHEN** neither harness-created fresh context nor user-managed fresh session is possible
- **THEN** the review MAY proceed in degraded mode
- **THEN** the limitation SHALL be explicitly disclosed in the review artifact
- **THEN** the human SHALL acknowledge the degraded independence
- **THEN** the result SHALL NOT be described as an independent review

### Requirement: Reviewer read-only posture
The plan-reviewer SHALL NOT modify proposal.md, specs/, or design.md during the review. The reviewer's ONLY permitted write is the review output file.

#### Scenario: Reviewer does not modify artifacts
- **WHEN** the plan-reviewer runs
- **THEN** it SHALL only read proposal.md, specs/, design.md, and relevant source files
- **THEN** it SHALL only write to the review output file

### Requirement: Review-round persistence
Each review artifact SHALL preserve enough information to know the current round and previous outcome.

- `ROUND: 1 | 2`
- Concise previous-round disposition/history inside the same artifact.

The artifact SHALL NOT simply overwrite the review with no evidence of which round is being performed.

#### Scenario: First round
- **WHEN** the plan-review is performed for the first time
- **THEN** the artifact SHALL record ROUND: 1

#### Scenario: Second round after revision
- **WHEN** the plan-review is performed after artifacts were revised following a REVISE or APPROVE_WITH_CHANGES verdict
- **THEN** the artifact SHALL record ROUND: 2
- **THEN** the artifact SHALL include a concise summary of the previous-round disposition

### Requirement: Structured review findings
Each review finding SHALL include a severity level (Critical, Moderate, Suggestion) and a category (compliance, quality, feasibility, scope, risk).

#### Scenario: Finding with severity and category
- **WHEN** the reviewer identifies an issue
- **THEN** the finding SHALL specify severity and category in the review output
