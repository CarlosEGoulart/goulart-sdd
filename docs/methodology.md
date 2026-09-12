# Goulart SDD Methodology

Goulart SDD is an opinionated Spec-Driven Development methodology on top of [OpenSpec](https://github.com/openspec-dev/openspec). It enforces a strict lifecycle where planning, review, implementation, verification, and archival are separated into distinct roles and gated by human decisions.

## Workflow Overview

```
goulart-plan (initial)
  → proposal → specs → design → STOP

goulart-review plan (fresh session)
  → independent plan review → verdict → human disposition → STOP

goulart-plan (later)
  → complete review gate → test-plan → coverage gate → tasks → STOP

goulart-apply (one task at a time)
  → TDD: RED → GREEN → REFACTOR → mark complete → STOP

goulart-review code (fresh session)
  → independent code review → verdict → human triage → STOP

goulart-verify
  → prerequisite checks → spec compliance → test integrity → DECISION → STOP

goulart-archive
  → decision-gated delegation to OpenSpec archive → STOP
```

## Roles

| Role | Adapter | Responsibility |
|---|---|---|
| Author | goulart-plan | Creates planning artifacts (proposal, specs, design, test-plan, tasks) |
| Reviewer | goulart-review | Evaluates plans and code independently, records findings and verdicts |
| Implementer | goulart-apply | Executes tasks one at a time using TDD |
| Verifier | goulart-verify | Audits completeness, spec compliance, test integrity, and review freshness |
| Archiver | goulart-archive | Delegates to OpenSpec archive after verify passes |

## Gates

Every stage transition is gated:

1. **Planning gate**: proposal/specs/design complete → STOP → fresh-session plan review
2. **Review gate**: plan-review verdict + human acceptance → later goulart-plan can proceed to test-plan/tasks
3. **Implementation gate**: plan-review verdict, human acceptance, test-plan existence → goulart-apply can execute tasks
4. **Code review gate**: all tasks complete → fresh-session code review
5. **Verification gate**: code-review verdict, triage complete, tasks complete, test-plan evaluable, reviews fresh → verify DECISION
6. **Archive gate**: verify PASS (or PASS_WITH_WARNINGS with human acceptance) → archive

## State-Aware Planning Continuation

`goulart-plan` is invoked twice:

1. **Initial**: creates proposal, specs, design → STOP
2. **Later**: after plan review + human acceptance, creates test-plan and tasks → STOP

The same skill handles both states. It reads the current artifact state to determine which phase applies. No separate "continue" or "resume" command is needed.

## Raw OpenSpec Escape Hatch

If the Goulart gates are too restrictive for a particular use case, users may bypass Goulart and use OpenSpec commands directly (raw execution). This is explicitly allowed but loses all Goulart guarantees: independent review, TDD enforcement, verification prerequisites, and staleness detection.

## One-Task-Per-Invocation TDD

`goulart-apply` processes exactly one task per invocation:

1. Verify pre-implementation gates
2. Select first eligible pending task
3. Load relevant context (task, specs, design, test-plan)
4. RED → GREEN → REFACTOR
5. Mark task complete
6. STOP

The user invokes `goulart-apply` again for the next task. This ensures fresh context per task and prevents context pollution.

## Independent and Degraded Review

`goulart-review` enforces independence:

- **Preferred**: harness-supplied fresh context
- **Fallback**: user-managed clean session
- **Last resort**: degraded review (disclosed, acknowledged, never described as independent)

Cross-model review is optional. Same model + fresh session MAY be independent. Different model + author history is NOT independent.

## Verdicts and Dispositions

### Plan Review
- **APPROVE**: plan acceptable as-is → human records STATUS: ACCEPTED
- **APPROVE_WITH_CHANGES**: Required Changes needed → each Applied: Yes/No + Materiality
- **REVISE**: plan requires revision → STOP

### Code Review
- **APPROVE**: clean review or non-blocking findings → human triage required for findings
- **APPROVE_WITH_CHANGES**: implementation changes needed → human triage + required changes
- **REVISE**: implementation revision needed → STOP

### Verify
- **PASS**: all checks pass → archive may proceed
- **PASS_WITH_WARNINGS**: non-blocking warnings → human accepts/defers → archive may proceed
- **FAIL**: blocking issues → archive SHALL NOT proceed

## Two-Round Staleness Rules

Both plan-review and code-review have independent round counters:

- **ROUND 1**: first review
- **ROUND 2**: follow-up after REVISE, material corrections, or material state change
- **ROUND 3**: NEVER — escalate to human if Round 2 unresolved

Staleness is detected via revision/diff comparison (preferred) or conservative semantic comparison (fallback). Filesystem timestamps are NEVER used as staleness evidence.

## Verification Prerequisites

`goulart-verify` checks all of the following before performing verification:

1. code-review.md exists with permitting verdict
2. Required human triage complete for all findings
3. All tasks in tasks.md complete
4. Required test-plan entries have final evaluable state
5. Review freshness verified (revision-based or semantic fallback)
6. Any explicit override recorded with mandatory reason

Both reviewer and implementer contexts are unconditionally forced to Degraded status during verification.
