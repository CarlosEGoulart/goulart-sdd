## Purpose

Defines the post-implementation independent code review with human triage of findings.

## ADDED Requirements

### Requirement: Code review after task completion
A code-review SHALL be triggered after all tasks are marked complete and before verify. The `goulart-apply` adapter SHALL verify task completion before requesting code-review.

#### Scenario: Code review triggered after tasks
- **WHEN** all tasks in tasks.md are marked complete
- **THEN** the code-review SHALL be triggered before verification proceeds

#### Scenario: goulart-apply checks task completion
- **WHEN** the apply workflow reaches the code-review stage
- **THEN** it SHALL verify all tasks are complete before proceeding

### Requirement: Fresh context for code review
The code-reviewer SHOULD use a fresh context where the coding-agent harness supports it. The reviewer SHALL NOT be the same context that performed the implementation.

#### Scenario: Implementation context isolation
- **WHEN** the code-review is triggered
- **THEN** the skill or command SHALL instruct the agent to use a fresh context

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
When the code-review contains findings, the human SHALL triage each finding: accept, reject with justification, or defer.

#### Scenario: Human triages findings
- **WHEN** the code-review contains findings
- **THEN** the human SHALL review each finding and record a disposition
- **THEN** accepted findings SHALL be addressed before verify proceeds

### Requirement: Code review verdict
The code-review SHALL emit exactly one verdict: APPROVE, APPROVE_WITH_CHANGES, or REVISE.

#### Scenario: Critical finding requires REVISE
- **WHEN** the code review contains any Critical severity finding
- **THEN** the verdict SHALL be REVISE (not APPROVE)
