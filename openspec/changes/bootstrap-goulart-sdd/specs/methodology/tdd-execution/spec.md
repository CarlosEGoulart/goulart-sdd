## Purpose

Defines the TDD-oriented apply workflow — fresh implementer context per task, RED-GREEN-REFACTOR progression, and bounded retry loops.

## ADDED Requirements

### Requirement: One task at a time
The apply workflow SHALL implement one focused task at a time, providing the implementer with only: relevant specs, relevant design sections, test-plan context, and the specific task.

#### Scenario: Focused task context
- **WHEN** the implementer starts a task
- **THEN** the context SHALL include only the task description, relevant spec scenarios, relevant design decisions, and the test-plan entry for that task

### Requirement: TDD progression per task
Where automated testing is appropriate, each task SHALL follow RED-GREEN-REFACTOR: write failing test first, implement minimum behavior to pass, refactor while suite remains green.

#### Scenario: Test before implementation
- **WHEN** a task involves behavior that can be tested
- **THEN** the first sub-step SHALL be writing a failing test
- **THEN** the implementer SHALL confirm the test fails before implementing

### Requirement: Fresh context per task
Each task SHOULD start with fresh or minimal context where the coding-agent harness supports it, to prevent context pollution from prior tasks.

#### Scenario: Context isolation
- **WHEN** the implementer starts a new task
- **THEN** the context SHOULD NOT include implementation history from prior tasks

### Requirement: Bounded TDD retry
If the same test fails twice for the same reason within a single task, the implementer SHALL stop and report the issue rather than attempting a third approach.

#### Scenario: TDD stall detection
- **WHEN** a test fails twice consecutively for the same root cause
- **THEN** the implementer SHALL stop that task and surface the issue to the human

### Requirement: Spec drift handling
If implementation reveals a requirement or scenario is wrong, incomplete, or untestable, the implementer SHALL stop implementing that scenario and surface the issue. The implementer SHALL NOT silently edit the spec or weaken the test.

#### Scenario: Spec drift detected
- **WHEN** implementation reveals a spec scenario is incorrect or untestable
- **THEN** the implementer SHALL stop and report the conflict
- **THEN** the spec SHALL be amended before implementation resumes
