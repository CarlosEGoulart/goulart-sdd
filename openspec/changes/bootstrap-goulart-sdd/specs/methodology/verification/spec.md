## Purpose

Defines the final whole-change audit that verifies spec compliance, test integrity, review completeness, and readiness for archive.

## ADDED Requirements

### Requirement: Verify depends on code-review
The verify artifact SHALL exist after code-review and before archive. The `goulart-verify` adapter SHALL require code-review to exist before proceeding.

#### Scenario: Archive blocked without verify
- **WHEN** the archive operation is invoked and verify.md does not exist
- **THEN** the workflow SHALL refuse to proceed

#### Scenario: Verify blocked without code-review
- **WHEN** goulart-verify is invoked and code-review.md does not exist
- **THEN** the adapter SHALL refuse to proceed

### Requirement: Verify checks spec compliance
The verify SHALL confirm that the implementation satisfies all requirements and scenarios in the approved specs.

#### Scenario: Spec compliance check
- **WHEN** verify runs
- **THEN** it SHALL compare implementation against each spec scenario
- **THEN** it SHALL report any unmet scenarios as findings

### Requirement: Verify checks test integrity
The verify SHALL confirm that all test-plan entries are satisfied (AUTOMATED entries pass, MECHANICAL entries pass, SEMANTIC entries are documented), and that the full test suite passes.

#### Scenario: Test integrity check
- **WHEN** verify runs
- **THEN** it SHALL check that no AUTOMATED or MECHANICAL test-plan entry fails
- **THEN** it SHALL confirm all SEMANTIC entries have documented evaluations
- **THEN** it SHALL run the full test suite and confirm it passes

#### Scenario: Behavioral coverage preservation
- **WHEN** verify runs
- **THEN** it SHALL confirm no tests were removed, weakened, or replaced in a way that reduces behavioral coverage without a corresponding spec change

### Requirement: Verify checks review completeness and staleness
The verify SHALL confirm that review verdicts are valid and not stale, distinguishing two review lineages.

#### Scenario: Plan-review staleness
- **WHEN** verify runs
- **THEN** it SHALL check that proposal.md, specs/, and design.md were not materially modified after the plan-review verdict (other than applying Required Changes)
- **THEN** material changes beyond Required Changes SHALL require a new plan-review

#### Scenario: Code-review staleness
- **WHEN** verify runs
- **THEN** it SHALL check that source code, tests, or implementation-relevant configuration were not materially modified after the code-review verdict
- **THEN** material changes SHALL require a new code-review before verify proceeds

### Requirement: Verify emits structured decision
The verify SHALL emit exactly one decision: PASS, PASS_WITH_WARNINGS, or FAIL. It SHALL clearly distinguish mechanical checks from semantic evaluations.

#### Scenario: PASS
- **WHEN** all checks pass with no issues
- **THEN** the decision SHALL be DECISION: PASS
- **THEN** archive MAY proceed

#### Scenario: PASS_WITH_WARNINGS
- **WHEN** checks pass but non-blocking warnings exist
- **THEN** the decision SHALL be DECISION: PASS_WITH_WARNINGS
- **THEN** warnings SHALL be surfaced to the human
- **THEN** archive MAY proceed only if the human explicitly accepts or defers the warnings

#### Scenario: FAIL
- **WHEN** blocking issues are found
- **THEN** the decision SHALL be DECISION: FAIL
- **THEN** archive SHALL NOT proceed
- **THEN** human override MUST be explicit and recorded
