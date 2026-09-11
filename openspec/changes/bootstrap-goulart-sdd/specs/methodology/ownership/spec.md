## Purpose

Defines clear ownership boundaries between OpenSpec upstream, Goulart SDD methodology, coding-agent adapters, and CI enforcement.

## ADDED Requirements

### Requirement: OpenSpec namespace preservation
Files under `.opencode/commands/opsx-*` and `.opencode/skills/openspec-*` SHALL NOT be manually modified by Goulart SDD. Running `openspec update` SHALL NOT destroy or overwrite Goulart SDD behavior.

#### Scenario: openspec update preserves Goulart files
- **WHEN** a user runs `openspec update`
- **THEN** all files under the `goulart-*` namespace SHALL remain unchanged

#### Scenario: OpenSpec files remain upstream-managed
- **WHEN** OpenSpec regenerates its skills or commands
- **THEN** Goulart SDD files under the `goulart-*` namespace SHALL NOT be affected

### Requirement: Goulart namespace isolation
All Goulart SDD-owned files SHALL use the `goulart-*` namespace prefix, including commands, skills, schemas, and scripts.

#### Scenario: Namespace separation
- **WHEN** a new Goulart SDD command, skill, or script is created
- **THEN** it SHALL use the `goulart-` prefix and SHALL NOT collide with OpenSpec-generated names

### Requirement: Schema ownership
The Goulart SDD custom schema SHALL live in `openspec/schemas/goulart-sdd/` and SHALL NOT modify the upstream `spec-driven` schema.

#### Scenario: Schema isolation
- **WHEN** the `goulart-sdd` schema is installed
- **THEN** the `spec-driven` schema SHALL remain available and unmodified

### Requirement: CI gates are repository-owned
CI enforcement scripts and workflows SHALL live in `.github/workflows/` or `scripts/gates/` and SHALL NOT depend on OpenSpec internals.

#### Scenario: CI portability
- **WHEN** a repository gate runs
- **THEN** it SHALL validate repository state using only public file contents and structural invariants
