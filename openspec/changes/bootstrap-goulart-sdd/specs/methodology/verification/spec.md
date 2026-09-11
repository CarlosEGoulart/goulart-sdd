## Purpose

Defines the final whole-change audit that verifies spec compliance, test integrity, review completeness, and readiness for archive.

## ADDED Requirements

### Requirement: goulart-verify produces the verify result
The `goulart-verify` adapter produces/evaluates the verify result. Its flow is:

```
check prerequisites
  → code-review valid
  → review triage complete
  → implementation state ready
  → perform verification
  → produce verify artifact
  → emit DECISION
```

Then `goulart-archive` consumes that decision. Do NOT describe goulart-verify's prerequisite check as "checking the verify decision" — the verify decision does not exist yet at that point.

#### Scenario: Verify prerequisites
- **WHEN** goulart-verify is invoked
- **THEN** it SHALL verify that code-review.md exists and has a verdict
- **THEN** it SHALL verify that review triage (where findings existed) is complete
- **THEN** it SHALL proceed to perform verification and produce the verify artifact

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
The verify SHALL confirm that review verdicts are valid and not stale, distinguishing two review lineages. Staleness detection uses repository revision/diff information where available, falling back to conservative semantic comparison when clean revision cannot represent the reviewed working tree. Filesystem timestamps SHALL NOT be used as reliable staleness evidence (unstable across clone, checkout, rebase, CI, file copy).

Each review artifact SHOULD record what it reviewed:
- reviewed revision/commit when available
- reviewed paths/artifacts

#### Scenario: Plan-review staleness
- **WHEN** verify runs
- **THEN** it SHALL check that proposal.md, specs/, and design.md were not materially modified after the plan-review verdict (other than applying Required Changes)
- **THEN** material changes beyond Required Changes SHALL require a new plan-review

#### Scenario: Code-review staleness
- **WHEN** verify runs
- **THEN** it SHALL check that source code, tests, or implementation-relevant configuration were not materially modified after the code-review verdict
- **THEN** material changes SHALL require a new code-review before verify proceeds

#### Scenario: Revision-based staleness detection
- **WHEN** Git state is available
- **THEN** staleness SHOULD be checked using repository revision/diff information as supporting evidence

#### Scenario: Semantic fallback
- **WHEN** a clean revision cannot represent the reviewed working tree
- **THEN** staleness SHALL fall back to conservative semantic comparison

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
- **THEN** PASS_WITH_WARNINGS SHALL NOT silently mean PASS

#### Scenario: FAIL
- **WHEN** blocking issues are found
- **THEN** the decision SHALL be DECISION: FAIL
- **THEN** archive SHALL NOT proceed
- **THEN** human override MUST be explicit and recorded

### Requirement: Verify decision records reviewed state
The verify artifact SHOULD record the revision/commit and paths reviewed, so that goulart-archive and future audits can assess staleness.

#### Scenario: Review metadata
- **WHEN** verify produces the verify artifact
- **THEN** it SHOULD record the reviewed revision/commit (when available) and the reviewed paths/artifacts
