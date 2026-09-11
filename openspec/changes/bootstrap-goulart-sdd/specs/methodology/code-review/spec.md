## Purpose

Defines the post-implementation independent code review that evaluates implementation against specs and design.

## ADDED Requirements

### Requirement: Code review after apply
A code-review artifact SHALL exist after all tasks are complete and before verify.

#### Scenario: Review after implementation
- **WHEN** all tasks in tasks.md are marked complete
- **THEN** the code-review SHALL be triggered before verification proceeds

### Requirement: Fresh context for code review
The code-reviewer SHOULD use a fresh context where the coding-agent harness supports it. The reviewer SHALL NOT be the same context that performed the implementation.

#### Scenario: Implementation context isolation
- **WHEN** the code-review is triggered
- **THEN** the skill or command SHALL instruct the agent to use a fresh context

### Requirement: Code review evaluates implementation
The code-review SHALL evaluate: implementation against specs, implementation against design, scope compliance, test quality, code quality, and potential regressions.

#### Scenario: Implementation-spec compliance check
- **WHEN** the code-reviewer runs
- **THEN** it SHALL compare implementation behavior against spec scenarios
- **THEN** it SHALL report discrepancies as findings

#### Scenario: Scope creep detection
- **WHEN** the implementation contains changes beyond what the spec describes
- **THEN** the code-reviewer SHALL flag scope creep as a finding

### Requirement: Code review verdict
The code-review SHALL emit exactly one verdict: APPROVE, APPROVE_WITH_CHANGES, or REVISE.

#### Scenario: Critical finding requires REVISE
- **WHEN** the code review contains any Critical severity finding
- **THEN** the verdict SHALL be REVISE (not APPROVE)
