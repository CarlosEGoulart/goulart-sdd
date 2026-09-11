## Purpose

Defines the test-plan artifact that maps specification scenarios to validation entries and serves as a behavioral coverage ledger.

## ADDED Requirements

### Requirement: Test-plan maps every scenario
The test-plan SHALL map every `#### Scenario:` in every spec file to at least one validation entry. No scenario may be left unmapped.

#### Scenario: Complete scenario coverage
- **WHEN** the test-plan is created
- **THEN** every scenario from every spec file SHALL have a corresponding entry in the test-plan

#### Scenario: Unmapped scenario is blocking
- **WHEN** a spec scenario has no corresponding test-plan entry
- **THEN** downstream tasks SHALL NOT be created until the gap is addressed

### Requirement: Three validation types
Each test-plan entry SHALL declare one of three validation types: AUTOMATED, MECHANICAL, or SEMANTIC.

#### Scenario: AUTOMATED entry
- **WHEN** a scenario can be verified by an automated test (unit, integration, E2E)
- **THEN** the entry SHALL record test file path and test function/method name
- **THEN** the validation type SHALL be AUTOMATED

#### Scenario: MECHANICAL entry
- **WHEN** a scenario can be verified by a tool or command (lint, typecheck, schema validation, build)
- **THEN** the entry SHALL record the validation command or check
- **THEN** the validation type SHALL be MECHANICAL

#### Scenario: SEMANTIC entry
- **WHEN** a scenario cannot reasonably be verified automatically or mechanically
- **THEN** the entry SHALL justify why automation is unsuitable
- **THEN** the entry SHALL identify who or what performs the semantic evaluation
- **THEN** the validation type SHALL be SEMANTIC

### Requirement: Test-plan as coverage ledger
The test-plan SHALL track implementation status for visibility. Statuses such as red/green are for tracking purposes and SHALL NOT be described as proof that RED chronologically occurred before GREEN.

#### Scenario: Status tracking during implementation
- **WHEN** a test implementation passes
- **THEN** the corresponding test-plan entry status SHALL be updated for visibility
- **THEN** the methodology SHALL NOT claim this proves TDD chronology

### Requirement: Behavioral coverage integrity
Tests SHALL NOT be removed, weakened, skipped, or replaced in a way that reduces required behavioral coverage without an explicitly justified specification change.

#### Scenario: Behavioral coverage preservation
- **WHEN** tests are modified during implementation
- **THEN** the test-plan SHALL confirm that required behavioral coverage is retained
- **THEN** any coverage reduction SHALL require a corresponding spec amendment
