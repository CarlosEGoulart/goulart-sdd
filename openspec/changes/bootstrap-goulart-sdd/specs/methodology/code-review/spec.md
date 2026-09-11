## Purpose

Defines post-implementation code review, independent by default with a disclosed degraded fallback, and human triage of findings.

## ADDED Requirements

### Requirement: Code review after task completion
A code-review SHALL be triggered after all tasks are marked complete and before verify. The `goulart-apply` adapter SHALL verify task completion before instructing the user to run code-review.

#### Scenario: Code review triggered after tasks
- **WHEN** all tasks in tasks.md are marked complete
- **THEN** the code-review SHALL be triggered before verification proceeds

#### Scenario: goulart-apply checks task completion
- **WHEN** the apply workflow reaches the code-review stage
- **THEN** it SHALL verify all tasks are complete before instructing the user to run `goulart-review code`

### Requirement: Independent code review
Independent code review SHALL use a context/session separate from the implementer context.

#### Scenario: Independent review with fresh context
- **WHEN** an independent code-review is performed using a harness-supplied separate context or a user-managed separate clean session
- **THEN** the reviewer SHALL use a context/session separate from the implementer context
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

### Requirement: Code review evaluates implementation
The code-review SHALL evaluate: implementation against specs, implementation against design, scope compliance, test quality, behavioral coverage, code quality, maintainability, architecture, unnecessary complexity, and potential regressions.

#### Scenario: Implementation-spec compliance check
- **WHEN** the code-reviewer runs
- **THEN** it SHALL compare implementation behavior against spec scenarios
- **THEN** it SHALL report discrepancies as findings

#### Scenario: Scope creep detection
- **WHEN** the implementation contains changes beyond what the spec describes
- **THEN** the code-reviewer SHALL flag scope creep as a finding

### Requirement: Human triage of findings
The human SHALL triage each finding as ACCEPT, REJECT with justification, or DEFER with justification. Human triage and an additional human approval are NOT required for a clean review with no findings. Degraded-mode acknowledgement remains required when applicable, separately from finding triage.

#### Scenario: Human triages findings
- **WHEN** the code-review contains findings
- **THEN** the human SHALL review each finding and record a disposition (ACCEPT, REJECT with justification, or DEFER with justification)
- **THEN** accepted findings that require implementation changes SHALL be addressed before verify proceeds

#### Scenario: No findings — no triage required
- **WHEN** the code-review contains no findings
- **THEN** the human SHALL NOT be required to perform triage
- **THEN** the code-review verdict SHALL stand as-is

### Requirement: Code review verdict
The code-review SHALL emit exactly one verdict: APPROVE, APPROVE_WITH_CHANGES, or REVISE.

#### Verdict: APPROVE

No blocking implementation changes are required.

- If non-blocking findings exist, human triage is still required.
- After required triage is complete, verification may proceed if no code changes invalidate the review.

#### Verdict: APPROVE_WITH_CHANGES

The reviewer identifies at least one implementation change required before verification, subject to the explicit human triage rules below.

- Human triages findings.
- Accepted/required findings SHALL be addressed before verify. Material implementation changes make the code-review stale and require a new code-review round before normal progression to verify. Demonstrably non-material corrections MAY avoid re-review only with a recorded justification in the review artifact.
- If the human explicitly rejects or defers a finding: justification MUST be recorded, and progression is allowed only if no unresolved blocking/Critical condition remains.

#### Verdict: REVISE

Normal progression to verify is blocked.

- Implementation must be revised and code-review run again.
- Human override MAY exist only as an explicit recorded decision with mandatory reason identifying the waived review condition. Rejecting or deferring a finding is not implicitly an override of an unresolved blocking/Critical condition. An override SHALL NOT claim a stale review covers changed implementation or substitute for task completion, required triage, or evaluable test-plan entries.
- Maximum 2 review/revision rounds per review stage.
- After round 2 still has unresolved blocking conditions, required changes, or material changes requiring further review: STOP and escalate to human. Do not automatically start a third round or reset history. The limit includes re-reviews after APPROVE_WITH_CHANGES or other material implementation changes.

#### Scenario: APPROVE permits verification after triage
- **WHEN** the verdict is APPROVE, any required triage is complete, and no material changes invalidate the review
- **THEN** verification MAY proceed subject to its implementation-completeness prerequisites

#### Scenario: Required changes are triaged
- **WHEN** the verdict is APPROVE_WITH_CHANGES
- **THEN** human triage SHALL resolve every finding and accepted/required changes SHALL be addressed
- **THEN** material implementation changes SHALL require another review before normal progression; justified non-material corrections MAY avoid re-review

#### Scenario: Rejected or deferred finding
- **WHEN** the human rejects or defers a code-review finding
- **THEN** the justification SHALL be recorded
- **THEN** normal progression SHALL remain blocked while any unresolved blocking/Critical condition remains

#### Scenario: REVISE blocks normal progression
- **WHEN** the verdict is REVISE
- **THEN** implementation SHALL be revised and reviewed again before normal progression to verify
- **THEN** any human override SHALL be explicit and recorded with mandatory reason

#### Scenario: Round-two escalation
- **WHEN** round 2 leaves unresolved blocking conditions, required changes, or material changes requiring further review
- **THEN** the workflow SHALL stop and escalate to the human instead of automatically starting round 3

#### Scenario: Critical finding requires REVISE
- **WHEN** the code review contains any Critical severity finding
- **THEN** the verdict SHALL be REVISE (not APPROVE)

#### Scenario: No findings — no unconditional approval needed
- **WHEN** the code-review has no findings
- **THEN** the verdict SHALL be APPROVE
- **THEN** no unconditional human approval is required beyond the verdict standing as-is

### Requirement: Implementation changes invalidate code-review
When implementation is changed to address an accepted code-review finding, the prior code-review SHALL be considered stale if the change is material. A new code-review SHALL occur before normal progression to verify, subject to the bounded-round escalation and explicit override rules above. Non-material changes may avoid another review only when clearly justified in the review artifact.

#### Scenario: Material implementation change after code-review
- **WHEN** implementation is changed to address an accepted code-review finding
- **AND** the change is material
- **THEN** the prior code-review SHALL be considered stale
- **THEN** a new code-review round SHALL occur before normal progression to verify; at the two-round limit the workflow SHALL stop and escalate instead

#### Scenario: Non-material change after code-review
- **WHEN** implementation is changed to address an accepted code-review finding
- **AND** the change is demonstrably non-material
- **THEN** the code-review MAY remain valid
- **THEN** the non-material classification MUST be justified

### Requirement: Code-review-round persistence
Each code-review artifact SHALL preserve enough information to know the current round and previous outcome.

- `ROUND: 1 | 2`
- Concise previous-round disposition/history inside the same artifact.

The artifact SHALL NOT simply overwrite the review with no evidence of which round is being performed.

#### Scenario: First round
- **WHEN** the code-review is performed for the first time
- **THEN** the artifact SHALL record ROUND: 1

#### Scenario: Second round after revision
- **WHEN** a second code-review follows round 1 due to REVISE, material APPROVE_WITH_CHANGES corrections, or another material implementation change
- **THEN** the artifact SHALL record ROUND: 2
- **THEN** the artifact SHALL include a concise summary of the previous-round disposition
