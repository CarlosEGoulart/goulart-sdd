## Purpose

Defines the strict Goulart SDD workflow lifecycle — the ordered sequence of artifacts, gates, and operations from proposal through archive.

## ADDED Requirements

### Requirement: Strict workflow sequence
Goulart SDD SHALL enforce the following artifact sequence for changes using the `goulart-sdd` schema: proposal, specs, design, plan-review, test-plan, tasks, apply, code-review, verify, archive.

#### Scenario: Workflow blocks out-of-order artifacts
- **WHEN** an agent attempts to create an artifact before its dependencies are satisfied
- **THEN** the workflow SHALL NOT proceed until all upstream dependencies exist

#### Scenario: Workflow blocks apply without review
- **WHEN** the apply operation is invoked and plan-review does not exist or its verdict is not APPROVE
- **THEN** the workflow SHALL refuse to proceed with implementation

### Requirement: Three gate categories
The workflow SHALL distinguish artifact gates (OpenSpec dependency graph), execution gates (Goulart commands/skills), and repository gates (scripts/CI).

#### Scenario: Artifact gate enforcement
- **WHEN** artifact B requires artifact A
- **THEN** OpenSpec's schema engine SHALL prevent creation of B until A exists

#### Scenario: Execution gate enforcement
- **WHEN** a Goulart skill or command checks a pre-implementation gate
- **THEN** it SHALL verify the required artifact state before allowing execution to proceed

#### Scenario: Repository gate enforcement
- **WHEN** a CI check or script validates repository state
- **THEN** it SHALL verify structural invariants (required artifacts exist, verdicts are present) without claiming to prove agent cognition

### Requirement: Bounded review iteration
The workflow SHALL enforce a maximum of 2 review-revision rounds per review stage (plan-review and code-review independently).

#### Scenario: Review escalation after two rounds
- **WHEN** a review stage reaches its second REVISE verdict
- **THEN** the workflow SHALL STOP and escalate to the human with a clear summary of unresolved issues

### Requirement: Human override authority
The human SHALL retain final authority to approve, reject, defer, or override any advisory model conclusion at any gate.

#### Scenario: Human overrides reviewer
- **WHEN** the human disagrees with a reviewer's REVISE verdict
- **THEN** the human MAY instruct the workflow to proceed despite the verdict, and the decision SHALL be recorded
