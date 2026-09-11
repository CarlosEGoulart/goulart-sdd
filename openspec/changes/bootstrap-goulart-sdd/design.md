## Context

Goulart SDD extends OpenSpec's `spec-driven` schema with engineering discipline. OpenSpec provides the artifact graph, dependency ordering, templates, validation, and lifecycle operations. Goulart SDD adds artifacts that OpenSpec does not have (plan-review, test-plan, code-review, verify), modifies instructions for existing artifacts (tasks with TDD ordering), and creates Goulart-owned adapter entry points that enforce methodology sequencing and execution gates.

The current OpenSpec version (1.13.0) supports custom schemas in `openspec/schemas/<name>/schema.yaml` with templates, instructions, and dependency graphs. OpenSpec's `requires:` field enforces artifact file existence but not semantic quality. All behavioral guarantees (TDD, review independence, human gates) are agent-honored, not mechanically enforced by the CLI.

See proposal.md - Why for motivation.

## Goals / Non-Goals

**Goals:**

- Define a custom OpenSpec schema that maps the Goulart SDD strict workflow onto OpenSpec's artifact model.
- Establish ownership boundaries that survive `openspec update` and upstream changes.
- Create Goulart-owned adapter entry points that enforce methodology sequencing.
- Create OpenCode skills as the first reference coding-agent adapter.
- Produce methodology documentation that is agent-agnostic.
- Dogfood the methodology by using OpenSpec to design itself.

**Non-Goals:**

- Modify the OpenSpec CLI or vendor its source code.
- Create a light/workflow profile (deferred).
- Create adapters for Claude Code, Cursor, GitHub Copilot, or Codex (deferred).
- Build CI enforcement scripts (deferred beyond basic structural checks).
- Prove TDD chronology from repository state.
- Create per-task code review (only per-change review in v0.1).
- Mandate specific LLM providers or models.

## Lifecycle Diagram

```
AUTHOR / goulart-plan
        |
        |-- proposal (OpenSpec artifact)
        |-- specs    (OpenSpec artifact)
        |-- design   (OpenSpec artifact)
        |
        v
       STOP -- goulart-plan enforces this stop
        |
        v
   REVIEWER (fresh context)
        plan-review (OpenSpec artifact)
        findings + VERDICT
        |
        v
    HUMAN DECISION
    STATUS: ACCEPTED | REVISE | OVERRIDDEN
        |
    accepted?
     /     \
   no       yes
    |        |
  revise     v
          test-plan (OpenSpec artifact)
              |
             tasks (OpenSpec artifact)
              |
              v
         goulart-apply (adapter command)
              |
         focused TDD tasks
         one at a time, fresh context
         codebase access for dependencies
              |
              v
         code-review (OpenSpec artifact)
         fresh reviewer
              |
              v
         HUMAN TRIAGE (where findings exist)
              |
              v
           verify (OpenSpec artifact)
         spec compliance
         test integrity
         review staleness
         DECISION: PASS | PASS_WITH_WARNINGS | FAIL
              |
              v
         goulart-archive (adapter command)
              |
         validate Goulart gates
              |
              v
         OpenSpec archive (upstream operation)
```

## Ownership Model

```
OpenSpec engine (upstream, unmodified)
      |
      v
Goulart methodology (schema + docs + config)
      |
      v
OpenCode adapter (goulart-* skills/commands)
```

- **OpenSpec engine**: CLI, change lifecycle, delta specs, schema engine, artifact dependency graph, archive operation, validation, generated `opsx-*` commands and `openspec-*` skills.
- **Goulart methodology**: Custom schema (`openspec/schemas/goulart-sdd/`), methodology documentation (`docs/`), project config (`openspec/config.yaml`).
- **OpenCode adapter**: Goulart-owned entry points (`.opencode/commands/goulart-*`, `.opencode/skills/goulart-*`).

## Gate Categories

### Artifact gates

Provided by OpenSpec's schema/dependency graph (`requires:` field).

- Enforce artifact file existence and ordering.
- Do NOT prove artifact semantic quality.
- Do NOT prove implementation completeness.

### Execution gates

Provided by Goulart adapter commands and skills.

- `goulart-plan`: stops after design, requires independent review.
- `goulart-apply`: checks pre-implementation gates (plan-review verdict, human acceptance, test-plan existence, task completion).
- `goulart-verify`: checks post-implementation state (code-review exists, verify decision).
- `goulart-archive`: checks verify decision permits archive.
- These are not necessarily provable from Git history.

### Repository gates

Provided by scripts, tests, CI.

- OpenSpec validation, automated tests, lint, schema validity, structural artifact checks.
- Validate repository state but generally cannot prove agent cognition or process chronology.

### Human decisions

Explicitly NOT a mechanical gate. The human retains final authority to accept, reject, defer, or override at any point.

## Decisions

### D1: Schema as the extension mechanism

**Decision:** Create a custom schema in `openspec/schemas/goulart-sdd/schema.yaml` rather than modifying the upstream `spec-driven` schema.

**Rationale:** OpenSpec's schema system is the designed extension point. Custom schemas support new artifacts, modified templates, custom instructions, and different dependency graphs. Forking `spec-driven` via `openspec schema fork spec-driven goulart-sdd` gives us a working starting point.

**Alternatives considered:**
- Modify `spec-driven` directly: rejected because it would be overwritten by `openspec update`.
- Use only project config (rules/context): rejected because config cannot add new artifacts or change the dependency graph.

### D2: Artifact graph mapping

**Decision:** Map the Goulart SDD workflow to these OpenSpec artifacts:

```
proposal -> specs -> design -> plan-review -> test-plan -> tasks -> code-review -> verify
```

Where:
- `proposal`, `specs`, `design`, `tasks` are standard OpenSpec artifacts (reused from `spec-driven`).
- `plan-review` is a new artifact (requires: proposal, specs, design).
- `test-plan` is a new artifact (requires: specs, plan-review).
- `tasks` depends on test-plan and design and plan-review (modified from spec-driven).
- `code-review` is a new artifact (requires: tasks).
- `verify` is a new artifact (requires: code-review).

Key distinction: `requires:` proves artifact existence/order but does NOT prove all implementation tasks are complete before code-review. The `goulart-apply` adapter checks task completion before triggering code-review.

### D3: Goulart adapter entry points

**Decision:** Create Goulart-owned adapter entry points that enforce methodology sequencing:

- `goulart-plan`: orchestrates proposal/specs/design, then stops for independent review.
- `goulart-review`: invokes fresh-context review for both plan-review and code-review stages.
- `goulart-apply`: checks pre-implementation gates, then delegates to OpenSpec apply for task execution.
- `goulart-verify`: checks post-implementation state, produces verify artifact.
- `goulart-archive`: checks verify decision, then delegates to OpenSpec archive.

**Rationale:** OpenSpec owns artifact creation mechanics; Goulart adapters own methodology sequencing. This is an execution/workflow guarantee, not a new OpenSpec engine feature.

**Alternatives considered:**
- Rely only on agent instructions to follow the sequence: rejected because instructions alone are too easily bypassed.
- Create a new OpenSpec engine feature: rejected because it requires forking/modifying OpenSpec.

### D4: Review as single artifact, not two reviewers

**Decision:** One review artifact per review stage (plan-review, code-review), with findings tagged by category (compliance, quality, feasibility, scope, risk).

**Rationale:** For personal and small academic projects, two separate review artifacts create disproportionate ceremony. One reviewer checking both spec compliance and code quality in a single pass is sufficient.

### D5: Human decision after plan-review

**Decision:** After plan-review, the human records a disposition: ACCEPTED, REVISE, or OVERRIDDEN. Reviewer approval alone is NOT sufficient to proceed.

**Rationale:** This ensures the human remains the final gate. An override MUST include a reason for traceability.

### D6: Three validation types in test-plan

**Decision:** Each test-plan entry declares one of: AUTOMATED, MECHANICAL, or SEMANTIC.

- AUTOMATED: unit/integration/E2E tests with file path and function name.
- MECHANICAL: tool-based checks (lint, typecheck, schema validation) with command.
- SEMANTIC: when automation is unsuitable, with justification and evaluator identification.

**Rationale:** The previous design forced every scenario into an automated test, which is too rigid for methodology-level and semantic guarantees (e.g., reviewer independence, architectural judgment).

### D7: Verify depends on code-review

**Decision:** `verify` requires `code-review` in the artifact graph. The `goulart-verify` adapter additionally checks that code-review exists before proceeding.

**Rationale:** This prevents verification from proceeding without an independent code review. The artifact graph enforces ordering; the adapter enforces the semantic gate.

### D8: Behavioral test integrity

**Decision:** Tests SHALL NOT be removed, weakened, skipped, or replaced in a way that reduces required behavioral coverage without an explicitly justified specification change. Verification focuses on retained behavior/scenario coverage, not file identity.

**Rationale:** The previous rule ("no test removal without REMOVED requirement") was too rigid. Tests may legitimately be renamed, consolidated, replaced, rewritten, or moved as long as behavioral coverage is preserved.

### D9: Review staleness — two lineages

**Decision:** Verify distinguishes two review staleness lineages:

- **Plan-review staleness**: material changes to proposal/specs/design after the plan-review verdict require a new plan-review.
- **Code-review staleness**: material changes to source code/tests/config after the code-review verdict require a new code-review.

**Rationale:** The two reviews cover different artifacts at different lifecycle stages. Staleness detection is partially mechanical (file timestamps) + semantic (materiality judgment).

### D10: Verify decision semantics

**Decision:** PASS permits archive. PASS_WITH_WARNINGS permits archive only if warnings are non-blocking and the human explicitly accepts/defers them. FAIL blocks archive unless the human explicitly overrides.

**Rationale:** PASS_WITH_WARNINGS is not identical to PASS. Silently treating warnings as acceptable defeats their purpose.

### D11: TDD as process guarantee, not mechanical proof

**Decision:** TDD (RED-GREEN-REFACTOR) is a methodology requirement enforced by agent workflow instructions and human oversight. The test-plan tracks status for visibility but SHALL NOT be described as proof that RED chronologically occurred before GREEN.

**Rationale:** Building heavyweight evidence systems for TDD chronology is disproportionate ceremony for solo developers.

### D12: Implementer codebase access

**Decision:** The implementer SHALL receive minimal conversational/history context but SHALL be allowed to inspect all repository files necessary to implement the task correctly. The isolation goal is to reduce irrelevant reasoning/history contamination, not to blind the implementer.

**Rationale:** Previous wording that restricted the implementer to only task/spec/design/test-plan would prevent understanding existing implementation dependencies.

### D13: OpenCode-first adapter with agent-agnostic core

**Decision:** OpenCode is the first reference adapter. The schema, methodology docs, and config are agent-agnostic. Skills are OpenCode-specific but their content can be adapted.

### D14: Ownership layout

**Decision:** Separate namespaces:
- `openspec/schemas/goulart-sdd/` — schema owned by Goulart SDD.
- `.opencode/commands/goulart-*` — commands owned by Goulart SDD.
- `.opencode/skills/goulart-*` — skills owned by Goulart SDD.
- `scripts/gates/` — CI scripts owned by Goulart SDD.
- `docs/` — methodology documentation.
- `.opencode/commands/opsx-*` and `.opencode/skills/openspec-*` — OpenSpec-owned, never modified.

### D15: Task granularity for limited context

**Decision:** Tasks SHALL be small enough for one coding-agent session. Each task maps to one acceptance criterion or one test-plan entry.

## Risks / Trade-offs

- **[Prompt-only TDD enforcement]** -> Mitigation: Test-plan tracks status for visibility; verify checks final suite; human reviews task commits. Cannot prove chronology from repo state alone.
- **[Reviewer independence is instructed, not enforced]** -> Mitigation: Fresh-context instruction in skills; adversarial posture in review template; read-only posture. Human remains the final gate.
- **[Schema maintenance as OpenSpec evolves]** -> Mitigation: Fork from `spec-driven`; use only standard schema fields; `openspec schema fork` makes rebasing easy.
- **[OpenCode lock-in for skills]** -> Mitigation: Schema and methodology docs are agent-agnostic; skills are the first adapter, not the only one.
- **[Excessive ceremony for very small changes]** -> Mitigation: `skip_specs: true` exists for spec-less changes. Future: a light profile.
- **[Verify feels redundant with archive validation]** -> Mitigation: Verify is a pre-archive audit checking spec compliance + test integrity + review completeness. Archive only checks delta spec formatting. Different purposes.
- **[Custom schema adds maintenance burden]** -> Mitigation: Minimal — only 4 new artifacts and modified instructions. Uses standard OpenSpec schema format.
- **[Review staleness detection is imperfect]** -> Mitigation: Partially mechanical (file timestamps) + semantic (materiality judgment). Documented honestly as not perfectly detectable.

## Migration Plan

This is a greenfield methodology — no migration needed. The first user installs by:
1. Copying `openspec/schemas/goulart-sdd/` into their project.
2. Copying Goulart OpenCode skills from `.opencode/skills/goulart-*`.
3. Setting `schema: goulart-sdd` in `openspec/config.yaml`.

## Open Questions

None remaining. All architectural decisions have been resolved per the human review findings.
