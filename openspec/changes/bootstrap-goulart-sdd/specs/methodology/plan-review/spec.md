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
- **WHEN** the plan-review verdict is APPROVE but no human decision has been recorded
- **THEN** the workflow SHALL refuse to proceed until the human records a disposition

### Requirement: Human decision after plan-review
After the plan-review verdict, the human SHALL record a disposition: ACCEPTED, REVISE, or OVERRIDDEN. Reviewer approval alone is NOT sufficient to proceed.

#### Scenario: Human accepts plan
- **WHEN** the plan-review verdict is APPROVE
- **THEN** the human SHALL record STATUS: ACCEPTED with optional reason
- **THEN** downstream planning may proceed

#### Scenario: Human requests revision
- **WHEN** the human is unsatisfied with the plan
- **THEN** the human SHALL record STATUS: REVISE with reason
- **THEN** artifacts SHALL be revised and re-reviewed

#### Scenario: Human overrides reviewer
- **WHEN** the plan-review verdict is REVISE but the human chooses to proceed
- **THEN** the human SHALL record STATUS: OVERRIDDEN with mandatory reason
- **THEN** downstream planning may proceed

### Requirement: Fresh context for review
The plan-reviewer SHOULD use a fresh context where the coding-agent harness supports it. The reviewer SHALL NOT be the same context that authored the proposal, specs, or design.

#### Scenario: Fresh context instruction
- **WHEN** a plan-review is triggered
- **THEN** the skill or command SHALL instruct the agent to use a fresh context or cross-model reviewer

### Requirement: Reviewer read-only posture
The plan-reviewer SHALL NOT modify proposal.md, specs/, or design.md during the review. The reviewer's ONLY permitted write is the review output file.

#### Scenario: Reviewer does not modify artifacts
- **WHEN** the plan-reviewer runs
- **THEN** it SHALL only read proposal.md, specs/, design.md, and relevant source files
- **THEN** it SHALL only write to the review output file

### Requirement: Structured review findings
Each review finding SHALL include a severity level (Critical, Moderate, Suggestion) and a category (compliance, quality, feasibility, scope, risk).

#### Scenario: Finding with severity and category
- **WHEN** the reviewer identifies an issue
- **THEN** the finding SHALL specify severity and category in the review output

### Requirement: Review verdict
The plan-review SHALL emit exactly one verdict: APPROVE, APPROVE_WITH_CHANGES, or REVISE.

#### Scenario: Critical finding requires REVISE
- **WHEN** the review contains any Critical severity finding
- **THEN** the verdict SHALL be REVISE (not APPROVE)
