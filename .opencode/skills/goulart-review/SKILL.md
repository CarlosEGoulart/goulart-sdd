---
name: goulart-review
description: Use when performing independent plan review or code review in the Goulart SDD workflow, with stage-specific artifact contracts, fresh-context independence, degraded fallback, findings, verdicts, materiality/staleness, explicit overrides, and bounded round persistence.
compatibility: Requires OpenSpec CLI and a Goulart-compatible schema.
metadata:
  author: goulart-sdd
  version: "1.0"
---

# Goulart Review

Two review stages under one reviewer entry point:

```text
goulart-review plan  → plan-review artifact
goulart-review code  → code-review artifact
```

## 1. Authorization and ownership

This is the REVIEWER context. The reviewer evaluates and records findings.
Write only the selected change's permitted review artifact.

FORBIDDEN actions (any stage):

- modify proposal.md, specs/, design.md
- modify implementation/source files or tests
- execute implementation fixes or planning fixes
- invoke goulart-plan, goulart-apply, goulart-verify, archive
- change schemas, templates, project configuration
- modify upstream `opsx-*` commands / `openspec-*` skills

Stage-specific permitted write:

| Stage | Only permitted write |
|---|---|
| plan | plan-review.md |
| code | code-review.md |

Do not silently fix what you review. Findings belong in the review artifact.

## 2. Stage selection

Exactly two stages: `plan` and `code`.

**If stage is explicitly provided:** use it.

**If stage is missing or ambiguous:** ASK whether the user wants plan review or code review. Do not guess.

Do NOT infer `code` merely because implementation files exist.
Do NOT infer `plan` merely because plan-review is missing.

## 3. Change / planning-home resolution

1. Run `openspec context --json`. If the user names a registered store or the work is in one, discover its id using `openspec store list --json` and use `openspec context --json --store "<id>"`. Once selected, keep `--store "<id>"` on every subsequent `context`, `list`, `schemas`, `new change`, `status`, `instructions`, `show`, or `validate` command.
2. Use the returned `root.path`. On `no_openspec_root`, STOP and report initialization is needed. On other context errors, STOP with the actual error.
3. Read `<root.path>/openspec/config.yaml`. Apply a valid string `context` field up to 51,200 UTF-8 bytes as project constraints. If absent/invalid, report the limitation.
4. Resolve the change: use an explicitly named existing change, or an unambiguous existing change identified in conversation. Otherwise run `openspec list --json`. With multiple candidates, ASK. Do not choose by timestamp.
5. Run `openspec status --change "<name>" --json` without a schema override. Use `planningHome`, `changeRoot`, `artifactPaths`, and `actionContext` as authoritative paths/scope.
6. Confirm schema compatibility: it must support proposal, specs, design, plan-review, test-plan, tasks, code-review and verify. If incompatible, STOP and report.

## 4. Independence hierarchy

Use this hierarchy in order:

1. **Harness-supplied separate/fresh context** — preferred.
2. **User-managed separate clean session** — fallback, still independent.
3. **Degraded review** — last resort only.

Rules:

- Independent means a context/session separate from the author/implementer context.
- Cross-model review is optional and NOT required for independence.
- Same model + truly fresh separate session MAY be independent.
- Different model + author/implementer conversation history is NOT independent.
- Never use model identity as proof of independence.

## 5. Independence detection

Before marking Review Mode = Independent, confirm one of:

- harness explicitly supplied isolated context;
- user explicitly opened a clean review session;
- environment/session semantics clearly establish fresh context.

If independence cannot be established:

- Do NOT silently label the review Independent.
- If a user-managed clean session is available, STOP and instruct the user to run the review there.
- Only use Degraded mode when BOTH independent alternatives are unavailable.
- If uncertain whether they are unavailable, ASK.

## 6. Degraded review

Degraded mode is a LAST RESORT.

When legitimately used:

- Set Review Mode = Degraded.
- Disclose why independent context could not be obtained.
- Explicitly state the limitation.
- Do NOT describe the result as independent.

Human acknowledgement is HUMAN-owned. Do NOT fabricate acknowledgement.

The review itself MAY be produced in degraded mode after the fallback condition
is established. However, progression beyond the review remains blocked until
actual human acknowledgement is recorded where required. Report that requirement
explicitly.

## 7. Shared finding taxonomy

Each real finding must contain:

| Field | Values |
|---|---|
| ID | F-001, F-002, ... (stable) |
| Severity | Critical \| Moderate \| Suggestion (exactly one) |
| Category | compliance \| quality \| feasibility \| scope \| risk (exactly one) |
| Description | What was found |
| Evidence/Context | Where or how the finding was observed |

Do NOT fabricate findings to populate template examples.

If there are zero findings, record a clean review.

## 8. Shared round persistence

Plan review and code review have independent round counters.

Allowed: ROUND 1, ROUND 2.
Never: ROUND 3.
Never silently reset history to Round 1.

| Round | When |
|---|---|
| ROUND: 1 | First actual review of this artifact |
| ROUND: 2 | Follows Round 1 due to REVISE, material APPROVE_WITH_CHANGES corrections, or another material change to reviewed state |

Round 2 must preserve a concise Previous Round Outcome in the same artifact.
Include: prior verdict, material findings/required changes, human disposition/triage where actually recorded, reason a second round occurred.

Do not erase review history.

## 9. Shared round-2 escalation

If Round 2 leaves:

PLAN: REVISE, unresolved Required Changes, or material change requiring another review.

CODE: unresolved Blocking Conditions, unresolved required changes, or material implementation change requiring another review.

Then:

STOP and ESCALATE TO HUMAN.

Do NOT start Round 3.
Do NOT rewrite Round 2 as Round 1.
Do NOT discard history.

An existing exact human override may resolve only the condition it actually waives. Do not broaden it.

## 10. Shared staleness rules

Record exactly which state was reviewed.

Prefer repository revision/diff evidence.

For clean committed files, record relevant revision/commit.

For reviewed working-tree content not represented by HEAD, do NOT falsely say HEAD contains it. Record the repository revision plus appropriate working-tree/content evidence.

Never use filesystem timestamps as proof of freshness.

When a previous review of the same stage exists, compare its recorded reviewed state against current state. Material change after previous review: STALE, new review round normally required. Non-material preservation is allowed only where the integrated contract permits it and rationale is recorded. Unknown freshness: do NOT call it FRESH, report the gap.

## 11. Shared override rules

Override is HUMAN-owned.

It must record:

- Override Invoked: Yes
- exact Waived Condition
- mandatory Override Reason
- Affected Review State / Inputs

An override:

- does NOT make stale review fresh
- does NOT mean changed inputs were reviewed
- does NOT replace task completion (code stage)
- does NOT replace mandatory finding triage (code stage)
- does NOT arise implicitly from REJECT/DEFER (code stage)

If override evidence is incomplete, report it as incomplete. Do NOT fabricate missing values.

---

# PLAN REVIEW

## 12. Plan-review prerequisites

For `goulart-review plan`, inspect structural AND semantic state.

Required reviewed inputs: proposal, all applicable change specs, design.

Steps:

1. Run `openspec status --change "<name>" --json` for structural evidence.
2. Run `openspec instructions plan-review --change "<name>" --json` for the current artifact contract.
3. Read the actual proposal/spec/design contents.
4. OpenSpec status is NOT enough to prove substantive completeness.

If required initial planning artifacts are missing or materially incomplete:

STOP. Report exactly what is missing. Next action should point back to the author/planning context. Do NOT create a fake review for an incomplete plan.

## 13. Plan review is adversarial

Evaluate at minimum:

**COMPLIANCE**: proposal/spec/design consistency; all approved requirements/scenarios represented; design supports specs; no design behavior contradicts specs; no required behavior omitted.

**QUALITY**: ambiguity; incomplete reasoning; assumptions presented as fact; untestable/unverifiable requirements; missing decisions that materially affect implementation.

**FEASIBILITY**: technically implausible assumptions; unaccounted dependencies/integrations; approach unable to satisfy specs; unresolved build-changing questions.

**SCOPE**: scope creep; missing in-scope behavior; unnecessary architecture; mismatch between proposal and design.

**RISK**: architecture; migration/compatibility; reliability; security; assumptions likely to cause implementation or verification failure.

Do not invent new product requirements because you prefer a different design. Review against the approved contract.

## 14. Plan verdicts

Emit EXACTLY ONE: APPROVE | APPROVE_WITH_CHANGES | REVISE.

### APPROVE

Use when plan is acceptable as-is. No Required Change should be invented.

The reviewer does NOT record human acceptance.

After review, human must record STATUS: ACCEPTED (reason optional).

The review skill must STOP after producing the review. Do NOT continue into test-plan/tasks.

### APPROVE_WITH_CHANGES

Use when concrete Required Changes are necessary before progression.

Each Required Change must include:

| Field | Values |
|---|---|
| ID | RC-* |
| Requested Change | What must be changed |
| Applied | Yes \| No |
| Materiality | MATERIAL \| NON_MATERIAL |
| Materiality Rationale | Why classified as material or non-material |

At initial review creation, do NOT pretend reviewer-requested changes were already applied. Normally they begin Applied: No, unless evaluating an already-recorded state where actual repository evidence proves otherwise.

ACCEPTED must never bypass unapplied Required Changes.

### REVISE

Use when normal progression must stop and planning artifacts require revision.

Reviewer does NOT modify those artifacts.

Human override is possible only through a real STATUS: OVERRIDDEN with mandatory reason. Do NOT fabricate one.

## 15. Plan human disposition is not reviewer output

The PLAN reviewer produces:

- review evidence
- findings
- verdict
- Required Changes where applicable

The reviewer does NOT choose the human disposition.

Do NOT automatically populate STATUS: ACCEPTED, STATUS: REVISE, or STATUS: OVERRIDDEN as if the reviewer were the human.

Leave current-round human disposition pending unless it was explicitly and actually supplied by the human through an authorized recording workflow.

Never infer a human disposition from "looks good", a reviewer APPROVE, absence of complaints, or a model decision.

After producing the review: STOP. Report the exact human action now required.

## 16. Plan materiality

**MATERIAL** means a correction affects requirements, scope, architecture, acceptance criteria, or implementation assumptions.

Applied MATERIAL corrections make the previous review stale. Normal progression requires another independent plan-review round.

Exception: human may explicitly record STATUS: OVERRIDDEN with mandatory reason. Such override must identify: exact waived re-review condition, affected reviewed input path(s), concise changed-input description, Re-review Waived: Yes, mandatory waiver/override reason.

Override != fresh. Override != reviewed changed plan.

**NON_MATERIAL** means truly editorial/non-semantic correction.

If all Required Changes are applied and demonstrably NON_MATERIAL, then human MAY record ACCEPTED without another review. Materiality rationale is mandatory. Do not infer missing rationale.

## 17. Plan verdict/disposition matrix

Every permitting row still requires all other gates to pass.

| Actual review/human state | Outcome |
|---|---|
| Missing/pending/ambiguous verdict or human STATUS | STOP; identify the missing/ambiguous field. |
| Human STATUS: REVISE, with any verdict | STOP; human requests revision. |
| APPROVE + ACCEPTED + current review | Permit if all other gates pass. |
| APPROVE_WITH_CHANGES + any unapplied RC | STOP; list every unapplied RC. |
| APPROVE_WITH_CHANGES + all RCs applied, NON_MATERIAL, recorded rationale + ACCEPTED | May permit without re-review. |
| APPROVE_WITH_CHANGES + applied MATERIAL RC, no current review or exact override | Prior review STALE; STOP for re-review or escalation. |
| Applied MATERIAL correction + later current review | Evaluate that current round normally. |
| Applied MATERIAL correction + exact permitted OVERRIDDEN | May permit only under override rules; prior review remains STALE. |
| REVISE without exact permitted OVERRIDDEN | STOP; normally revise and obtain re-review. |
| REVISE + exact permitted OVERRIDDEN with mandatory reason | May permit; disclose the exception. |

---

# CODE REVIEW

## 18. Code-review prerequisite

For `goulart-review code`, first verify implementation stage readiness.

Required: tasks.md exists AND all actual implementation task checkboxes are complete.

Do NOT interpret "tasks artifact exists" as "implementation is complete".

Read tasks.md.

If any implementation task remains pending: STOP. Report exact pending task IDs. Next action: return to implementation/goulart-apply. Do NOT perform partial final code review and call it complete.

## 19. Code-review inputs

Use `openspec instructions code-review --change "<name>" --json`.

Read: specs, design, tasks, test-plan, implementation diff/state, tests, configuration, other affected source paths as needed.

Identify: Reviewed Revision, Base Revision where appropriate, Reviewed Paths, Test Paths, Additional Evidence Paths.

If the implementation contains uncommitted changes, do not pretend HEAD alone identifies the reviewed implementation. Record the actual reviewed state honestly using revision plus working-tree/diff evidence.

Never use timestamps as freshness evidence.

## 20. Required code-review coverage

Evaluate:

- implementation vs specs
- implementation vs design
- scope compliance
- test quality
- behavioral coverage
- code quality
- maintainability
- architecture
- unnecessary complexity
- potential regressions

Spec scenarios are the behavioral contract. Compare actual behavior/implementation to those scenarios. Flag scope creep. Do not reduce code review to lint/style review.

## 21. Code finding contract

Each finding must have:

| Field | Values |
|---|---|
| ID | F-* (stable) |
| Severity | Critical \| Moderate \| Suggestion |
| Category | compliance \| quality \| feasibility \| scope \| risk |
| Description | What was found |
| Evidence/Context | Where or how observed |
| Affected Path/Component | Which file/module/component |
| Requires Implementation Change | Yes \| No |
| Blocking Condition | Yes \| No |

**IMPORTANT**: Requires Implementation Change and Blocking Condition are INDEPENDENT.

Do NOT infer: Requires Change: Yes therefore Blocking: Yes. A non-blocking correction may still require a change.

Blocking Condition: Yes means normal progression is blocked until resolved or an exact permitted human override waives that condition.

Any Critical finding MUST force VERDICT: REVISE. An unresolved Critical condition remains blocking regardless of triage.

## 22. Code-review verdicts

Emit EXACTLY ONE: APPROVE | APPROVE_WITH_CHANGES | REVISE.

### Clean review (findings = 0)

VERDICT: APPROVE.

No human finding triage is required. No general/unconditional human approval is required. Do NOT manufacture a human approval gate.

If degraded mode was used, its human acknowledgement requirement remains separate and still applies.

### APPROVE with non-blocking findings

APPROVE may contain findings when no blocking implementation change is required. Every actual finding still requires human triage. Do not treat APPROVE as "findings can be ignored".

### APPROVE_WITH_CHANGES

Use when at least one implementation change is required before verification, without a Critical condition forcing REVISE.

Human triage is required. Accepted/required implementation changes must be addressed before verification.

### REVISE

Normal progression to verify is blocked. Implementation must normally be revised and reviewed again.

Any Critical finding forces REVISE.

Explicit human override may exist only with: Override Invoked: Yes, exact Waived Condition, mandatory Override Reason, affected review state/inputs.

Reviewer must NOT fabricate override.

## 23. Code-review human triage

If findings exist, the HUMAN must triage EACH finding as exactly one: ACCEPT | REJECT | DEFER.

Reviewer must NOT fabricate dispositions.

| Disposition | Justification | Effect |
|---|---|---|
| ACCEPT | Optional | If change required, must be addressed before verify. Does NOT resolve blocking condition alone. |
| REJECT | REQUIRED | Does NOT automatically waive blocking or Critical conditions. |
| DEFER | REQUIRED | Does NOT automatically waive blocking or Critical conditions. |

REJECT or DEFER without justification is incomplete triage.

All mandatory triage must be complete before normal progression.

If an unresolved blocking/Critical condition remains, normal progression remains blocked.

REJECT/DEFER is NOT itself an override. Record the actual disposition and follow-up without silently treating a rejected/deferred change as applied.

## 24. Code-review Required Changes

For APPROVE_WITH_CHANGES or REVISE, use the template's RC-* records where implementation changes are required.

Link them to relevant finding IDs.

Record: requested implementation change, Applied state, affected paths/revision where available, Materiality, Materiality Rationale.

Do not claim a fix is applied unless repository evidence proves it. Reviewer never applies the fix.

## 25. Code-review materiality

**MATERIAL** implementation change: prior code-review is STALE. Normal progression requires a new code-review round. This includes material fixes made in response to accepted findings.

**NON_MATERIAL** correction: may avoid re-review only when demonstrably non-material AND a clear rationale is recorded.

Do not use timestamps. Use revision/diff evidence where possible. Use conservative semantic comparison when a clean revision cannot represent the reviewed state.

Do not call stale review fresh because a human override exists.

## 26. Code-review verdict/triage matrix

| Actual state | Outcome |
|---|---|
| Findings = 0 | APPROVE. No triage or general approval required. |
| Findings exist, all non-blocking, triage complete | APPROVE may be valid. |
| At least one implementation change required, no Critical | APPROVE_WITH_CHANGES may be appropriate. Triage required. |
| Any Critical finding | REVISE. Triage does not resolve Critical. |
| Unresolved Blocking Condition | Normal progression blocked. |
| REVISE + exact permitted OVERRIDDEN | May proceed under override rules. |

---

# BOTH STAGES

## 27. Idempotent repeated invocation

If a review artifact already exists and accurately covers the current reviewed state:

do NOT create an artificial new review round simply because goulart-review was invoked again.

Report the current review state. If human action remains pending, report the exact pending action.

If a substantive/material state change requires re-review, advance according to Round 1 -> Round 2 rules.

If Round 2 already exists and another review would be required: STOP/escalate.

## 28. Review artifact ownership

The skill writes only the appropriate artifact:

| Stage | Artifact |
|---|---|
| plan | plan-review.md |
| code | code-review.md |

Use OpenSpec `openspec instructions "<artifact>" --change "<name>" --json` for current execution guidance. Re-read dependencies from disk.

Do not create both review artifacts in one invocation.

## 29. Human-owned fields

Never fabricate:

- plan STATUS (ACCEPTED/REVISE/OVERRIDDEN)
- Human Reason
- degraded human acknowledgement
- code finding triage (ACCEPT/REJECT/DEFER)
- REJECT/DEFER justification
- override invocation
- override reason
- waived condition
- human escalation resolution

Template placeholders must not be mistaken for recorded human decisions. If human action has not occurred, leave it pending/unrecorded.

The final response must clearly tell the human what must be recorded next.

## 30. Raw OpenSpec boundary

Raw OpenSpec remains an escape hatch.

If evidence indicates artifacts/reviews were produced outside Goulart gates, do not retroactively describe them as Goulart-compliant merely because the files exist.

Evaluate their actual evidence. Report missing guarantees precisely. Do not delete them automatically. Do not modify upstream OpenSpec behavior.

## 31. Required output after PLAN review

Report:

- change name, schema, planning home/store
- review stage = plan
- review mode = Independent | Degraded
- review context basis
- ROUND
- reviewed input paths/state
- finding count and findings summary
- verdict
- Required Changes
- human disposition state
- degraded acknowledgement state if applicable
- staleness/re-review requirement
- escalation state
- next required human/author action

Examples:

APPROVE: human records STATUS: ACCEPTED; later goulart-plan.

APPROVE_WITH_CHANGES: author applies Required Changes; material changes normally require another review; non-material path requires actual rationale + human acceptance.

REVISE: author revision/re-review or exact human override; never silently continue.

Round 2 unresolved: human escalation.

Always STOP after the review.

## 32. Required output after CODE review

Report:

- change name, schema, planning home/store
- review stage = code
- review mode
- review context basis
- ROUND
- Reviewed Revision / Base Revision
- Reviewed Paths
- finding count and findings summary
- verdict
- triage state
- Required Changes
- blocking conditions
- staleness/re-review requirement
- override state
- escalation state
- next required human/implementation action

Clean APPROVE: no finding triage or general approval required; if independent and otherwise valid, goulart-verify is the next stage.

APPROVE with findings: human triage required.

APPROVE_WITH_CHANGES: human triage + accepted/required implementation changes.

REVISE: implementation revision + re-review or exact permitted override.

Round 2 unresolved: human escalation.

Do NOT invoke verify automatically. Always STOP.

## 33. Keep skill usable by local/cheap models

This skill will often be executed by smaller/local models. Make behavior explicit.

Prefer: ordered state checks, small decision tables, exact permitted values, explicit STOP conditions, clear stage split, concrete required evidence.

Avoid: vague "use judgment" where a gate can be explicit, duplicated prose, model/vendor-specific assumptions, hidden state, reliance on another LLM vendor.

Do NOT make any specific model a methodology requirement.
