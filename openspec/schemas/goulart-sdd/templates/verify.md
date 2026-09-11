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
**Reviewed Artifacts:** proposal, specs, design

**Material Changes After Review:** Yes | No
**Evidence / Diff Reference:** _______________

**Freshness Result:** FRESH | STALE | OVERRIDDEN
<!-- FRESH = no material changes detected -->
<!-- STALE = material changes detected, re-review required -->
<!-- OVERRIDDEN = material changes exist, explicitly waived with reason -->

**Override Reason:** _______________
<!-- Required when Freshness Result = OVERRIDDEN -->
<!-- Override records an exception, NOT proof of freshness -->

<!-- Material changes to proposal/specs/design after plan-review make it stale. -->
<!-- Reviewer-requested material changes still make the prior review stale. -->
<!-- Non-material corrections may avoid re-review with recorded rationale. -->

### Code Review Staleness

**Reviewed Revision:** _______________
**Reviewed Artifacts:** source, tests, configuration

**Material Changes After Review:** Yes | No
**Evidence / Diff Reference:** _______________

**Freshness Result:** FRESH | STALE | OVERRIDDEN

**Override Reason:** _______________

**Accepted Finding Caused Material Change:** Yes | No
<!-- If an accepted finding caused material implementation changes, -->
<!-- the prior code review is stale regardless of acceptance. -->

**Round Limit State:** _______________
<!-- Record bounded-round escalation state if relevant -->

<!-- Source/test/config changes after code review make it stale. -->
<!-- Material fixes for accepted findings still make the prior review stale. -->
<!-- Non-material corrections may avoid re-review with clear justification. -->

---

## Review Overrides

| Review | Condition Waived | Override Recorded | Reason | Freshness Still Stale? |
|---|---|---|---|---|
| | | | | |

<!-- Mandatory reason when override is invoked. -->
<!-- Override may waive the identified condition only when permitted. -->
<!-- Override does NOT make a stale review fresh. -->
<!-- Override does NOT imply independent review occurred. -->
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
