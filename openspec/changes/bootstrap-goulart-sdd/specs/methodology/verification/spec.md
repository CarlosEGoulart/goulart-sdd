## Purpose

Defines the final whole-change audit that verifies spec compliance, test integrity, review completeness, and readiness for archive.

## ADDED Requirements

### Requirement: Verify is mandatory
The verify artifact SHALL exist before archive may proceed.

#### Scenario: Archive blocked without verify
- **WHEN** the archive operation is invoked and verify.md does not exist
- **THEN** the workflow SHALL refuse to proceed

### Requirement: Verify checks spec compliance
The verify SHALL confirm that the implementation satisfies all requirements and scenarios in the approved specs.

#### Scenario: Spec compliance check
- **WHEN** verify runs
- **THEN** it SHALL compare implementation against each spec scenario
- **THEN** it SHALL report any unmet scenarios as findings

### Requirement: Verify checks test integrity
The verify SHALL confirm that all test-plan entries are green (or documented non-executable with passing validation), and that the full test suite passes.

#### Scenario: Test integrity check
- **WHEN** verify runs
- **THEN** it SHALL check that no test-plan entry remains red
- **THEN** it SHALL run the full test suite and confirm it passes

#### Scenario: No weakened tests
- **WHEN** verify runs
- **THEN** it SHALL confirm no tests were weakened or deleted without a corresponding REMOVED requirement

### Requirement: Verify checks review completeness
The verify SHALL confirm that review verdicts are valid and not stale (artifacts were not modified after the verdict was issued beyond applying listed Required Changes).

#### Scenario: Review staleness check
- **WHEN** verify runs
- **THEN** it SHALL check that proposal.md, specs/, and design.md were not modified after the last review verdict (other than applying Required Changes)

### Requirement: Verify emits structured decision
The verify SHALL emit exactly one decision: PASS, PASS_WITH_WARNINGS, or FAIL. It SHALL clearly distinguish mechanical checks from semantic evaluations.

#### Scenario: Structured decision output
- **WHEN** verify completes
- **THEN** the decision SHALL be machine-readable (DECISION: PASS / PASS_WITH_WARNINGS / FAIL)
- **THEN** each finding SHALL note whether it came from a mechanical check or semantic evaluation
