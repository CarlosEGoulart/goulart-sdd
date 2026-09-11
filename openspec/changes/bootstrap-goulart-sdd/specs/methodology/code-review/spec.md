## Purpose

Defines the post-implementation independent code review with human triage of findings.

## ADDED Requirements

### Requirement: Code review after task completion
A code-review SHALL be triggered after all tasks are marked complete and before verify. The `goulart-apply` adapter SHALL verify task completion before instructing the user to run code-review.

#### Scenario: Code review triggered after tasks
- **WHEN** all tasks in tasks.md are marked complete
- **THEN** the code-review SHALL be triggered before verification proceeds

#### Scenario: goulart-apply checks task completion
- **WHEN** the apply workflow reaches the code-review stage
- **THEN** it SHALL verify all tasks are complete before instructing the user to run `goulart-review code`

### Requirement: Fresh context for code review with fallback hierarchy
The code-reviewer SHOULD use a fresh context. The reviewer SHALL NOT be the same context that performed the implementation.

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
- **THEN** the code-reviewer SHALL use a fresh context

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
When the code-review contains findings, the human SHALL triage each finding: ACCEPT, REJECT with justification, or DEFER with justification. Human triage is NOT required when the code-review has no findings.

#### Scenario: Human triages findings
- **WHEN** the code-review contains findings
- **THEN** the human SHALL review each finding and record a disposition
- **THEN** accepted findings that require implementation changes SHALL be addressed before verify proceeds

#### Scenario: No findings — no triage required
- **WHEN** the code-review contains no findings
- **THEN** the human SHALL NOT be required to perform triage
- **THEN** the code-review verdict SHALL stand as-is

### Requirement: Code review verdict
The code-review SHALL emit exactly one verdict: APPROVE, APPROVE_WITH_CHANGES, or REVISE.

#### Scenario: Critical finding requires REVISE
- **WHEN** the code review contains any Critical severity finding
- **THEN** the verdict SHALL be REVISE (not APPROVE)

### Requirement: Code-review-round persistence
Each code-review artifact SHALL preserve enough information to know the current round and previous outcome.

- `ROUND: 1 | 2`
- Concise previous-round disposition/history inside the same artifact.

The artifact SHALL NOT simply overwrite the review with no evidence of which round is being performed.

#### Scenario: First round
- **WHEN** the code-review is performed for the first time
- **THEN** the artifact SHALL record ROUND: 1

#### Scenario: Second round after revision
- **WHEN** the code-review is performed after code was revised following a REVISE or APPROVE_WITH_CHANGES verdict
- **THEN** the artifact SHALL record ROUND: 2
- **THEN** the artifact SHALL include a concise summary of the previous-round disposition
