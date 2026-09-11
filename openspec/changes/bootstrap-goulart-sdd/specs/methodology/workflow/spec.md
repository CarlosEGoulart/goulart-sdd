## Purpose

Defines the strict Goulart SDD workflow lifecycle — the ordered sequence of artifacts, gates, operations, and adapter entry points from proposal through archive.

## ADDED Requirements

### Requirement: Goulart-compliant vs raw OpenSpec execution
Goulart SDD extends OpenSpec; it does not disable or replace upstream OpenSpec commands.

- **Goulart-compliant execution** uses `goulart-*` lifecycle entry points (goulart-plan, goulart-review, goulart-apply, goulart-verify, goulart-archive) and honors their gates and recorded human dispositions. Disclosed degraded review or explicit overrides SHALL NOT be represented as providing the guarantees they waive.
- **Raw OpenSpec execution** (direct use of `/opsx-propose`, `/opsx-apply`, `/opsx-archive`, `openspec ...`) remains available as an escape hatch. Raw execution MAY bypass Goulart execution gates. Raw execution SHALL NOT be represented as satisfying all Goulart SDD process guarantees.

Do NOT attempt to modify or disable upstream OpenSpec commands.

#### Scenario: Goulart-compliant plan
- **WHEN** a user runs `goulart-plan`
- **THEN** the adapter SHALL enforce planning sequencing (proposal, specs, design, stop for review, resume after acceptance)
- **THEN** the adapter SHALL report the planning state and any acknowledged limitations or overrides accurately

#### Scenario: Raw plan escape hatch
- **WHEN** a user runs `/opsx-propose` directly
- **THEN** OpenSpec MAY generate the entire planning artifact set required by the selected schema
- **THEN** it MAY therefore bypass Goulart sequencing, independent-review handoff, or human gates
- **THEN** the result SHALL NOT be represented as Goulart-compliant planning

### Requirement: Strict workflow sequence
Goulart SDD SHALL enforce the following conceptual sequence for changes using the `goulart-sdd` schema: proposal, specs, design, plan-review, human decision, test-plan, tasks, apply, code-review, verify, archive. Not every box is an OpenSpec artifact — some are operations, adapter commands, or human decisions.

#### Scenario: Planning stops for independent review
- **WHEN** the design artifact is created
- **THEN** the `goulart-plan` adapter SHALL stop and instruct the user to run `goulart-review plan` in a fresh session before proceeding to test-plan or tasks

#### Scenario: Workflow blocks out-of-order artifacts
- **WHEN** an agent attempts to create an artifact before its dependencies are satisfied
- **THEN** the workflow SHALL NOT proceed until all upstream dependencies exist

### Requirement: Three gate categories
The workflow SHALL distinguish artifact gates (OpenSpec dependency graph), execution gates (Goulart commands/skills), and repository gates (scripts/CI).

#### Scenario: Artifact gate enforcement
- **WHEN** artifact B requires artifact A
- **THEN** OpenSpec's schema engine SHALL prevent creation of B until A exists

#### Scenario: Execution gate enforcement
- **WHEN** a Goulart skill or command checks a pre-implementation gate
- **THEN** it SHALL verify the required artifact state before allowing execution to proceed

#### Scenario: Repository gate enforcement
- **WHEN** a CI check or script validates repository state
- **THEN** it SHALL verify structural invariants (required artifacts exist, verdicts are present) without claiming to prove agent cognition

### Requirement: Bounded review iteration
The workflow SHALL enforce a maximum of 2 review-revision rounds per review stage (plan-review and code-review independently), including re-reviews following APPROVE_WITH_CHANGES or material post-review changes. Reaching the limit SHALL NOT automatically start a third round or reset round history.

#### Scenario: Review escalation after two rounds
- **WHEN** round 2 leaves REVISE, unresolved blocking/required changes, or a material change requiring another review
- **THEN** the workflow SHALL STOP and escalate to the human with a clear summary of unresolved issues

### Requirement: Human override authority
The human SHALL retain final authority to approve, reject, defer, or override any advisory model conclusion at any gate.

#### Scenario: Human overrides reviewer
- **WHEN** the human disagrees with a reviewer's REVISE verdict
- **THEN** the human MAY instruct the workflow to proceed despite the verdict, and the decision SHALL be recorded with a reason

### Requirement: State-aware goulart-plan sequencing
The `goulart-plan` adapter entry point SHALL be state-aware. Its behavior depends on the current state of the change:

**Invocation while initial planning is incomplete** (proposal/specs/design not yet fully created):
- Create/continue proposal, specs, and design.
- STOP.
- Instruct user to run `goulart-review plan` in a fresh session.

**Later invocation when the plan-review gate permits continuation** (proposal/specs/design exist, plan-review and human disposition satisfy the plan-review verdict and staleness rules, including any explicitly recorded override):
- Create/continue test-plan.
- Create/continue tasks.
- Report planning complete.
- STOP.
- Instruct user that `goulart-apply` is the next Goulart-compliant execution step.

An already-complete planning state SHALL be reported without recreating completed artifacts. Initial planning SHALL stop after design even if it began in the same invocation; downstream planning requires a later invocation. A permitted degraded review still requires its disclosure and human acknowledgement and SHALL NOT be called independent.

**Invocation when a gate is unresolved** (plan-review missing, verdict pending, human disposition not recorded, or disposition blocks continuation):
- STOP.
- Report exactly what gate is unresolved.

Do NOT create `goulart-continue` or another lifecycle command unless a concrete technical limitation requires it. The public command surface SHALL remain minimal.

#### Scenario: Initial planning invocation
- **WHEN** goulart-plan is invoked and proposal/specs/design do not yet fully exist
- **THEN** the adapter SHALL create/continue proposal, specs, and design
- **THEN** it SHALL stop and instruct the user to run `goulart-review plan` in a fresh session

#### Scenario: Resume after accepted plan-review
- **WHEN** goulart-plan is invoked later and proposal/specs/design exist, the plan-review gate is satisfied under its verdict/staleness rules, and human disposition permits continuation
- **THEN** the adapter SHALL create/continue test-plan and tasks
- **THEN** it SHALL report planning complete
- **THEN** it SHALL stop and instruct the user that `goulart-apply` is the next step

#### Scenario: Gate unresolved
- **WHEN** goulart-plan is invoked and a required gate is unresolved (plan-review missing, verdict pending, or human disposition blocks)
- **THEN** the adapter SHALL stop and report exactly what gate is unresolved

### Requirement: Handoff to independent reviews
`goulart-plan` and `goulart-apply` MUST NOT perform their independent reviews inside their own author/implementer context. The normal handoff is:

```
goulart-plan → STOP → instruct user to run goulart-review plan in a fresh session
goulart-apply (last task complete) → STOP → instruct user to run goulart-review code in a fresh session
```

If the harness can supply a separate/fresh context for review, an adapter MAY automate that handoff. This capability SHALL NOT be assumed in the generic methodology.

#### Scenario: Plan handoff
- **WHEN** goulart-plan completes the initial proposal/specs/design phase
- **THEN** it SHALL stop and instruct the user to run `goulart-review plan` in a fresh session

#### Scenario: Code-review handoff
- **WHEN** goulart-apply completes all tasks
- **THEN** it SHALL stop and instruct the user to run `goulart-review code` in a fresh session
