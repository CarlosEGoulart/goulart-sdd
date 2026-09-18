## Purpose

Implements the OpenSpec backend behind the WorkflowEngine interface for v0.2.

## ADDED Requirements

### Requirement: OpenSpecEngine implements WorkflowEngine
An `OpenSpecEngine` class or module SHALL implement the `WorkflowEngine` interface.

#### Scenario: Engine instantiation
- **WHEN** the system resolves the workflow engine from config
- **THEN** it creates an `OpenSpecEngine` instance when the config specifies `openspec`

### Requirement: OpenSpecEngine delegates to OpenSpec CLI
The `OpenSpecEngine` SHALL delegate lifecycle operations to the `openspec` CLI.

#### Scenario: Plan delegation
- **WHEN** a planning operation is invoked
- **THEN** `OpenSpecEngine` executes the corresponding `openspec` command and returns the result

#### Scenario: OpenSpec not installed
- **WHEN** the `openspec` CLI is not available on the system
- **THEN** `OpenSpecEngine` returns a clear error indicating OpenSpec is required

### Requirement: OpenSpec-specific paths are isolated
OpenSpec-specific file paths and formats SHALL NOT leak into core, CLI, or adapter code.

#### Scenario: Path isolation
- **WHEN** core or adapter code references workflow files
- **THEN** references go through the engine interface, not direct `openspec/` paths
