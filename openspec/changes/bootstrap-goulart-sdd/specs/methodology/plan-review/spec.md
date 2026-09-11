## Purpose

Defines the pre-implementation adversarial review that gates downstream artifacts and implementation, including explicit human acceptance.

## ADDED Requirements

### Requirement: Plan review gates implementation
A plan-review artifact SHALL exist before Goulart-compliant test-plan, tasks, or apply may proceed. The gate SHALL require the verdict conditions below, STATUS: ACCEPTED or an explicitly permitted STATUS: OVERRIDDEN with reason, and resolution of staleness through re-review or the recorded override. Degraded review additionally requires disclosure and human acknowledgement; it does not establish independence.

#### Scenario: Apply blocked without review
- **WHEN** goulart-apply is invoked and plan-review.md does not exist
- **THEN** the workflow SHALL refuse to proceed

#### Scenario: Apply blocked on REVISE
- **WHEN** goulart-apply is invoked and plan-review.md verdict is REVISE without a recorded human override with mandatory reason
- **THEN** the workflow SHALL refuse to proceed until artifacts are revised and re-reviewed

#### Scenario: Apply blocked without human acceptance
- **WHEN** the plan-review verdict is APPROVE or APPROVE_WITH_CHANGES but no human decision has been recorded
- **THEN** the workflow SHALL refuse to proceed until the human records a disposition

### Requirement: Plan-review verdict semantics
The plan-review SHALL emit exactly one verdict: APPROVE, APPROVE_WITH_CHANGES, or REVISE. Each verdict has specific human disposition semantics.

#### Verdict: APPROVE

The plan is acceptable as-is.

- Human records `STATUS: ACCEPTED` with optional reason.
- Downstream planning may proceed.

#### Verdict: APPROVE_WITH_CHANGES

Required changes MUST be applied before proceeding.

After applying Required Changes, determine whether changes are material:

- **Material changes** (changes that affect requirements, scope, architecture, acceptance criteria, or implementation assumptions): new plan-review round required, unless human explicitly records `STATUS: OVERRIDDEN` with mandatory reason.
- **Non-material changes** (editorial corrections that do not change requirements, scope, architecture, acceptance criteria, or implementation assumptions): human MAY record `STATUS: ACCEPTED` with justification for why changes are classified as non-material. Re-review MAY be skipped.

The review artifact MUST state why the changes are classified as non-material when re-review is skipped.

`ACCEPTED` MUST NOT be used to bypass unapplied Required Changes.

#### Verdict: REVISE

Normal progression is blocked.

- Artifacts must be revised and reviewed again.
- Human may explicitly `STATUS: OVERRIDDEN` only with mandatory justification.
- Maximum 2 review/revision rounds per review stage.
- After round 2 leaves REVISE, unresolved Required Changes, or a material change requiring further review: STOP and escalate to the human with a clear summary of unresolved issues. Do not start a third round automatically. The same limit applies to APPROVE_WITH_CHANGES and other material post-review changes.

#### Scenario: Human accepts plan
- **WHEN** the plan-review verdict is APPROVE
- **THEN** the human SHALL record STATUS: ACCEPTED with optional reason
- **THEN** downstream planning may proceed

#### Scenario: Human accepts with non-material changes applied
- **WHEN** the plan-review verdict is APPROVE_WITH_CHANGES
- **AND** the required changes are demonstrably non-material (editorial corrections only)
- **AND** the changes have been applied
- **THEN** the human MAY record STATUS: ACCEPTED with justification for non-material classification
- **THEN** downstream planning may proceed without another review

#### Scenario: Material changes require re-review
- **WHEN** the plan-review verdict is APPROVE_WITH_CHANGES
- **AND** the required changes are material
- **THEN** a new plan-review round SHALL be required after changes are applied for normal progression; at the two-round limit the workflow SHALL stop and escalate instead

#### Scenario: Human overrides re-review
- **WHEN** the plan-review verdict is APPROVE_WITH_CHANGES and all Required Changes have been applied, including material changes
- **THEN** the human MAY record STATUS: OVERRIDDEN with mandatory reason to proceed without re-review
- **THEN** the artifact SHALL record the affected inputs and waived re-review requirement without claiming the stale review covers the changed plan

#### Scenario: Unapplied Required Changes block acceptance
- **WHEN** APPROVE_WITH_CHANGES has unapplied Required Changes
- **THEN** STATUS: ACCEPTED SHALL NOT permit downstream planning or goulart-apply

#### Scenario: Human requests revision
- **WHEN** the human is unsatisfied with the plan
- **THEN** the human SHALL record STATUS: REVISE with reason
- **THEN** artifacts SHALL be revised and re-reviewed

#### Scenario: Human overrides reviewer
- **WHEN** the plan-review verdict is REVISE but the human chooses to proceed
- **THEN** the human SHALL record STATUS: OVERRIDDEN with mandatory reason
- **THEN** downstream planning may proceed

#### Scenario: Review escalation
- **WHEN** round 2 leaves REVISE, unresolved Required Changes, or material changes requiring further review
- **THEN** the workflow SHALL STOP and escalate to the human

### Requirement: Independent review
Independent review SHALL use a context/session separate from the author/implementer context.

#### Scenario: Independent review with fresh context
- **WHEN** an independent plan-review is performed using a harness-supplied separate context or a user-managed separate clean session
- **THEN** the reviewer SHALL use a context/session separate from the author/implementer context
- **THEN** the review result SHALL be described as independently reviewed

### Requirement: Degraded review fallback
Degraded review MAY run without a separate context only when fresh context and a manual fresh session are unavailable. Degraded review MUST disclose the limitation, MUST receive human acknowledgement, and MUST NOT be described as independent review.

The fallback hierarchy is:

1. Separate/fresh context supplied by the harness (preferred)
2. User-managed separate clean session (fallback)
3. Degraded review (last resort)

Cross-model review remains optional and is NOT required for independence.

#### Scenario: User-managed fresh session
- **WHEN** the harness cannot supply a fresh context but the user can start a separate session
- **THEN** the user SHALL start a separate clean session for the review
- **THEN** the review result SHALL be described as independently reviewed

#### Scenario: Degraded review mode
- **WHEN** neither harness-supplied fresh context nor user-managed fresh session is possible
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
- **WHEN** a second plan-review follows round 1 due to REVISE, material APPROVE_WITH_CHANGES corrections, or another material change to reviewed inputs
- **THEN** the artifact SHALL record ROUND: 2
- **THEN** the artifact SHALL include a concise summary of the previous-round disposition

### Requirement: Structured review findings
Each review finding SHALL include a severity level (Critical, Moderate, Suggestion) and a category (compliance, quality, feasibility, scope, risk).

#### Scenario: Finding with severity and category
- **WHEN** the reviewer identifies an issue
- **THEN** the finding SHALL specify severity and category in the review output
