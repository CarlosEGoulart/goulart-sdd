## Purpose

Provides automated tests for installation behavior, including temporary-repository E2E paths.

## ADDED Requirements

### Requirement: Init works in a temporary repository
The installation SHALL be testable by initializing Goulart in a temporary git repository.

#### Scenario: E2E init in temp repo
- **WHEN** the test creates a temporary git repository and runs `goulart init --agent opencode`
- **THEN** the repository contains `.goulart/config.yaml`, `.opencode/commands/goulart-*.md`, and `.opencode/skills/goulart-*/`

### Requirement: Init is idempotent in tests
Running init twice in the same temporary repository SHALL produce identical results.

#### Scenario: Idempotent re-init
- **WHEN** the test runs `goulart init --agent opencode` twice in the same temp repo
- **THEN** all files are byte-identical after the second run

### Requirement: Config structure is valid
The installed `.goulart/config.yaml` SHALL parse as valid YAML with required fields.

#### Scenario: Config validation
- **WHEN** init completes in a temp repo
- **THEN** `.goulart/config.yaml` contains `schema`, `agent`, and `adapterVersion` fields with non-empty values

### Requirement: Adapter files match package source
Installed adapter files SHALL match the files from the current package version.

#### Scenario: File integrity
- **WHEN** init installs OpenCode adapter files
- **THEN** each installed file matches the corresponding source file in the package
