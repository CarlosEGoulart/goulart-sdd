# Test Plan

## Coverage Ledger

| ID | Spec Path | Requirement | Scenario | Validation Type | Test File/Command | Test Name | Status |
|---|---|---|---|---|---|---|---|
| TP-001 | specs/dogfood-smoke-command/spec.md | Deterministic smoke result | Successful smoke invocation | AUTOMATED | tests/dogfood-smoke-command.sh | dogfood_smoke_returns_deterministic_output | NOT_STARTED |
| TP-002 | specs/dogfood-smoke-command/spec.md | No repository mutation | Repository unchanged after invocation | MECHANICAL | `test -z "$(git status --short)" && opencode run --command goulart-dogfood-smoke >/dev/null && test -z "$(git status --short)"` | repo-unchanged-check | NOT_STARTED |
| TP-003 | specs/dogfood-smoke-command/spec.md | Temporary fixture lifecycle | Fixture is identifiable as temporary | SEMANTIC | (human inspection of command file) | fixture-temporary-header-review | NOT_STARTED |

### Validation Types

- **AUTOMATED**: Scenario verified by an automated test (unit, integration, E2E). Record test file path and test function/method name.
- **MECHANICAL**: Scenario verified by a deterministic command or tool (lint, typecheck, schema validation, build, static checks). Record the command or mechanical check.
- **SEMANTIC**: Scenario cannot reasonably be validated automatically or mechanical. Record why automation is unsuitable, who or what performs the evaluation, and sufficient evaluation description for later verification.

## Scenario Mapping Rules

- Every `#### Scenario:` from every spec file must appear in this ledger.
- One scenario may have multiple validation entries where useful.
- Zero validation entries for a scenario is invalid and blocks downstream task creation.
- Duplicate scenario names across different specs remain distinguishable through Spec Path + Requirement + Scenario.
- Do not pre-populate actual project scenarios. Use placeholder rows only where necessary to demonstrate structure.

## Semantic Entry Detail

| ID | Semantic Rationale | Semantic Evaluator | Evaluation Description |
|---|---|---|---|
| TP-003 | The requirement mandates that a maintainer inspects the file. Deterministic command evaluation could check file content but the spec's "inspects" language and the need for human judgment about the header's clarity and placement make semantic evaluation more appropriate. | Maintainer (human reviewer) | A reviewer opens the command file and confirms it contains a clearly visible comment or header identifying it as a temporary dogfooding fixture scheduled for removal. The header must be unambiguous and prominently placed. |

## Coverage Summary

Total Scenarios: 3
Mapped Scenarios: 3
Unmapped Scenarios: 0

## Coverage Integrity

| Entry | Change | Coverage Impact | Spec Amendment | Rationale |
|---|---|---|---|---|
| (none) | | | | |
