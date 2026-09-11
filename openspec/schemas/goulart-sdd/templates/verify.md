# Verify

<!-- Final whole-change verification artifact. -->
<!-- Distinguishes prerequisite blocking from final audit failure. -->

## Prerequisites Checked

<!-- All prerequisites must be SATISFIED before verification proceeds. -->
<!-- SATISFIED = condition met; BLOCKED = condition unmet, verification cannot proceed. -->
<!-- Blocked prerequisites do NOT produce a FAIL decision — verification simply does not run. -->

| Prerequisite | Status | Evidence | Blocking Gap |
|---|---|---|---|
| Code review exists | | | |
| Code review verdict permits progression | | | |
| Mandatory finding triage complete | | | |
| All tasks complete | | | |
| Test-plan entries evaluable | | | |
| Review staleness assessed | | | |
| Degraded-review disclosure/acknowledgement | | | |
| Review override (if applicable) | | | |

<!-- Status: SATISFIED | BLOCKED -->
<!-- Required prerequisites that are BLOCKED cannot be waived by a review override. -->
<!-- Missing code-review, incomplete tasks, incomplete mandatory triage, and -->
<!-- unevaluable test-plan entries are mandatory and non-waivable. -->

---

## Reviewed State

**Reviewed Revision:** _______________
<!-- Repository revision/commit audited, if available -->

**Base Revision:** _______________
<!-- Where useful for diff comparison -->

**Timestamp (informational only):** _______________
<!-- Informational only — MUST NOT be used as freshness evidence -->

**Reviewed Paths / Artifacts:**

- [ ] proposal
- [ ] specs
- [ ] design
- [ ] tasks
- [ ] test-plan
- [ ] code-review
- [ ] implementation / source
- [ ] tests
- [ ] implementation-relevant configuration

<!-- Record the state that was actually audited. -->

---

## Task Completion

**Total Tasks:** ___
**Completed Tasks:** ___
**Incomplete Tasks:** ___

**Evidence:** _______________
<!-- Path or reference to tasks.md -->

<!-- Unfinished mandatory tasks block verification BEFORE the audit runs. -->
<!-- Existence of code-review.md does NOT prove tasks are complete. -->

---

## Spec Compliance

| Spec Path | Requirement | Scenario | Result | Evidence/Finding |
|---|---|---|---|---|
| | | | | |

<!-- Result: SATISFIED | UNMET -->
<!-- Any unmet required scenario is reportable as a finding. -->

---

## Test Integrity

### Validation Results

| Entry | Type | Validation | Result | Evidence |
|---|---|---|---|---|
| | | | | |

<!-- Type: AUTOMATED | MECHANICAL | SEMANTIC -->
<!-- AUTOMATED/MECHANICAL: completed check with result -->
<!-- SEMANTIC: documented evaluation -->

<!-- Verification must establish: -->
<!-- - no required AUTOMATED entry fails -->
<!-- - no required MECHANICAL entry fails -->
<!-- - every SEMANTIC entry has documented evaluation -->
<!-- Evaluable does NOT mean passing — failures produce FAIL during audit. -->

### Full Test Suite

**Command:** _______________
**Result:** _______________
**Evidence:** _______________

### Behavioral Coverage Integrity

**Coverage Reduction Detected:** Yes | No

| Entry | Change | Coverage Impact | Spec Amendment | Rationale |
|---|---|---|---|---|
| | | | | |

<!-- Tests/validations must NOT be removed, weakened, skipped, or replaced -->
<!-- in a way that reduces required behavioral coverage unless there is an -->
<!-- explicitly justified corresponding specification change. -->
<!-- Unauthorized coverage reduction is a blocking finding. -->

---

## Review Staleness

<!-- Two distinct lineages — do NOT collapse into one freshness flag. -->

### Plan Review Staleness

**Reviewed Revision:** _______________
**Reviewed Paths:** proposal, specs, design

**Changes After Review:** Yes | No
**Affected Paths:** _______________
<!-- Paths/artifacts modified after the plan-review verdict -->

**Materiality:** MATERIAL | NON_MATERIAL
<!-- MATERIAL = changes affect requirements, scope, architecture, acceptance criteria, or implementation assumptions -->
<!-- NON_MATERIAL = corrections that do not change requirements, scope, architecture, acceptance criteria, or implementation assumptions -->

**Materiality Rationale:** _______________
<!-- Required for both MATERIAL and NON_MATERIAL classifications -->

**Assessment Method:** REVISION_DIFF | SEMANTIC_FALLBACK
<!-- REVISION_DIFF = repository revision/diff evidence available -->
<!-- SEMANTIC_FALLBACK = conservative semantic comparison when clean revision cannot represent reviewed working tree -->

**Assessment Evidence:** _______________
<!-- Diff reference, commit range, or semantic comparison rationale -->

**Freshness Result:** FRESH | STALE
<!-- FRESH = no material changes detected -->
<!-- STALE = material changes detected, re-review required for normal progression -->

**Re-review Required:** Yes | No

**Override Invoked:** Yes | No
<!-- Override is an exception to the gate; it does NOT change STALE to FRESH -->
<!-- It records a permitted exception allowing progression despite the stale condition -->

**Condition Waived:** _______________
<!-- What the override explicitly permits -->

**Override Reason:** _______________
<!-- Mandatory when override is invoked -->

<!-- Material changes to proposal/specs/design after plan-review make it STALE. -->
<!-- Reviewer-requested material changes are still MATERIAL and still make it STALE. -->
<!-- NON_MATERIAL corrections may preserve normal progression only with recorded rationale. -->
<!-- An override does NOT make a stale review fresh. -->
<!-- An override does NOT imply independent review occurred. -->
<!-- Filesystem timestamps MUST NOT be freshness evidence. -->

### Code Review Staleness

**Reviewed Revision:** _______________
**Reviewed Paths:** source, tests, configuration

**Changes After Review:** Yes | No
**Affected Paths:** _______________

**Materiality:** MATERIAL | NON_MATERIAL

**Materiality Rationale:** _______________

**Assessment Method:** REVISION_DIFF | SEMANTIC_FALLBACK

**Assessment Evidence:** _______________

**Freshness Result:** FRESH | STALE

**Re-review Required:** Yes | No

**Accepted Finding Caused Material Change:** Yes | No
<!-- If an accepted finding caused material implementation changes, -->
<!-- the prior code review is STALE regardless of acceptance. -->

**Round Limit State:** _______________
<!-- Record bounded-round escalation state if relevant -->

**Override Invoked:** Yes | No

**Condition Waived:** _______________

**Override Reason:** _______________

<!-- Source/test/config changes after code review make it STALE. -->
<!-- Material fixes for accepted findings still make the prior review STALE. -->
<!-- NON_MATERIAL corrections may avoid re-review only with clear recorded justification. -->
<!-- Override does NOT imply freshness. -->
<!-- Override does NOT imply independent review occurred. -->

---

## Review Overrides

| Review | Condition Waived | Override Invoked | Reason | Freshness Result |
|---|---|---|---|---|
| | | | | |

<!-- Freshness Result: FRESH | STALE -->
<!-- If an override covers a stale review, Freshness Result must remain STALE. -->
<!-- Override permits only the explicitly allowed exception. -->
<!-- It does not change freshness classification. -->
<!-- Override does NOT replace: code-review existence, task completion, -->
<!-- mandatory triage, or evaluable test-plan entries. -->

---

## Scope Drift

| Path/Area | Drift | Description | Impact |
|---|---|---|---|
| | | | |

<!-- Drift: NONE | DETECTED -->
<!-- Impact: NON_BLOCKING | BLOCKING -->
<!-- Blocking scope drift contributes to DECISION: FAIL -->

---

## Findings / Warnings

<!-- Separates blocking findings from non-blocking warnings. -->

| ID | Type | Area | Description | Evidence |
|---|---|---|---|---|
| | | | | |

<!-- Type: BLOCKING | WARNING -->
<!-- BLOCKING findings force DECISION: FAIL -->
<!-- WARNING items are non-blocking but must be surfaced -->
<!-- Warnings use stable IDs (W-001, W-002) for later human disposition -->

---

## Decision

**DECISION:** _______________
<!-- PASS | PASS_WITH_WARNINGS | FAIL -->

### Decision Semantics

- **PASS**: Verification ran, all required checks pass, no blocking findings, no warnings requiring disposition. Archive MAY proceed.
- **PASS_WITH_WARNINGS**: Verification checks pass, no blocking issue, non-blocking warnings exist. Archive MAY proceed only after human explicitly accepts or defers warnings. PASS_WITH_WARNINGS must NOT silently mean PASS.
- **FAIL**: Verification ran, blocking audit issues found. Archive SHALL NOT proceed. Human override, if used at archive stage, must be explicit with reason.

<!-- A prerequisite BLOCK is not the same as DECISION: FAIL. -->
<!-- FAIL means verification actually ran and found blocking audit issues. -->

### Decision Metadata

**Reviewed Revision:** _______________
**Decision Summary:** _______________
**Blocking Finding Count:** ___
**Warning Count:** ___
**Verification Evidence Paths:** _______________
