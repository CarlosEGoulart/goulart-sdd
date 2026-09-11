## Why

AI coding assistants are powerful but unpredictable when requirements live only in chat history. OpenSpec adds a spec layer, but its default workflow lacks engineering discipline: no adversarial review, no TDD enforcement, no independent verification, and no bounded review loops. Goulart SDD adds an opinionated methodology layer that combines these practices into a coherent workflow suitable for personal, academic, portfolio, and long-lived side projects — without modifying the OpenSpec upstream engine.

## What Changes

- Introduce a custom OpenSpec schema (`goulart-sdd`) that extends the default `spec-driven` workflow with additional artifacts: pre-implementation review, test-plan mapping, TDD-oriented task ordering, post-implementation code review, and final verification.
- Create OpenCode skills that enforce the methodology: fresh-context reviewer, TDD-per-task implementation, post-implementation audit.
- Define clear ownership boundaries between OpenSpec upstream and Goulart SDD methodology layer.
- Document the methodology, getting-started guide, and architectural decisions.
- Dogfood the methodology by using OpenSpec to design Goulart SDD itself.

## Capabilities

### New Capabilities

- `methodology/workflow`: The strict Goulart SDD workflow — artifact lifecycle, dependency graph, gate definitions, and role model.
- `methodology/ownership`: Ownership boundaries between OpenSpec upstream, Goulart SDD schema, coding-agent adapters, and CI enforcement.
- `methodology/plan-review`: Pre-implementation adversarial review that gates downstream artifacts (test-plan, tasks, apply).
- `methodology/test-planning`: Test-plan artifact that maps specification scenarios to named automated tests and serves as a live coverage ledger.
- `methodology/tdd-execution`: TDD-oriented apply workflow — fresh implementer context per task, RED-GREEN-REFACTOR progression, bounded retry loops.
- `methodology/code-review`: Post-implementation independent code review against specs and design.
- `methodology/verification`: Final whole-change audit before archive — spec compliance, test integrity, review completeness, scope drift.
- `methodology/portability`: Agent-agnostic methodology design that supports multiple coding-agent integrations and inexpensive/local models.

### Modified Capabilities

(none — this is a greenfield methodology, no existing specs to modify)

## Impact

- Only the `goulart-sdd` repository itself is affected.
- No upstream OpenSpec code is modified or vendored.
- No production application code exists yet; this change establishes the methodology framework.
- Future users will add `openspec/schemas/goulart-sdd/` and OpenCode skills to their own projects.
