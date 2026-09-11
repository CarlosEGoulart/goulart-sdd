## Why

AI coding assistants are powerful but unpredictable when requirements live only in chat history. OpenSpec adds a spec layer, but its default workflow lacks engineering discipline: no adversarial review, no TDD enforcement, no independent verification, and no bounded review loops. Goulart SDD adds an opinionated methodology layer that combines these practices into a coherent workflow suitable for personal, academic, portfolio, and long-lived side projects — without modifying the OpenSpec upstream engine.

## What Changes

- Introduce a custom OpenSpec schema (`goulart-sdd`) that extends the default `spec-driven` workflow with additional artifacts: plan-review, test-plan, code-review, and verify.
- Create Goulart-owned adapter entry points (`goulart-plan`, `goulart-apply`, `goulart-review`, `goulart-verify`, `goulart-archive`) that enforce methodology sequencing and execution gates without modifying OpenSpec.
- Define explicit human acceptance gates after plan-review and code-review.
- Define clear ownership boundaries between OpenSpec upstream and Goulart SDD methodology layer.
- Document the methodology, getting-started guide, and architectural decisions.
- Dogfood the methodology by using OpenSpec to design Goulart SDD itself.

## Capabilities

### New Capabilities

- `methodology/workflow`: The strict Goulart SDD workflow — artifact lifecycle, dependency graph, gate definitions, role model, and adapter sequencing.
- `methodology/ownership`: Ownership boundaries between OpenSpec upstream, Goulart SDD schema, coding-agent adapters, and CI enforcement.
- `methodology/plan-review`: Pre-implementation adversarial review with explicit human acceptance that gates downstream artifacts.
- `methodology/test-planning`: Test-plan artifact mapping scenarios to validation entries (automated, mechanical, or semantic) with behavioral coverage tracking.
- `methodology/tdd-execution`: TDD-oriented apply workflow with fresh implementer context, codebase access, and bounded retry loops.
- `methodology/code-review`: Post-implementation independent code review with human triage of findings.
- `methodology/verification`: Final whole-change audit before archive — spec compliance, test integrity, review completeness, and structured decision with clear archive semantics.
- `methodology/portability`: Agent-agnostic methodology design supporting multiple coding-agent integrations and inexpensive/local models.

### Modified Capabilities

(none — this is a greenfield methodology, no existing specs to modify)

## Impact

- Only the `goulart-sdd` repository itself is affected.
- No upstream OpenSpec code is modified or vendored.
- No production application code exists yet; this change establishes the methodology framework.
- Future users will add `openspec/schemas/goulart-sdd/` and Goulart OpenCode skills to their own projects.
