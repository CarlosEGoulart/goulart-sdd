---
name: goulart-verify
description: Use when verifying Goulart SDD implementation completeness, spec compliance, test integrity, review freshness, and producing the verify decision.
compatibility: Requires OpenSpec CLI and a Goulart-compatible schema.
metadata:
  author: goulart-sdd
  version: "1.0"
---

# Goulart Verify

One verification per invocation:

```text
goulart-verify
  → verify prerequisites (code-review, triage, tasks, test-plan, freshness)
  → force degraded status for reviewer and implementer
  → perform verification
  → produce verify artifact
  → emit DECISION
  → STOP
```

## 1. Authorization and ownership

This is the Verifier context. You read implementation state, review artifacts, test results, and specs to produce the verify artifact.

FORBIDDEN actions:

- modify proposal.md, specs/, design.md, test-plan.md, tasks.md
- modify plan-review.md, code-review.md
- modify implementation/source files or tests
- execute implementation fixes or planning fixes
- invoke goulart-plan, goulart-apply, goulart-review, goulart-archive
- change schemas, templates, project configuration
- modify upstream `opsx-*` commands / `openspec-*` skills

Write only the verify artifact at the path provided by `openspec instructions verify`.

## 2. Resolve scope and select the change

1. Run `openspec context --json`. If the user names a registered store or the work is in one, discover its id using `openspec store list --json` and use `openspec context --json --store "<id>"`. Once selected, keep `--store "<id>"` on every subsequent `context`, `list`, `schemas`, `new change`, `status`, `instructions`, `show`, or `validate` command.
2. Use the returned `root.path`. On `no_openspec_root`, STOP and report initialization is needed. On other context errors, STOP with the actual error.
3. Read `<root.path>/openspec/config.yaml`. Apply a valid string `context` field up to 51,200 UTF-8 bytes as project constraints. If absent/invalid, report the limitation.
4. Resolve the change: use an explicitly named existing change, or an unambiguous existing change identified in conversation. Otherwise run `openspec list --json`. With multiple candidates, ASK. Do not choose by timestamp.
5. Run `openspec status --change "<name>" --json` without a schema override. Use `planningHome`, `changeRoot`, `artifactPaths`, and `actionContext` as authoritative paths/scope.
6. Confirm schema compatibility: it must support proposal, specs, design, plan-review, test-plan, tasks, code-review and verify. If incompatible, STOP and report.

## 3. Force degraded status

This is an involuntary, unconditional requirement.

**Before** any evaluation, both the reviewer and the implementer contexts are declared Degraded for this verification. No reasoning, claim, model identity, harness capability, session hygiene, or user assertion may lift this status.

Record in the verify artifact:

- Reviewer Context Status: Degraded
- Implementer Context Status: Degraded

Rationale: the verifier cannot independently prove the isolation or provenance of the upstream review and implementation contexts it consumes.

The review artifacts may still contain evidence, findings, and disposition records, but they shall not be treated as independent for this verification.

## 4. Pre-verification prerequisite checks

Execute these checks in order. If any gate fails, STOP and report the exact unresolved prerequisite. Do NOT proceed to verification.

### 4.1 Code-review existence

Check that `<changeRoot>/code-review.md` exists and is not empty.

If missing: STOP. Report: code-review.md missing. Next action: run goulart-review code.

### 4.2 Code-review verdict

Read code-review.md. Confirm exactly one verdict is recorded: APPROVE, APPROVE_WITH_CHANGES, or REVISE.

- APPROVE: permitted (may proceed if triage is complete)
- APPROVE_WITH_CHANGES: permitted only if all Required Changes are Applied: Yes
- REVISE: STOP. Report: code-review requires revision. Next action: run goulart-review code again after implementation revision.

If verdict is missing or ambiguous: STOP. Report exact missing field.

### 4.3 Human triage completeness

If code-review.md contains findings (any entry with Severity: Critical, Moderate, or Suggestion):

Check that every finding has a triage disposition: ACCEPT, REJECT, or DEFER.

If any finding lacks a triage disposition: STOP. Report exact untriaged finding IDs. Next action: human must triage each finding.

REJECT/DEFER findings without justification is incomplete triage. Report the gap.

### 4.4 Task completion

Read tasks.md. Confirm every implementation task checkbox is checked (`- [x]`).

If any task is unchecked: STOP. Report exact pending task IDs. Next action: run goulart-apply for each pending task.

### 4.5 Test-plan completeness

Read test-plan.md. For each required entry:

- AUTOMATED: must have a completed check result (pass/fail recorded)
- MECHANICAL: must have a completed check result (pass/fail recorded)
- SEMANTIC: must have a documented evaluation

If any required entry is pending or lacks a completed result: STOP. Report exact unevaluable entry IDs. Next action: complete the test-plan evaluation.

### 4.6 Review freshness

#### Plan-review staleness

Read plan-review.md. If it records a reviewed revision, compare against current state of proposal.md, specs/, and design.md.

Check whether any of these files were materially modified after the plan-review verdict:

- If Git revision is available: compare file content at recorded revision against current HEAD
- Otherwise: fall back to conservative semantic comparison

If material change is detected: check for an explicit override in plan-review.md (STATUS: OVERRIDDEN with mandatory reason covering this specific condition).

If material change without permitted override: STOP. Report: plan-review is stale. Next action: run goulart-review plan.

If non-material change: verify that plan-review.md records the rationale for non-material classification.

#### Code-review staleness

Read code-review.md. If it records a reviewed revision, compare against current state of source files, tests, and implementation-relevant configuration.

Check whether any of these files were materially modified after the code-review verdict:

- If Git revision is available: compare file content at recorded revision against current HEAD
- Otherwise: fall back to conservative semantic comparison

If material change is detected: check for an explicit override in code-review.md.

If material change without permitted override: STOP. Report: code-review is stale. Next action: run goulart-review code.

If non-material change: verify that code-review.md records the rationale for non-material classification.

#### Accepted finding causes material implementation change

If implementation was changed to address an accepted code-review finding and the change is material: the prior code-review is stale. A new code-review is required before normal progression. At the round limit, STOP and escalate.

#### Non-material implementation corrections

If post-review implementation corrections are classified as non-material: re-review may be skipped only with a clear justification recorded in the code-review artifact. Assess that justification semantically.

#### Timestamps

NEVER use filesystem timestamps as staleness evidence. Use revision/diff information where available. Fall back to conservative semantic comparison when a clean revision cannot represent the reviewed working tree.

## 5. Perform verification

After all prerequisites pass, perform the following checks:

### 5.1 Spec compliance

For each spec file in the change's specs/ directory:

- Read each `#### Scenario:` block
- Compare the implementation against the scenario's WHEN/THEN behavior
- Report any unmet scenario as a finding

### 5.2 Test integrity

- Run the full test suite (the project's standard test command)
- Confirm it passes
- Check that no tests were removed, weakened, or replaced in a way that reduces required behavioral coverage without a corresponding spec change

### 5.3 Three validation types

For each test-plan entry:

- AUTOMATED: verify the recorded test passes
- MECHANICAL: verify the recorded command/check passes
- SEMANTIC: verify a documented evaluation exists

### 5.4 Behavioral coverage preservation

Confirm that required behavioral coverage from the specs is retained in the implementation and tests. Any coverage reduction requires a corresponding spec amendment.

## 6. Produce verify artifact

Use `openspec instructions verify --change "<name>" --json` for the current artifact contract.

Write the verify artifact at the provided path with:

| Field | Value |
|---|---|
| Decision | PASS \| PASS_WITH_WARNINGS \| FAIL |
| Reviewer Context Status | Degraded |
| Implementer Context Status | Degraded |
| Reviewer Context Rationale | Cannot independently prove upstream context isolation |
| Implementer Context Rationale | Cannot independently prove upstream context isolation |
| Reviewed Revision | Current HEAD commit (when available) |
| Reviewed Paths | All source, test, and configuration files in the change |
| Findings | Any spec compliance, test integrity, or coverage issues |
| Warnings | Non-blocking issues that do not force FAIL |
| Review Freshness | Staleness assessment for plan-review and code-review |
| Override State | Any overrides assessed during prerequisite checks |
| Escalation State | Any Round 2 unresolved conditions |

### 6.1 Decision rules

**PASS**: all checks pass with no issues.

**PASS_WITH_WARNINGS**: checks pass but non-blocking warnings exist. WARNINGS SHALL NOT silently mean PASS. Archive MAY proceed only if the human explicitly accepts or defers the warnings.

**FAIL**: blocking issues found (unmet spec scenarios, failing tests, incomplete triage, stale reviews without override, unevaluable test-plan entries, behavioral coverage reduction). Archive SHALL NOT proceed. Human override MUST be explicit and recorded.

### 6.2 Degraded review note

Record that both reviewer and implementer contexts are Degraded. This does not block verification itself, but it means the verification does not carry the assurance of truly independent review and implementation. The human must understand this limitation when deciding whether to accept the PASS.

## 7. Required output after verification

Report:

- change name, schema, planning home/store
- DECISION: PASS | PASS_WITH_WARNINGS | FAIL
- Reviewer Context Status: Degraded
- Implementer Context Status: Degraded
- prerequisite check results (each gate: pass/block)
- spec compliance findings (if any)
- test integrity results (tests passing, behavioral coverage)
- review freshness assessment
- override state (if any)
- escalation state (if any)
- next required action:
  - PASS: instruct human to run goulart-archive
  - PASS_WITH_WARNINGS: instruct human to accept/defer warnings, then goulart-archive
  - FAIL: instruct human to address blocking issues, then re-run goulart-verify

Always STOP after verification.

## 8. Keep skill usable by local/cheap models

This skill will often be executed by smaller/local models. Make behavior explicit.

Prefer: ordered state checks, small decision tables, exact permitted values, explicit STOP conditions, concrete required evidence.

Avoid: vague "use judgment" where a gate can be explicit, duplicated prose, model/vendor-specific assumptions, hidden state, reliance on another LLM vendor.

Do NOT make any specific model a methodology requirement.
