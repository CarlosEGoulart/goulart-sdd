## Purpose

Installs OpenCode-specific adapter files (commands, skills, configuration) into the target repository.

## ADDED Requirements

### Requirement: OpenCode adapter files are installed
The OpenCode adapter installer SHALL copy the required commands and skills into `.opencode/`.

#### Scenario: Commands installed
- **WHEN** the user selects OpenCode during init
- **THEN** `.opencode/commands/` contains all `goulart-*.md` command files

#### Scenario: Skills installed
- **WHEN** the user selects OpenCode during init
- **THEN** `.opencode/skills/` contains all `goulart-*` skill directories

### Requirement: Adapter files are versioned
The installer SHALL record the installed adapter version in `.goulart/config.yaml`.

#### Scenario: Version recorded
- **WHEN** OpenCode adapter files are installed
- **THEN** `.goulart/config.yaml` contains an `adapterVersion` field matching the package version

### Requirement: Existing adapter files are handled safely
The installer SHALL not silently overwrite user-modified adapter files.

#### Scenario: Modified file detected
- **WHEN** an adapter file already exists and differs from the package version
- **THEN** the installer prompts the user to choose between overwriting, keeping the existing file, or skipping
