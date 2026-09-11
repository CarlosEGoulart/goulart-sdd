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

### Requirement: Namespace isolation for collision-prone identifiers
Namespace isolation applies primarily to collision-prone integration identifiers:

- Commands: `goulart-*`
- Skills: `goulart-*`
- Schema id/path: `goulart-sdd`
- Goulart-specific executable scripts where applicable

Normal project documentation (e.g., `docs/methodology.md`, `docs/getting-started.md`) MAY use standard naming without the `goulart-` prefix.

#### Scenario: Namespace separation for integration points
- **WHEN** a new Goulart SDD command, skill, or schema is created
- **THEN** it SHALL use the `goulart-` prefix and SHALL NOT collide with OpenSpec-generated names

#### Scenario: Documentation uses standard naming
- **WHEN** methodology documentation is created under `docs/`
- **THEN** it MAY use standard naming (e.g., `methodology.md`) without requiring the `goulart-` prefix

### Requirement: Schema ownership
The Goulart SDD custom schema SHALL live in `openspec/schemas/goulart-sdd/` and SHALL NOT modify the upstream `spec-driven` schema.

#### Scenario: Schema isolation
- **WHEN** the `goulart-sdd` schema is installed
- **THEN** the `spec-driven` schema SHALL remain available and unmodified

### Requirement: Adapter entry points
Goulart-owned adapter entry points (`goulart-plan`, `goulart-apply`, `goulart-review`, `goulart-verify`, `goulart-archive`) SHALL use the `goulart-*` namespace and SHALL delegate to OpenSpec operations without modifying upstream behavior.

#### Scenario: Adapter delegation
- **WHEN** a Goulart adapter needs OpenSpec to perform an operation
- **THEN** it SHALL call the OpenSpec operation (e.g., `openspec archive`) without modifying the upstream command or skill

### Requirement: CI gates are repository-owned
CI enforcement scripts and workflows SHALL live in `.github/workflows/` or `scripts/gates/` and SHALL NOT depend on OpenSpec internals.

#### Scenario: CI portability
- **WHEN** a repository gate runs
- **THEN** it SHALL validate repository state using only public file contents and structural invariants
