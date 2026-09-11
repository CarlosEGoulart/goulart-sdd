## Purpose

Defines the TDD-oriented apply workflow — one-task-per-invocation baseline, fresh implementer context with codebase access, RED-GREEN-REFACTOR progression, and bounded retry loops.

## ADDED Requirements

### Requirement: One task per invocation (portable baseline)
The apply workflow SHALL implement one focused task per invocation. A subsequent invocation processes the next task. This ensures that a user can start each task in a fresh session even when the harness cannot spawn isolated subagents.

The per-invocation flow is:

```
select first eligible pending task
  → load relevant task/spec/design/test-plan context
  → allow repository exploration as needed
  → RED → GREEN → REFACTOR
  → mark that task complete
  → report result
  → STOP
```

If a harness supports true isolated subcontexts, an adapter MAY optimize execution by spawning one fresh subcontext per task. But the portable/default behavioral contract is one task per invocation. `goulart-apply` SHALL NOT run the entire task list in one accumulated conversational context by default.

#### Scenario: Single task execution
- **WHEN** the user invokes `goulart-apply`
- **THEN** the adapter SHALL select the first eligible pending task
- **THEN** the adapter SHALL execute that task using TDD
- **THEN** the adapter SHALL mark that task complete and report the result
- **THEN** the adapter SHALL STOP

#### Scenario: Subsequent task invocation
- **WHEN** the user invokes `goulart-apply` again after a task completes
- **THEN** the adapter SHALL select the next eligible pending task
- **THEN** the adapter SHALL execute that task
- **THEN** the adapter SHALL STOP

#### Scenario: All tasks complete
- **WHEN** `goulart-apply` is invoked and all tasks are complete
- **THEN** the adapter SHALL report completion
- **THEN** the adapter SHALL instruct the user to run `goulart-review code` in a fresh session

### Requirement: One task at a time with codebase access
The implementer SHALL receive minimal conversational/history context but SHALL be allowed to inspect all repository files necessary to implement the task correctly.

#### Scenario: Focused task with codebase access
- **WHEN** the implementer starts a task
- **THEN** the context SHALL include the task description, relevant spec scenarios, relevant design decisions, and the test-plan entry for that task
- **THEN** the implementer SHALL be able to inspect any repository file needed to understand implementation dependencies

### Requirement: TDD progression per task
Where automated testing is appropriate, each task SHALL follow RED-GREEN-REFACTOR: write failing test first, implement minimum behavior to pass, refactor while suite remains green.

#### Scenario: Test before implementation
- **WHEN** a task involves behavior that can be tested
- **THEN** the first sub-step SHALL be writing a failing test
- **THEN** the implementer SHALL confirm the test fails before implementing

### Requirement: Fresh context per task
Each task SHOULD start with fresh or minimal context where the coding-agent harness supports it, to prevent context pollution from prior tasks. The isolation goal is to reduce irrelevant reasoning/history contamination, not to blind the implementer to the repository.

#### Scenario: Context isolation
- **WHEN** the implementer starts a new task
- **THEN** the context SHOULD NOT include implementation history from prior tasks
- **THEN** the implementer SHALL still have access to the repository for discovering dependencies

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
