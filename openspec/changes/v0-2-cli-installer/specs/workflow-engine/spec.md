## Purpose

Defines the WorkflowEngine abstraction that separates core lifecycle logic from workflow-backend execution.

## ADDED Requirements

### Requirement: WorkflowEngine interface exists
The Goulart core SHALL define a `WorkflowEngine` interface that abstracts workflow-backend operations.

#### Scenario: Interface defined
- **WHEN** the core package is imported
- **THEN** a `WorkflowEngine` type/interface is exported with methods for change lifecycle operations

### Requirement: Core does not call OpenSpec directly
Once the engine boundary exists, Goulart core, CLI, and agent adapters SHALL NOT invoke `openspec` commands directly.

#### Scenario: No direct openspec calls in core
- **WHEN** the core package source is inspected
- **THEN** no file contains direct `openspec` CLI invocations or imports of OpenSpec internals

#### Scenario: Engine delegation
- **WHEN** a lifecycle operation requires workflow execution
- **THEN** the operation is delegated to the configured `WorkflowEngine` implementation

### Requirement: Engine is configurable
The workflow engine implementation SHALL be configurable via `.goulart/config.yaml`.

#### Scenario: Engine field present
- **WHEN** `.goulart/config.yaml` is read
- **THEN** it contains a `workflowEngine` field identifying the active engine

#### Scenario: Default engine
- **WHEN** no `workflowEngine` field is present
- **THEN** the system defaults to `openspec` for v0.2
