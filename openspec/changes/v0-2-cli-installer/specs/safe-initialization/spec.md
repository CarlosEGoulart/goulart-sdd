## Purpose

Ensures initialization is safe and idempotent — re-running must not silently destroy user changes.

## ADDED Requirements

### Requirement: Re-init detects existing installation
The initializer SHALL detect an existing Goulart installation and inform the user.

#### Scenario: Already initialized
- **WHEN** a user runs `goulart init` in an already-initialized repository
- **THEN** the command prints that Goulart is already installed and asks whether to re-initialize

### Requirement: User changes are preserved
The initializer SHALL NOT silently overwrite files that the user has modified.

#### Scenario: Modified config preserved
- **WHEN** `.goulart/config.yaml` has been manually edited
- **THEN** re-init prompts the user before overwriting and shows the diff

#### Scenario: Modified adapter files preserved
- **WHEN** an adapter command or skill file has been modified
- **THEN** re-init prompts the user for that file specifically

### Requirement: Init is idempotent for unmodified state
The initializer SHALL produce identical output when run twice without user changes in between.

#### Scenario: Clean re-init
- **WHEN** a user runs `goulart init` twice without modifying any files
- **THEN** the second run completes without prompts and the repository state is identical
