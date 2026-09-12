---
name: goulart-archive
description: Use when archiving a completed Goulart SDD change after verify passes, with decision-gated delegation to OpenSpec archive.
compatibility: Requires OpenSpec CLI and a Goulart-compatible schema.
metadata:
  author: goulart-sdd
  version: "1.0"
---

# Goulart Archive

One archive per invocation:

```text
goulart-archive
  → verify artifact exists and has decision
  → check decision: PASS | PASS_WITH_WARNINGS | FAIL
  → if PASS: delegate to openspec archive
  → if PASS_WITH_WARNINGS: confirm human warning dispositions recorded, then delegate
  → if FAIL: block, require human override with reason
  → STOP
```

## 1. Authorization and ownership

This is the Archiver context. You read the verify artifact and delegate to OpenSpec archive.

FORBIDDEN actions:

- modify proposal.md, specs/, design.md, test-plan.md, tasks.md
- modify plan-review.md, code-review.md, verify artifact
- modify implementation/source files or tests
- invoke goulart-plan, goulart-review, goulart-apply, goulart-verify
- change schemas, templates, project configuration
- modify upstream `opsx-*` commands / `openspec-*` skills

## 2. Resolve scope and select the change

1. Run `openspec context --json`. If the user names a registered store or the work is in one, discover its id using `openspec store list --json` and use `openspec context --json --store "<id>"`. Once selected, keep `--store "<id>"` on every subsequent `context`, `list`, `schemas`, `new change`, `status`, `instructions`, `show`, or `validate` command.
2. Use the returned `root.path`. On `no_openspec_root`, STOP and report initialization is needed. On other context errors, STOP with the actual error.
3. Read `<root.path>/openspec/config.yaml`. Apply a valid string `context` field up to 51,200 UTF-8 bytes as project constraints. If absent/invalid, report the limitation.
4. Resolve the change: use an explicitly named existing change, or an unambiguous existing change identified in conversation. Otherwise run `openspec list --json`. With multiple candidates, ASK. Do not choose by timestamp.
5. Run `openspec status --change "<name>" --json` without a schema override. Use `planningHome`, `changeRoot`, `artifactPaths`, and `actionContext` as authoritative paths/scope.
6. Confirm schema compatibility: it must support proposal, specs, design, plan-review, test-plan, tasks, code-review and verify. If incompatible, STOP and report.

## 3. Check verify decision

Read the verify artifact at the path recorded by `openspec status`.

If verify artifact is missing: STOP. Report: verify artifact has not been produced. Next action: run goulart-verify.

If verify artifact exists, read its Decision field.

### 3.1 Decision: PASS

All checks passed with no issues.

Proceed to Section 4 (delegate to OpenSpec archive).

### 3.2 Decision: PASS_WITH_WARNINGS

Checks passed but non-blocking warnings exist.

Before proceeding, confirm that the human has explicitly accepted or deferred each warning. Check the verify artifact for evidence of human warning dispositions.

If human dispositions are not recorded: STOP. Report exact pending warnings. Next action: human must accept or defer each warning.

If all warnings have human dispositions: proceed to Section 4 (delegate to OpenSpec archive).

### 3.3 Decision: FAIL

Blocking issues found.

Report:
- the FAIL decision
- exact blocking issues (spec compliance failures, test failures, incomplete triage, stale reviews, unevaluable test-plan entries)
- next action: address blocking issues, then re-run goulart-verify

If the human requests an override: record the override with mandatory reason, affected condition, and the explicit acknowledgment that this bypasses normal verification. Then proceed to Section 4.

If no override: STOP. Archive SHALL NOT proceed.

## 4. Delegate to OpenSpec archive

After confirming the decision permits archiving, delegate to OpenSpec:

Run `openspec archive "<change-name>"` (or the appropriate store-scoped variant).

Do not modify the upstream archive command or its behavior. Report the result as OpenSpec produces it.

## 5. Required output after archive

Report:

- change name, schema, planning home/store
- verify decision
- human warning dispositions (if PASS_WITH_WARNINGS)
- override state (if FAIL override was used)
- archive result from OpenSpec
- confirmation that the change is archived

Always STOP after archive.

## 6. Keep skill usable by local/cheap models

This skill will often be executed by smaller/local models. Make behavior explicit.

Prefer: ordered state checks, small decision tables, exact permitted values, explicit STOP conditions, concrete required evidence.

Avoid: vague "use judgment" where a gate can be explicit, duplicated prose, model/vendor-specific assumptions, hidden state, reliance on another LLM vendor.

Do NOT make any specific model a methodology requirement.
