## Context

Goulart SDD extends OpenSpec's `spec-driven` schema with engineering discipline. OpenSpec provides the artifact graph, dependency ordering, templates, validation, and lifecycle operations. Goulart SDD adds artifacts that OpenSpec does not have (plan-review, test-plan, code-review, verify), modifies instructions for existing artifacts (tasks with TDD ordering), and creates coding-agent skills that enforce the methodology.

The current OpenSpec version (1.13.0) supports custom schemas in `openspec/schemas/<name>/schema.yaml` with templates, instructions, and dependency graphs. OpenSpec's `requires:` field enforces artifact file existence but not semantic quality. All behavioral guarantees (TDD, review independence, human gates) are agent-honored, not mechanically enforced by the CLI.

See proposal.md - Why for motivation.

## Goals / Non-Goals

**Goals:**

- Define a custom OpenSpec schema that maps the Goulart SDD strict workflow onto OpenSpec's artifact model.
- Establish ownership boundaries that survive `openspec update` and upstream changes.
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
proposal → specs → design → plan-review → test-plan → tasks → apply → code-review → verify → archive
```

Where:
- `proposal`, `specs`, `design`, `tasks` are standard OpenSpec artifacts (reused from `spec-driven`).
- `plan-review` is a new artifact (requires: proposal, specs, design).
- `test-plan` is a new artifact (requires: specs, plan-review).
- `tasks` depends on test-plan (modified from spec-driven which requires specs + design).
- `code-review` is a new artifact (requires: tasks — produced after apply completes).
- `verify` is a new artifact (requires: tasks — produced after code-review).
- `apply` is the standard OpenSpec operation (requires: tasks).
- `archive` is the standard OpenSpec operation (requires: verify via execution gate).

**Rationale:** This maps the full Goulart lifecycle onto OpenSpec's dependency graph while preserving standard OpenSpec behavior for the artifacts it already owns.

**Alternatives considered:**
- Put code-review and verify as "operations" rather than artifacts: rejected because they produce output files that need to persist for audit evidence.
- Make plan-review and code-review the same artifact: rejected because they serve different lifecycle stages with different inputs and different independence requirements.
- Make verify optional: rejected because it is the cheap insurance against "each task passed but the whole doesn't work."

### D3: Review as single artifact, not two reviewers

**Decision:** One review artifact per review stage (plan-review, code-review), with findings tagged by category (compliance, quality, feasibility, scope, risk).

**Rationale:** For personal and small academic projects, two separate review artifacts create disproportionate ceremony. One reviewer checking both spec compliance and code quality in a single pass is sufficient. The reviewer tags findings by category so the human can triage.

**Alternatives considered:**
- Separate compliance-reviewer and code-reviewer personas: rejected as enterprise ceremony for the target audience. Could be added later if projects grow.

### D4: Reviewer role with fresh context, not cross-model mandate

**Decision:** The reviewer SHOULD use fresh context where supported. Cross-model review MAY be recommended but is NOT mandatory. Fresh-context independence is more important than model diversity.

**Rationale:** The methodology must work with local/free models where only one model is available. Fresh context already provides significant independence (no authoring history). Cross-model is a nice-to-have, not a requirement.

**Alternatives considered:**
- Mandatory cross-model review: rejected because it would exclude users with only local models.

### D5: Three gate categories

**Decision:** Explicitly distinguish:
1. **Artifact gates** (OpenSpec `requires:`) — enforce file existence/order.
2. **Execution gates** (Goulart skills/commands) — enforce pre-implementation state checks.
3. **Repository gates** (scripts/CI) — validate repository state structurally.

**Rationale:** This honest classification prevents pretending that agent instructions are mechanical enforcement, and prevents pretending that CI can prove agent cognition.

### D6: TDD as process guarantee, not mechanical proof

**Decision:** TDD (RED-GREEN-REFACTOR) is a methodology requirement enforced by agent workflow instructions and human oversight. The repository can verify that tests exist and pass, but cannot prove the temporal sequence.

**Rationale:** Building heavyweight evidence systems (signed logs, forced micro-commits) for TDD chronology is disproportionate ceremony for solo developers. The test-plan tracks red→green state, which provides reasonable visibility.

### D7: Verify as mandatory pre-archive gate

**Decision:** The verify artifact MUST exist with DECISION: PASS or DECISION: PASS_WITH_WARNINGS before archive proceeds. This is enforced as an execution gate in the apply/verify workflow.

**Rationale:** Verify is the cheap insurance that catches the "each task passed but the whole doesn't work" problem. It is lightweight (checklist comparison) and mandatory.

### D8: OpenCode-first adapter with agent-agnostic core

**Decision:** OpenCode is the first reference adapter. The schema, methodology docs, and config are agent-agnostic. Skills are OpenCode-specific but their content can be adapted.

**Rationale:** Starting with one adapter keeps Issue #1 focused. The UTF-SDD reference demonstrates that one methodology can map to 4+ tools. Agent-agnostic core means future adapters don't require methodology redesign.

### D9: Ownership layout

**Decision:** Separate namespaces:
- `openspec/schemas/goulart-sdd/` — schema owned by Goulart SDD.
- `.opencode/commands/goulart-*` — commands owned by Goulart SDD.
- `.opencode/skills/goulart-*` — skills owned by Goulart SDD.
- `scripts/gates/` — CI scripts owned by Goulart SDD.
- `docs/` — methodology documentation.
- `.opencode/commands/opsx-*` and `.opencode/skills/openspec-*` — OpenSpec-owned, never modified.

**Rationale:** `openspec update` regenerates OpenSpec-owned files. Goulart-owned files use a different prefix and are never touched by upstream operations.

### D10: Task granularity for limited context

**Decision:** Tasks SHALL be small enough for one coding-agent session. Each task maps to one acceptance criterion or one test-plan entry. Tasks that use "and" twice should be split.

**Rationale:** Coding agents with limited context windows perform better on focused tasks. This also enables fresh-context-per-task without losing critical information.

## Risks / Trade-offs

- **[Prompt-only TDD enforcement]** → Mitigation: Test-plan tracks red/green state; verify checks final suite; human reviews task commits. Cannot prove chronology from repo state alone.
- **[Reviewer independence is instructed, not enforced]** → Mitigation: Fresh-context instruction in skills; adversarial posture in review template; read-only posture. Human remains the final gate.
- **[Schema maintenance as OpenSpec evolves]** → Mitigation: Fork from `spec-driven`; use only standard schema fields; `openspec schema fork` makes rebasing easy.
- **[OpenCode lock-in for skills]** → Mitigation: Schema and methodology docs are agent-agnostic; skills are the first adapter, not the only one. Same content can be adapted for Cursor, Claude Code, etc.
- **[Excessive ceremony for very small changes]** → Mitigation: `skip_specs: true` exists for spec-less changes. Design instruction already says "create only if applicable." Future: a light profile.
- **[Verify feels redundant with archive validation]** → Mitigation: Verify is a pre-archive audit that checks spec compliance + test integrity + review completeness. Archive only checks delta spec formatting. Different purposes.
- **[Custom schema adds maintenance burden]** → Mitigation: Minimal — only 4 new artifacts and modified instructions. Uses standard OpenSpec schema format.

## Migration Plan

This is a greenfield methodology — no migration needed. The first user installs by:
1. Copying `openspec/schemas/goulart-sdd/` into their project.
2. Copying OpenCode skills from `.opencode/skills/goulart-*`.
3. Setting `schema: goulart-sdd` in `openspec/config.yaml`.

## Open Questions

- Should `code-review` and `verify` produce separate files, or could verify be a section within code-review? (Current design: separate files for clearer audit trail.)
- Should the test-plan track individual test status (red/green) during apply, or only check final suite status at verify? (Current design: track during apply for visibility.)
