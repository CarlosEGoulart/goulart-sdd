## Purpose

Defines the Goulart-owned project configuration and state layout, separate from agent-specific files.

## ADDED Requirements

### Requirement: Goulart configuration directory
Goulart SDD SHALL use `.goulart/` as its project-owned configuration directory.

#### Scenario: Directory created on init
- **WHEN** `goulart init` completes
- **THEN** `.goulart/` exists at the repository root

#### Scenario: Directory not created outside init
- **WHEN** any Goulart command other than `init` is run
- **THEN** no `.goulart/` directory is created if it does not already exist

### Requirement: Configuration file format
The `.goulart/config.yaml` file SHALL contain the schema reference, selected agent, adapter version, and workflow engine identifier.

#### Scenario: Valid config structure
- **WHEN** `.goulart/config.yaml` is read
- **THEN** it contains at minimum: `schema`, `agent`, `adapterVersion`, and `workflowEngine` fields

### Requirement: Configuration is human-readable
The `.goulart/config.yaml` file SHALL be plain YAML that a developer can read and edit by hand.

#### Scenario: Hand-editable
- **WHEN** a developer edits `.goulart/config.yaml` manually
- **THEN** subsequent Goulart commands respect the edited values without corruption
