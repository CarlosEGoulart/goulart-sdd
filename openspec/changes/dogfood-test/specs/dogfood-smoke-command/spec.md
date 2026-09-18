## Purpose

Provides a temporary adapter command that exercises the Goulart SDD lifecycle with
a deterministic, non-mutating smoke result, validating the planning-to-archive
pipeline without affecting production code.

## ADDED Requirements

### Requirement: Deterministic smoke result
The `/goulart-dogfood-smoke` command SHALL return a fixed, predictable success
response on every invocation. The output MUST be identical across invocations
within the same adapter version.

#### Scenario: Successful smoke invocation
- **WHEN** a user invokes `/goulart-dogfood-smoke`
- **THEN** the command outputs a deterministic success message containing a
  fixed status code and the text `dogfood-smoke: OK`

### Requirement: No repository mutation
The `/goulart-dogfood-smoke` command SHALL NOT modify any file in the repository.
The command MUST NOT execute git operations, write configuration, or alter
any schema or artifact state.

#### Scenario: Repository unchanged after invocation
- **WHEN** a user invokes `/goulart-dogfood-smoke` on a clean working tree
- **THEN** `git status` reports no staged, modified, or untracked files
  after the command completes

### Requirement: Temporary fixture lifecycle
The `/goulart-dogfood-smoke` command SHALL be marked as temporary. Its removal
MUST be tracked as a discrete cleanup task in the implementing change.

#### Scenario: Fixture is identifiable as temporary
- **WHEN** a maintainer inspects the command file
- **THEN** the file contains a comment or header identifying it as a
  temporary dogfooding fixture scheduled for removal
