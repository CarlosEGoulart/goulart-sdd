---
name: goulart-apply
description: Use when executing Goulart SDD implementation tasks one at a time with TDD, fresh context, bounded retry, and pre-implementation gate checks.
compatibility: Requires OpenSpec CLI and a Goulart-compatible schema.
metadata:
  author: goulart-sdd
  version: "1.0"
---

# Goulart Apply

One task per invocation:

```text
goulart-apply
  → verify pre-implementation gates
  → select first eligible pending task
  → load relevant task/spec/design/test-plan context
  → allow repository exploration as needed
  → RED → GREEN → REFACTOR
  → mark that task complete
  → report result
  → STOP

goulart-apply (subsequent)
  → verify pre-implementation gates (cached if unchanged)
  → select next eligible pending task
  → execute
  → STOP

goulart-apply (all complete)
  → report completion
  → instruct: run goulart-review code in a fresh session
  → STOP
```

## 1. Authorization and ownership

This is the Implementer context. You write only implementation source files, test files, and configuration changes required by the selected task.

FORBIDDEN actions:

- modify proposal.md, specs/, design.md, test-plan.md, plan-review.md
- modify code-review.md, verify artifact, archive artifact
- modify schemas, templates, project configuration unrelated to the task
- modify upstream `opsx-*` commands / `openspec-*` skills
- execute review, verification, or archive
- invoke goulart-plan, goulart-review, goulart-verify, goulart-archive
- add `goulart-continue`, `goulart-resume`, or other continuation commands
- batch multiple tasks into a single invocation unless the harness explicitly spawns isolated subcontexts per task

## 2. Resolve scope and select the change

1. Run `openspec context --json`. If the user names a registered store or the work is in one, discover its id using `openspec store list --json` and use `openspec context --json --store "<id>"`. Once selected, keep `--store "<id>"` on every subsequent `context`, `list`, `schemas`, `new change`, `status`, `instructions`, `show`, or `validate` command.
2. Use the returned `root.path`. On `no_openspec_root`, STOP and report initialization is needed. On other context errors, STOP with the actual error.
3. Read `<root.path>/openspec/config.yaml`. Apply a valid string `context` field up to 51,200 UTF-8 bytes as project constraints. If absent/invalid, report the limitation.
4. Resolve the change: use an explicitly named existing change, or an unambiguous existing change identified in conversation. Otherwise run `openspec list --json`. With multiple candidates, ASK. Do not choose by timestamp.
5. Run `openspec status --change "<name>" --json` without a schema override. Use `planningHome`, `changeRoot`, `artifactPaths`, and `actionContext` as authoritative paths/scope.
6. Confirm schema compatibility: it must support proposal, specs, design, plan-review, test-plan, tasks, code-review and verify. If incompatible, STOP and report.

## 3. Pre-implementation gate checks

Before selecting any task, verify all of the following. If any gate fails, STOP and report the exact unresolved gate.

### 3.1 Planning completeness

Run `openspec status --change "<name>" --json`.

Required: proposal, specs, design, test-plan, tasks artifacts must all be present (status not `missing`).

If any is missing: STOP. Report which artifact is missing. Next action: return to goulart-plan.

### 3.2 Plan-review verdict

If plan-review.md exists and has a verdict:

- APPROVE: check that STATUS: ACCEPTED is recorded (human disposition)
- APPROVE_WITH_CHANGES: check all Required Changes are Applied: Yes AND STATUS: ACCEPTED is recorded
- REVISE: STOP. Plan requires revision before implementation.

If plan-review.md is missing: STOP. Plan review has not been performed. Return to goulart-plan.

If plan-review verdict or human STATUS is missing/pending: STOP. Report exact missing field.

### 3.3 Test-plan existence

test-plan.md must exist with at least one entry.

If missing: STOP. Return to goulart-plan.

### 3.4 Tasks artifact

tasks.md must exist with at least one unchecked task checkbox.

If all tasks are checked: report completion. Instruct the user to run `goulart-review code` in a fresh session. STOP.

## 4. Task selection

From tasks.md, select the first unchecked task (`- [ ]`).

Rules:
- Process tasks in order as listed (top to bottom)
- Skip checked tasks (`- [x]`) — they are already complete
- Do not skip unchecked tasks to work on a "more interesting" one
- If no unchecked tasks exist, all tasks are complete — follow the completion path in Section 9

## 5. Load task context

For the selected task, load:

1. **Task description**: the full text of the selected task checkbox line
2. **Relevant specs**: read spec files referenced by the task or by the change's specs/ directory. Focus on scenarios directly related to this task's scope
3. **Design decisions**: read design.md for architecture choices relevant to this task
4. **Test-plan entries**: read test-plan.md for entries related to this task's scope
5. **Existing implementation**: read any source files the task will modify or depend on

Do not load unrelated task history or implementation context from prior tasks. Keep context focused.

## 6. Repository exploration

You MAY inspect any repository file to understand implementation dependencies, existing patterns, conventions, libraries, and test infrastructure.

Before writing code, confirm:
- the project's language, framework, and test framework
- existing code conventions (imports, naming, error handling, test structure)
- relevant dependencies in package.json, cargo.toml, go.mod, pyproject.toml, or equivalent

Never assume a library is available. Check the actual dependency manifest.

## 7. TDD execution

Follow RED → GREEN → REFACTOR per task.

### 7.1 RED

Write a failing test that exercises the behavior described in the task and its linked spec scenarios.

- Confirm the test fails before writing implementation
- The test must fail for the right reason (not a syntax/import error)
- If a test cannot be written (behavior is not testable, e.g., schema-level YAML), document why and proceed to GREEN with the appropriate validation type (MECHANICAL or SEMANTIC)

### 7.2 GREEN

Write the minimum implementation that makes the test pass.

- Do not add behavior beyond what the task requires
- Do not add speculative features or "nice to have" improvements
- Confirm all tests pass

### 7.3 REFACTOR

Refactor while the test suite remains green.

- Remove duplication
- Improve clarity
- Simplify where appropriate
- Confirm tests still pass after refactoring

### 7.4 Bounded TDD retry

If the same test fails twice consecutively for the same root cause within a single task:

STOP that task.

Report:
- the failing test
- the root cause hypothesis
- what was attempted
- what remains unresolved

Do NOT attempt a third approach. The issue may indicate a spec problem, a design flaw, or a misunderstanding that requires human input.

## 8. Spec drift handling

If implementation reveals a requirement, scenario, or design assumption is wrong, incomplete, or untestable:

STOP implementing that scenario.

Report:
- the scenario that is wrong/incomplete/untestable
- what was discovered
- the affected spec path

Do NOT silently edit the spec. Do NOT weaken the test to make it pass. Do NOT implement a workaround without a spec amendment.

The spec SHALL be amended (by the author, via goulart-plan) before implementation resumes.

## 9. Mark task complete and report

After successful implementation and test passage:

1. Mark the task checkbox as complete in tasks.md: change `- [ ]` to `- [x]` for this task only
2. Report what was implemented
3. Report test results (tests written, tests passing)
4. Report any test-plan entries that were updated
5. Report any spec drift issues discovered (even if resolved)

Then STOP.

Do NOT automatically select the next task. The user must invoke goulart-apply again for the next task.

## 10. All tasks complete

When goulart-apply is invoked and all task checkboxes are checked:

1. Report that all tasks are complete
2. Instruct the user to run `goulart-review code` in a fresh session
3. STOP

Do NOT invoke goulart-review automatically. Do NOT proceed to verification. Do NOT archive.

## 11. Keep skill usable by local/cheap models

This skill will often be executed by smaller/local models. Make behavior explicit.

Prefer: ordered state checks, small decision tables, exact permitted values, explicit STOP conditions, concrete required evidence.

Avoid: vague "use judgment" where a gate can be explicit, duplicated prose, model/vendor-specific assumptions, hidden state, reliance on another LLM vendor.

Do NOT make any specific model a methodology requirement.
