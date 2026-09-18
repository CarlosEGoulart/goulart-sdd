## Purpose

Handles interactive coding-agent selection and auto-detection during initialization.

## ADDED Requirements

### Requirement: Agent selection presents supported options
The initializer SHALL present a list of coding agents for the user to choose from. Only agents with installed adapters are functional; others are display-only placeholders for future support.

#### Scenario: Supported agents listed
- **WHEN** the user reaches the agent selection step
- **THEN** the CLI displays OpenCode as the functional option and any future agents as unimplemented placeholders marked experimental

#### Scenario: Unimplemented agent selected
- **WHEN** the user selects an agent without an installed adapter
- **THEN** the CLI prints an error explaining that the agent is not yet supported and exits with code 1

### Requirement: Auto-detect existing agent configuration
The initializer SHALL detect if the target repository already has a coding agent configured.

#### Scenario: OpenCode detected
- **WHEN** the target repository contains `.opencode/` directory
- **THEN** the initializer suggests OpenCode as the default selection

#### Scenario: No agent detected
- **WHEN** no coding agent configuration is found
- **THEN** the initializer presents the full selection list without a default

### Requirement: Selected agent is recorded
The selected coding agent SHALL be recorded in `.goulart/config.yaml`.

#### Scenario: Agent persisted
- **WHEN** the user selects a coding agent
- **THEN** the agent identifier is written to `.goulart/config.yaml` under an `agent` field
