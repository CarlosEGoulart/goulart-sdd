# Code Review

## Review Metadata

<!-- Required: Record review context and mode -->

**Review Mode:** _______________
<!-- Independent | Degraded -->
<!-- Independent = fresh context (harness-supplied or user-managed clean session) -->
<!-- Degraded = no fresh context available; limitation disclosed below -->
<!-- Cross-model review is optional and is NOT required for independence -->

**Reviewer Context/Session:** _______________
<!-- Describe the context used: "harness-supplied fresh context", "user-managed clean session", or "degraded — same session as implementer" -->

**Degraded Review Disclosure:**
<!-- If Review Mode is Degraded, the limitation MUST be disclosed here. -->
<!-- The human MUST acknowledge degraded mode before any disposition that permits downstream progression. -->
<!-- A degraded review MUST NOT be described as independent. -->

- [ ] Degraded review limitation disclosed (required if Degraded mode)
- [ ] Human acknowledgement of degraded-review limitation recorded (required if Degraded mode)

---

## Review Round

**ROUND:** 1 | 2

<!-- Round 1: First review of this implementation -->
<!-- Round 2: Follows Round 1 due to REVISE, material APPROVE_WITH_CHANGES corrections, or material implementation change -->
<!-- Maximum 2 rounds per review stage. Round 2 with unresolved issues → STOP/escalate. -->

**Previous Round Outcome:**
<!-- Required for Round 2. Concise summary of previous-round disposition/history. -->
<!-- Leave empty for Round 1. -->

```
<!-- Paste previous round verdict, findings summary, and human triage disposition here -->
```

---

## Reviewed Implementation

<!-- Required: Record exactly what implementation was reviewed and at which revision -->

**Review Timestamp:** _______________
<!-- Timestamp is informational only; review freshness is determined from reviewed revisions/paths, not time. -->

**Reviewed Revision:** _______________
<!-- The commit/revision of the implementation being reviewed -->

**Base Revision:** _______________
<!-- Optional: The reference/base revision for comparison -->

**Reviewed Paths:**

| Path | Description |
|---|---|
| | |

**Test Paths:**

| Path | Description |
|---|---|
| | |

**Additional Evidence Paths:**

| Path | Reason |
|---|---|
| | |

---

## Review Coverage

<!-- Evidence that the review considered the required evaluation scope -->

| Area | Result | Evidence/Notes |
|---|---|---|
| Implementation vs specs | | |
| Implementation vs design | | |
| Scope compliance | | |
| Test quality | | |
| Behavioral coverage | | |
| Code quality | | |
| Maintainability | | |
| Architecture | | |
| Unnecessary complexity | | |
| Potential regressions | | |

<!-- Result: Pass | Concern | Not Applicable -->

---

## Findings

<!-- Record all findings. Each finding MUST include severity and category. -->
<!-- Any Critical finding forces VERDICT: REVISE -->

### Finding: F-001

**Severity:** Critical | Moderate | Suggestion
**Category:** compliance | quality | feasibility | scope | risk

**Description:**
<!-- What was found -->

**Evidence/Context:**
<!-- Where or how the finding was observed -->

**Affected Path/Component:**
<!-- Which file, module, or component is affected -->

**Requires Implementation Change:** Yes | No

**Blocking Condition:** Yes | No
<!-- Yes = normal progression remains blocked until the condition is resolved -->
<!-- or an explicitly permitted override records the waived condition and reason -->

---

### Finding: F-002

**Severity:** Critical | Moderate | Suggestion
**Category:** compliance | quality | feasibility | scope | risk

**Description:**

**Evidence/Context:**

**Affected Path/Component:**

**Requires Implementation Change:** Yes | No

**Blocking Condition:** Yes | No

---

<!-- Add additional findings as needed. Use F-003, F-004, etc. -->

**Finding Summary:**

| ID | Severity | Category | Requires Change | Blocking | Triage |
|---|---|---|---|---|---|
| F-001 | | | | | Pending |
| F-002 | | | | | Pending |

<!-- Triage: Pending | ACCEPT | REJECT | DEFER -->

---

## Per-Finding Human Triage

<!-- Each finding MUST be triaged if findings exist. -->
<!-- Clean review with no findings requires NO triage and NO unconditional approval. -->

### Triage: F-001

**Disposition:** ACCEPT | REJECT | DEFER

**Justification:**
<!-- Required for REJECT and DEFER -->
<!-- Optional for ACCEPT -->

**Follow-up / Required Action:**
<!-- What must be done if accepted; why rejected/deferred -->

---

### Triage: F-002

**Disposition:** ACCEPT | REJECT | DEFER

**Justification:**

**Follow-up / Required Action:**

---

<!-- Add additional triage entries as needed -->

**Triage Summary:**

| Finding | Disposition | Justification Recorded | Action Required |
|---|---|---|---|
| F-001 | | | |
| F-002 | | | |

<!-- Rules: -->
<!-- ACCEPT: finding accepted; if it requires a change, that change must be addressed before verify -->
<!-- ACCEPT does not by itself resolve a blocking condition -->
<!-- REJECT: justification REQUIRED; does not implicitly waive a blocking condition -->
<!-- DEFER: justification REQUIRED; does not implicitly waive a blocking condition -->
<!-- Unresolved blocking conditions prevent normal progression -->
<!-- Unresolved Critical findings are also blocking and force VERDICT: REVISE -->
<!-- An explicit override, when permitted, must use the Explicit Human Override section -->

---

## Verdict

<!-- Exactly one verdict must be selected -->

**APPROVE** | **APPROVE_WITH_CHANGES** | **REVISE**

<!-- APPROVE: No blocking implementation changes required. Non-blocking findings may exist and require triage. -->
<!-- Clean review with zero findings: verdict stands as-is, no triage or unconditional approval needed. -->
<!-- APPROVE_WITH_CHANGES: At least one implementation change required. All findings must be triaged. -->
<!-- REVISE: Normal progression to verify is blocked. Implementation must be revised and reviewed again. -->
<!-- Critical finding: verdict SHALL be REVISE (not APPROVE). -->

---

## Required Changes

<!-- Only applicable for APPROVE_WITH_CHANGES or REVISE -->
<!-- Links required implementation changes to findings -->

### Change: RC-001

**Related Finding(s):** F-001

**Requested Implementation Change:**
<!-- What must be changed in the implementation -->

**Applied:** Yes | No

**Affected Paths/Revision:**
<!-- Which files were changed and at which revision, if applicable -->

**Materiality:** MATERIAL | NON_MATERIAL

**Materiality Rationale:**
<!-- Why this change is classified as material or non-material -->
<!-- Material: makes the prior review stale; new code-review round required -->
<!-- Non-material: may avoid re-review only with recorded justification -->

---

### Change: RC-002

**Related Finding(s):**

**Requested Implementation Change:**

**Applied:** Yes | No

**Affected Paths/Revision:**

**Materiality:** MATERIAL | NON_MATERIAL

**Materiality Rationale:**

---

<!-- Add additional Required Changes as needed. Use RC-003, RC-004, etc. -->

**Required Changes Summary:**

| ID | Finding | Applied | Materiality |
|---|---|---|---|
| RC-001 | F-001 | No | |
| RC-002 | | No | |

---

## Review Staleness / Re-review Evidence

<!-- Required when implementation changes are made after review -->
<!-- The artifact must NOT claim a stale review covers changed implementation -->

**Implementation Changes After Review:** Yes | No

**Changed Paths:**

| Path | Change Description | Resulting Revision |
|---|---|---|
| | | |

**Staleness Classification:** MATERIAL | NON_MATERIAL

**Materiality Rationale:**
<!-- Material: prior review stale; new code-review round required -->
<!-- Non-material: re-review may be skipped only with recorded justification -->

**Re-review Required:** Yes | No

**Re-review Justification:**
<!-- Required if re-review is skipped for non-material changes -->

---

## Explicit Human Override

<!-- Used ONLY when human explicitly overrides a review condition -->
<!-- Do NOT create a general human-approval gate -->
<!-- Clean review with no findings requires NO override -->

**Override Invoked:** Yes | No

**Waived Condition:**
<!-- What review condition is being waived -->

**Override Reason:**
<!-- Mandatory when override is invoked -->

**Affected Review State / Inputs:**
<!-- What review state is affected; must not claim stale review is fresh -->

<!-- Rules: -->
<!-- Override reason is mandatory when invoked -->
<!-- Override must identify what review condition is being waived -->
<!-- Override must not claim stale review is fresh -->
<!-- Override does NOT replace: task completion, required finding triage, evaluable test-plan entries -->
<!-- Rejecting or deferring a finding is NOT automatically an override -->

---

## Escalation Evidence

<!-- Required when Round 2 leaves unresolved blocking conditions, -->
<!-- required changes, or material changes requiring further review. -->
<!-- The workflow MUST STOP here rather than start ROUND 3. -->

**Escalation Required:** Yes | No

**Escalation Reason:**
<!-- Why escalation is needed: unresolved blocking conditions, required changes, or round limit reached -->

**Unresolved Conditions:**
<!-- Concise summary for human escalation, e.g.: -->
<!-- - F-003 — Blocking Condition: Yes -->
<!-- - F-004 — Critical -->
<!-- The workflow MUST NOT start a third round automatically -->
<!-- The limit includes re-reviews after APPROVE_WITH_CHANGES or other material implementation changes -->
