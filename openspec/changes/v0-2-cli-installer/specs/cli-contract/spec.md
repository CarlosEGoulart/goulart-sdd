## Purpose

Defines the public CLI interface for Goulart SDD — command names, flags, output format, and error behavior.

## ADDED Requirements

### Requirement: CLI package is installable via npm
The `goulart-sdd` package SHALL be installable via `npm install -g goulart-sdd` or `npx goulart-sdd`.

#### Scenario: Global install
- **WHEN** a user runs `npm install -g goulart-sdd`
- **THEN** the `goulart` command becomes available on the system PATH

#### Scenario: npx execution
- **WHEN** a user runs `npx goulart-sdd init`
- **THEN** the init command executes without requiring a global install

### Requirement: CLI exposes goulart command
The CLI SHALL expose a `goulart` command (or `goulart-sdd` alias) that accepts subcommands.

#### Scenario: Help output
- **WHEN** a user runs `goulart --help`
- **THEN** the CLI prints available subcommands and a brief description of each

#### Scenario: Version output
- **WHEN** a user runs `goulart --version`
- **THEN** the CLI prints the current version number

### Requirement: CLI reports errors clearly
The CLI SHALL print actionable error messages to stderr and exit with a non-zero code on failure.

#### Scenario: Unknown subcommand
- **WHEN** a user runs `goulart unknown-cmd`
- **THEN** the CLI prints an error listing available commands and exits with code 1

#### Scenario: Missing required input
- **WHEN** a required flag or argument is missing
- **THEN** the CLI prints a message explaining the missing input and exits with code 1
