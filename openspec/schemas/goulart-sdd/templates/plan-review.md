# Plan Review

## Review Metadata

<!-- Required: Record review context and mode -->

**Review Mode:** _______________
<!-- Independent | Degraded -->
<!-- Independent = fresh context (harness-supplied or user-managed clean session) -->
<!-- Degraded = no fresh context available; limitation disclosed below -->

**Reviewer Context/Session:** _______________
<!-- Describe the context used: "harness-supplied fresh context", "user-managed clean session", or "degraded — same session as author" -->

**Degraded Review Disclosure:**
<!-- If Review Mode is Degraded, the limitation MUST be disclosed here. -->
<!-- The human MUST acknowledge degraded mode before any disposition that permits downstream progression (ACCEPTED or permitted OVERRIDDEN). -->
<!-- A degraded review MUST NOT be described as independent. -->

- [ ] Degraded review limitation disclosed (required if Degraded mode)
- [ ] Human acknowledgement of degraded-review limitation recorded (required if Degraded mode)

---

## Review Round

**ROUND:** 1 | 2

<!-- Round 1: First review of this artifact -->
<!-- Round 2: Follows Round 1 due to REVISE, material APPROVE_WITH_CHANGES corrections, or material change to reviewed inputs -->
<!-- Maximum 2 rounds per review stage. Round 2 with unresolved issues → STOP/escalate. -->

**Previous Round Outcome:**
<!-- Required for Round 2. Concise summary of previous-round disposition/history. -->
<!-- Leave empty for Round 1. -->

```
<!-- Paste previous round verdict, findings summary, and human disposition here -->
```

---

## Reviewed Inputs

<!-- Required: Record exactly what was reviewed and at which revision -->

**Review Timestamp:** _______________
<!-- Timestamp is informational only; review freshness is determined from reviewed revisions/paths, not time. -->

**Proposal:**

- Path: `openspec/changes/<change>/proposal.md`
- Revision/Commit: _______________

**Specs:**

| Spec Path | Revision/Commit |
|---|---|
| `openspec/changes/<change>/specs/...` | |
| | |

**Design:**

- Path: `openspec/changes/<change>/design.md`
- Revision/Commit: _______________

**Additional Reviewed Paths:**
<!-- Record any other files or revisions reviewed for context -->

| Path | Revision/Commit | Reason |
|---|---|---|
| | | |

---

## Findings

<!-- Record all findings. Each finding MUST include severity and category. -->

### Finding: F-001

**Severity:** Critical | Moderate | Suggestion
**Category:** compliance | quality | feasibility | scope | risk

**Description:**
<!-- What was found -->

**Evidence/Context:**
<!-- Where or how the finding was observed -->

---

### Finding: F-002

**Severity:** Critical | Moderate | Suggestion
**Category:** compliance | quality | feasibility | scope | risk

**Description:**

**Evidence/Context:**

---

<!-- Add additional findings as needed. Use F-003, F-004, etc. -->

**Finding Summary:**

| ID | Severity | Category | Status |
|---|---|---|---|
| F-001 | | | Open |
| F-002 | | | Open |

<!-- Status: Open | Addressed | Superseded -->

---

## Verdict

<!-- Exactly one verdict must be selected -->

**APPROVE** | **APPROVE_WITH_CHANGES** | **REVISE**

<!-- APPROVE: The plan is acceptable as-is. Human may record ACCEPTED. -->
<!-- APPROVE_WITH_CHANGES: Required changes must be applied. Materiality determines re-review requirement. -->
<!-- REVISE: Normal progression blocked. Artifacts must be revised and re-reviewed. -->

---

## Required Changes

<!-- Only applicable for APPROVE_WITH_CHANGES or REVISE -->

### Change: RC-001

**Requested Change:**
<!-- What must be changed -->

**Applied:** Yes | No

**Materiality:** MATERIAL | NON_MATERIAL

**Materiality Rationale:**
<!-- Why this change is classified as material or non-material -->
<!-- Material: affects requirements, scope, architecture, acceptance criteria, or implementation assumptions -->
<!-- Non-material: editorial correction only, does not alter those semantics -->

---

### Change: RC-002

**Requested Change:**

**Applied:** Yes | No

**Materiality:** MATERIAL | NON_MATERIAL

**Materiality Rationale:**

---

<!-- Add additional Required Changes as needed. Use RC-003, RC-004, etc. -->

**Required Changes Summary:**

| ID | Applied | Materiality |
|---|---|---|
| RC-001 | No | |
| RC-002 | No | |

<!-- ACCEPTED MUST NOT be used to bypass unapplied Required Changes -->

---

## Human Decision

<!-- Must remain unrecorded until the human actually chooses -->

**STATUS:** ACCEPTED | REVISE | OVERRIDDEN

**Human Reason:**
<!-- Human Reason is REQUIRED for REVISE -->
<!-- Human Reason is REQUIRED for OVERRIDDEN -->
<!-- Human Reason is optional for APPROVE + ACCEPTED -->
<!-- For APPROVE_WITH_CHANGES + ACCEPTED, include the required NON_MATERIAL justification -->

```
<!-- Human records decision reason here -->
```

---

## Re-review / Override Evidence

<!-- Required when material changes require another round but human overrides, -->
<!-- or when re-review is waived. Record affected inputs and waived requirement. -->

**Affected Inputs (if applicable):**
<!-- Which reviewed inputs were modified by applied changes -->

| Input Path | Change Description |
|---|---|
| | |

**Re-review Waived:** Yes | No

**Waiver Reason:**
<!-- Mandatory if re-review is waived for material changes -->
<!-- Must not falsely claim the stale review covers modified inputs -->

---

## Escalation Evidence

<!-- Required when Round 2 leaves REVISE, unresolved Required Changes, -->
<!-- or material changes requiring further review. The workflow MUST STOP here. -->

**Escalation Required:** Yes | No

**Escalation Reason:**
<!-- Why escalation is needed: unresolved findings, unapplied material changes, or round limit reached -->

**Unresolved Issues Summary:**
<!-- Concise summary for human escalation -->
<!-- The workflow MUST NOT start a third round automatically -->
