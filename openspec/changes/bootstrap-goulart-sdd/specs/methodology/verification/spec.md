## Purpose

Defines the final whole-change audit that verifies spec compliance, test integrity, review completeness, and readiness for archive.

## ADDED Requirements

### Requirement: goulart-verify produces the verify result
The `goulart-verify` adapter SHALL independently check implementation completeness before performing verification and producing the verify result. Its flow is:

```
check prerequisites
  → code-review exists and permits progression
  → required human triage is complete
  → all tasks in tasks.md are complete
  → required test-plan entries have a final evaluable state
  → review freshness checked; any permitted override explicitly recorded
  → perform verification
  → produce verify artifact
  → emit DECISION
```

Then `goulart-archive` consumes that decision. Do NOT describe goulart-verify's prerequisite check as "checking the verify decision" — the verify decision does not exist yet at that point.

This prerequisite checking is intentional defense in depth. Do not assume that the existence of `code-review.md` proves tasks were completed, because OpenSpec `requires:` enforces file existence/order, not semantic implementation completion.

A final evaluable test-plan state means each required AUTOMATED/MECHANICAL entry has a recorded completed check and result and each SEMANTIC entry has a documented evaluation; pending or unevaluated entries block verification. Evaluable does not mean passing: failures found during the audit produce FAIL. Degraded reviews require recorded disclosure and human acknowledgement and SHALL NOT be treated as independent.

The adapter SHALL assess verdict, triage, and staleness together. An explicitly permitted human override with mandatory reason may waive the identified review condition, but SHALL NOT make a stale review fresh or imply independent review occurred. Missing review artifacts, incomplete tasks, incomplete mandatory triage, or unevaluable test-plan entries SHALL still block verification.

#### Scenario: Verify prerequisites
- **WHEN** goulart-verify is invoked
- **THEN** it SHALL verify that code-review.md exists and has a verdict that permits progression
- **THEN** it SHALL verify that required human triage (where findings existed) is complete
- **THEN** it SHALL verify that all tasks in tasks.md are marked complete
- **THEN** it SHALL verify that required test-plan entries have a final evaluable state
- **THEN** it SHALL check review freshness and any explicit permitted override of the identified stale-review condition
- **THEN** only after all prerequisites permit continuation SHALL it perform verification and produce the verify artifact

#### Scenario: Verify blocked without code-review
- **WHEN** goulart-verify is invoked and code-review.md does not exist
- **THEN** the adapter SHALL refuse to proceed

#### Scenario: Verify blocked on incomplete tasks
- **WHEN** goulart-verify is invoked and tasks remain incomplete
- **THEN** the adapter SHALL refuse to proceed and report incomplete tasks

#### Scenario: Verify blocked on incomplete triage
- **WHEN** goulart-verify is invoked and mandatory review triage is incomplete
- **THEN** the adapter SHALL refuse to proceed

#### Scenario: Verify blocked on stale reviews
- **WHEN** goulart-verify is invoked and review state is obviously stale without an explicit permitted override with reason covering that condition
- **THEN** the adapter SHALL refuse to proceed and report which review is stale

#### Scenario: Verify blocked on review verdict
- **WHEN** the code-review verdict and recorded human dispositions do not permit progression
- **THEN** goulart-verify SHALL block and identify the unresolved verdict or finding

#### Scenario: Verify blocked on unevaluable test-plan entries
- **WHEN** required test-plan entries are pending or lack a completed check result or semantic evaluation
- **THEN** goulart-verify SHALL block and identify those entries

#### Scenario: Recorded review exception
- **WHEN** a permitted explicit human override waives re-review with a mandatory reason and all other prerequisites are satisfied
- **THEN** verification MAY proceed while recording the waived review condition
- **THEN** it SHALL NOT represent the prior review as covering materially changed inputs

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
The verify SHALL check review verdicts, dispositions, and freshness, distinguishing two review lineages and any explicitly permitted human override. Staleness detection uses repository revision/diff information where available, falling back to conservative semantic comparison when clean revision cannot represent the reviewed working tree. Filesystem timestamps SHALL NOT be used as reliable staleness evidence (unstable across clone, checkout, rebase, CI, file copy).

Any material change to proposal, specs, or design after the plan-review makes the plan-review stale and requires a new plan-review for normal progression. This remains true even when the material change was requested by the previous reviewer. Only non-material corrections may preserve freshness without re-review; the plan-review's explicit STATUS: OVERRIDDEN with mandatory reason is a recorded exception, not evidence of freshness. If verification observes a post-review change classified as non-material, it SHOULD verify that the review artifact records the rationale for that classification.

Each review artifact SHOULD record what it reviewed:
- reviewed revision/commit when available
- reviewed paths/artifacts

#### Scenario: Plan-review staleness
- **WHEN** verify runs
- **THEN** it SHALL check that proposal.md, specs/, and design.md were not materially modified after the plan-review verdict
- **THEN** material changes (including reviewer-requested material changes) SHALL require a new plan-review for normal progression, or the explicit permitted override recorded under plan-review semantics
- **THEN** non-material changes MAY avoid re-review if justified in the review artifact

#### Scenario: Code-review staleness
- **WHEN** verify runs
- **THEN** it SHALL check that source code, tests, or implementation-relevant configuration were not materially modified after the code-review verdict
- **THEN** material changes SHALL require a new code-review before normal progression, subject to bounded-round escalation and the explicit code-review override rules

#### Scenario: Accepted finding causes material implementation changes
- **WHEN** implementation is changed to address an accepted code-review finding and the change is material
- **THEN** verification SHALL consider the prior code-review stale
- **THEN** a new code-review SHALL occur before normal progression to verify; at the round limit the workflow SHALL stop and escalate instead

#### Scenario: Non-material implementation corrections
- **WHEN** post-review implementation corrections are classified as non-material
- **THEN** re-review MAY be skipped only with a clear justification recorded in the code-review artifact
- **THEN** verification SHALL assess that justification semantically

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
