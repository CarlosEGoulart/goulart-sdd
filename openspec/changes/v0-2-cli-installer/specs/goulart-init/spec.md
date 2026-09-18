## Purpose

Implements the `goulart init` command that interactively initializes Goulart SDD in a target repository.

## ADDED Requirements

### Requirement: init command scaffolds Goulart SDD
The `goulart init` command SHALL install Goulart SDD configuration and adapter files into the current or specified repository.

#### Scenario: Initialize in current directory
- **WHEN** a user runs `goulart init` in a git repository
- **THEN** the command creates `.goulart/` configuration and installs adapter files for the selected coding agent

#### Scenario: Initialize in specified directory
- **WHEN** a user runs `goulart init --dir /path/to/repo`
- **THEN** the command initializes Goulart SDD in the specified directory

#### Scenario: Not a git repository
- **WHEN** a user runs `goulart init` outside a git repository
- **THEN** the command prints an error explaining that a git repository is required and exits with code 1

### Requirement: init is interactive by default
The `goulart init` command SHALL prompt the user for coding-agent selection when no agent flag is provided.

#### Scenario: Interactive selection
- **WHEN** a user runs `goulart init` without specifying an agent
- **THEN** the command presents a selection prompt listing supported coding agents

#### Scenario: Agent specified via flag
- **WHEN** a user runs `goulart init --agent opencode`
- **THEN** the command skips the selection prompt and installs the OpenCode adapter

### Requirement: init creates project configuration
The init command SHALL create a `.goulart/config.yaml` file containing the schema reference and selected agent.

#### Scenario: Config file created
- **WHEN** `goulart init` completes successfully
- **THEN** `.goulart/config.yaml` exists and contains the schema name and selected agent identifier

#### Scenario: Config file preserved on re-init
- **WHEN** a user re-runs `goulart init` in an already-initialized repository
- **THEN** the command detects the existing configuration and prompts for confirmation before overwriting
