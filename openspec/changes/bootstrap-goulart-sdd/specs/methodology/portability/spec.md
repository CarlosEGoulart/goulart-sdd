## Purpose

Defines the agent-agnostic methodology design that supports multiple coding-agent integrations and inexpensive or local models.

## ADDED Requirements

### Requirement: No mandatory LLM vendor
Goulart SDD SHALL NOT mandate a particular LLM vendor, model, or API provider. The methodology SHALL remain usable with local, free, or inexpensive models.

#### Scenario: Vendor independence
- **WHEN** a user adopts Goulart SDD
- **THEN** they SHALL NOT be required to use a specific LLM provider

### Requirement: Role-tier model support
The methodology SHOULD support an architecture where planning and review use stronger reasoning models, implementation uses cheaper models, and verification uses lightweight models.

#### Scenario: Model tier separation
- **WHEN** different roles use different models
- **THEN** the methodology SHALL NOT require all roles to use the same model

### Requirement: Fresh-context independence prioritized over cross-model
Fresh-context independence SHALL be prioritized over cross-model review. Cross-model review MAY be recommended but SHALL NOT be mandatory. The fallback hierarchy for achieving reviewer independence is:

1. Harness-created fresh context (preferred)
2. User-managed fresh session (fallback)
3. Degraded review mode (last resort — explicitly disclosed, human-acknowledged, NOT described as independent)

#### Scenario: Same-model fresh context
- **WHEN** only one model is available
- **THEN** the methodology SHALL still function with fresh-context review on the same model

#### Scenario: Degraded mode disclosure
- **WHEN** reviewer independence cannot be achieved
- **THEN** the limitation SHALL be explicitly disclosed in the review artifact
- **THEN** the human SHALL acknowledge the degraded independence

### Requirement: OpenCode as first reference adapter
OpenCode SHALL be the first coding-agent integration implemented. The methodology SHALL NOT be OpenCode-specific; the schema and methodology docs SHALL be agent-agnostic.

#### Scenario: Adapter portability
- **WHEN** a user wants to use Goulart SDD with a different coding agent
- **THEN** they SHALL be able to create a new adapter without modifying the methodology or schema

### Requirement: Adapter entry points
The OpenCode adapter SHALL implement entry points for: `goulart-plan`, `goulart-review`, `goulart-apply`, `goulart-verify`, `goulart-archive`. Fewer files may implement these concepts cleanly if the design demonstrates that combined entry points are sufficient.

#### Scenario: Adapter covers full lifecycle
- **WHEN** a user follows the Goulart SDD workflow
- **THEN** the adapter entry points SHALL cover planning, review, implementation, verification, and archive stages

### Requirement: Graceful degradation
When a coding-agent harness does not support a feature (fresh context, cross-model review, automated TDD), the methodology SHALL degrade gracefully with clear documentation of what is lost. The degradation MUST be explicitly disclosed and the human MUST acknowledge it.

#### Scenario: Missing harness feature
- **WHEN** the coding-agent harness cannot spawn fresh contexts
- **THEN** the methodology SHALL still function
- **THEN** the limitation SHALL be documented
- **THEN** the human SHALL acknowledge the degraded independence
