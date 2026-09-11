## Purpose

Defines the test-plan artifact that maps specification scenarios to named automated tests and serves as a live coverage ledger.

## ADDED Requirements

### Requirement: Test-plan maps every scenario
The test-plan SHALL map every `#### Scenario:` in every spec file to at least one named test. No scenario may be left unmapped.

#### Scenario: Complete scenario coverage
- **WHEN** the test-plan is created
- **THEN** every scenario from every spec file SHALL have a corresponding entry in the test-plan

#### Scenario: Unmapped scenario is blocking
- **WHEN** a spec scenario has no corresponding test-plan entry
- **THEN** downstream tasks SHALL NOT be created until the gap is addressed

### Requirement: Test-plan entries include traceability
Each test-plan entry SHALL record: the requirement it belongs to, the scenario name, the test file path, and the test function/method name.

#### Scenario: Traceable entry
- **WHEN** a developer reads a test-plan entry
- **THEN** they can identify which spec scenario it covers and where the test lives

### Requirement: Test-plan as coverage ledger
The test-plan SHALL track test status: red (failing/not yet implemented) or green (passing). During apply, entries flip from red to green as tests pass.

#### Scenario: Status tracking during implementation
- **WHEN** a test implementation makes its test pass
- **THEN** the corresponding test-plan entry status SHALL be updated from red to green

### Requirement: Non-executable change handling
For changes with no executable test surface (documentation, config), scenarios MAY map to equivalent mechanical validations (linting, schema validation, link checking) instead of code tests.

#### Scenario: Mechanical validation mapping
- **WHEN** a change has no code test surface
- **THEN** scenarios MAY map to mechanical validation commands
- **THEN** the test-plan SHALL note the validation command and initial state as non-executable
