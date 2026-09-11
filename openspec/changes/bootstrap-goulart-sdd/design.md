## Context

Goulart SDD extends OpenSpec's `spec-driven` schema with engineering discipline. OpenSpec provides the artifact graph, dependency ordering, templates, validation, and lifecycle operations. Goulart SDD adds artifacts that OpenSpec does not have (plan-review, test-plan, code-review, verify), modifies instructions for existing artifacts (tasks with TDD ordering), and creates Goulart-owned adapter entry points that enforce methodology sequencing and execution gates.

The current OpenSpec version (1.13.0) supports custom schemas in `openspec/schemas/<name>/schema.yaml` with templates, instructions, and dependency graphs. OpenSpec's `requires:` field enforces artifact file existence but not semantic quality. All behavioral guarantees (TDD, review independence, human gates) are agent-honored, not mechanically enforced by the CLI.

Goulart SDD extends OpenSpec; it does not disable or replace upstream OpenSpec commands. Raw commands (`/opsx-propose`, `/opsx-apply`, `/opsx-archive`, `openspec ...`) remain available as escape hatches but MAY bypass Goulart execution gates and SHALL NOT be represented as satisfying all Goulart SDD process guarantees.

See proposal.md - Why for motivation.

## Goals / Non-Goals

**Goals:**

- Define a custom OpenSpec schema that maps the Goulart SDD strict workflow onto OpenSpec's artifact model.
- Establish ownership boundaries that survive `openspec update` and upstream changes.
- Create Goulart-owned adapter entry points that enforce methodology sequencing.
- Create OpenCode skills as the first reference coding-agent adapter.
- Produce methodology documentation that is agent-agnostic.
- Dogfood the methodology by using OpenSpec to design itself.
- Distinguish Goulart-compliant execution from raw OpenSpec execution.

**Non-Goals:**

- Modify the OpenSpec CLI or vendor its source code.
- Create a light/workflow profile (deferred).
- Create adapters for Claude Code, Cursor, GitHub Copilot, or Codex (deferred).
- Build CI enforcement scripts (deferred beyond basic structural checks).
- Prove TDD chronology from repository state.
- Create per-task code review (only per-change review in v0.1).
- Mandate specific LLM providers or models.
- Enterprise governance or elaborate audit/evidence infrastructure.

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
       instruct user: "run goulart-review plan in a fresh session"
        |
        v
   REVIEWER (fresh context or degraded mode with disclosure)
        plan-review (OpenSpec artifact)
        ROUND: 1 | 2
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
         one task per invocation
         select → load context → RED → GREEN → REFACTOR → mark complete → STOP
              |
         (repeat per invocation)
              |
         all tasks complete
              |
              v
         STOP -- goulart-apply enforces this stop
         instruct user: "run goulart-review code in a fresh session"
              |
              v
         REVIEWER (fresh context or degraded mode with disclosure)
         code-review (OpenSpec artifact)
         ROUND: 1 | 2
         findings + VERDICT
              |
              v
         HUMAN TRIAGE (where findings exist; not required when no findings)
         ACCEPT | REJECT | DEFER per finding
              |
              v
           goulart-verify (adapter command)
         check prerequisites → code-review valid, triage complete
         perform verification
         produce verify artifact
         DECISION: PASS | PASS_WITH_WARNINGS | FAIL
              |
              v
         goulart-archive (adapter command)
         check verify decision
         PASS → delegate to OpenSpec archive
         PASS_WITH_WARNINGS → delegate after human warning dispositions recorded
         FAIL → block archive (human override must be explicit with reason)
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

## Goulart Compliance Boundary

Goulart-compliant execution uses `goulart-*` lifecycle entry points and satisfies all Goulart SDD process guarantees. Raw OpenSpec execution remains available as an escape hatch but MAY bypass Goulart execution gates.

This is documented honestly in the methodology. We do not attempt to modify or disable upstream OpenSpec commands.

## Gate Categories

### Artifact gates

Provided by OpenSpec's schema/dependency graph (`requires:` field).

- Enforce artifact file existence and ordering.
- Do NOT prove artifact semantic quality.
- Do NOT prove implementation completeness.

### Execution gates

Provided by Goulart adapter commands and skills.

- `goulart-plan`: stops after design, instructs user to run `goulart-review plan` in fresh session.
- `goulart-apply`: one task per invocation, checks pre-implementation gates (plan-review verdict, human acceptance, test-plan existence), executes TDD, instructs user to run `goulart-review code` when all tasks complete.
- `goulart-verify`: checks prerequisites (code-review exists, triage complete), performs verification, produces verify artifact with DECISION.
- `goulart-archive`: checks verify decision, delegates to OpenSpec archive (PASS), requires human warning dispositions (PASS_WITH_WARNINGS), blocks on FAIL (human override explicit with reason).
- These are not necessarily provable from Git history.

### Repository gates

Provided by scripts, tests, CI.

- OpenSpec validation, automated tests, lint, schema validity, structural artifact checks.
- Validate repository state but generally cannot prove agent cognition or process chronology.

### Human decisions

Explicitly NOT a mechanical gate. The human retains final authority to accept, reject, defer, or override at any point.

- **plan-review**: explicit human acceptance (ACCEPTED/REVISE/OVERRIDDEN) is mandatory.
- **code-review**: human triage is mandatory only when findings exist.
- **verify**: human warning dispositions required for PASS_WITH_WARNINGS; explicit override required for FAIL.

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

Key distinction: `requires:` proves artifact existence/order but does NOT prove all implementation tasks are complete before code-review. The `goulart-apply` adapter checks task completion before instructing the user to run code-review.

### D3: Goulart adapter entry points

**Decision:** Create Goulart-owned adapter entry points that enforce methodology sequencing:

- `goulart-plan`: orchestrates proposal/specs/design, then stops and instructs user to run `goulart-review plan` in a fresh session.
- `goulart-review`: invokes fresh-context review for both plan-review and code-review stages. Supports degraded mode with explicit disclosure.
- `goulart-apply`: one task per invocation. Checks pre-implementation gates, executes TDD, marks task complete, stops. Instructs user to run `goulart-review code` when all tasks complete.
- `goulart-verify`: checks prerequisites (code-review exists, triage complete), performs verification, produces verify artifact with DECISION.
- `goulart-archive`: checks verify decision. PASS → delegate to OpenSpec archive. PASS_WITH_WARNINGS → require human warning dispositions. FAIL → block archive.

**Rationale:** OpenSpec owns artifact creation mechanics; Goulart adapters own methodology sequencing. This is an execution/workflow guarantee, not a new OpenSpec engine feature.

**Alternatives considered:**
- Rely only on agent instructions to follow the sequence: rejected because instructions alone are too easily bypassed.
- Create a new OpenSpec engine feature: rejected because it requires forking/modifying OpenSpec.

### D4: Split review templates

**Decision:** Two separate review templates: `templates/plan-review.md` and `templates/code-review.md`, reflecting structurally different artifacts. One Reviewer role, two artifact contracts.

- `plan-review.md`: Review Metadata, Round, Reviewed Inputs, Findings, Verdict, Required Changes, Human Decision, Human Reason.
- `code-review.md`: Review Metadata, Round, Reviewed Implementation, Findings, Verdict, Per-Finding Human Triage, Required Changes.

**Rationale:** A single generic template is insufficient for two structurally different review stages with different human interaction patterns.

### D5: Human decision after plan-review

**Decision:** After plan-review, the human records a disposition: ACCEPTED, REVISE, or OVERRIDDEN. Reviewer approval alone is NOT sufficient to proceed. APPROVE_WITH_CHANGES requires either re-review after material changes or explicit OVERRIDDEN with reason.

**Rationale:** This ensures the human remains the final gate. An override MUST include a reason for traceability.

### D6: Three validation types in test-plan

**Decision:** Each test-plan entry declares one of: AUTOMATED, MECHANICAL, or SEMANTIC.

- AUTOMATED: unit/integration/E2E tests with file path and function name.
- MECHANICAL: tool-based checks (lint, typecheck, schema validation) with command.
- SEMANTIC: when automation is unsuitable, with justification and evaluator identification.

**Rationale:** The previous design forced every scenario into an automated test, which is too rigid for methodology-level and semantic guarantees (e.g., reviewer independence, architectural judgment).

### D7: Verify depends on code-review

**Decision:** `verify` requires `code-review` in the artifact graph. The `goulart-verify` adapter additionally checks that code-review exists and triage is complete before proceeding.

**Rationale:** This prevents verification from proceeding without an independent code review. The artifact graph enforces ordering; the adapter enforces the semantic gate.

### D8: Behavioral test integrity

**Decision:** Tests SHALL NOT be removed, weakened, skipped, or replaced in a way that reduces required behavioral coverage without an explicitly justified specification change. Verification focuses on retained behavior/scenario coverage, not file identity.

**Rationale:** The previous rule ("no test removal without REMOVED requirement") was too rigid. Tests may legitimately be renamed, consolidated, replaced, rewritten, or moved as long as behavioral coverage is preserved.

### D9: Review staleness — two lineages, revision-based detection

**Decision:** Verify distinguishes two review staleness lineages:

- **Plan-review staleness**: material changes to proposal/specs/design after the plan-review verdict require a new plan-review.
- **Code-review staleness**: material changes to source code/tests/config after the code-review verdict require a new code-review.

Staleness detection uses repository revision/diff information where available, falling back to conservative semantic comparison. Filesystem timestamps SHALL NOT be used as reliable staleness evidence (unstable across clone, checkout, rebase, CI, file copy).

Each review artifact SHOULD record the reviewed revision/commit and reviewed paths/artifacts.

**Rationale:** The two reviews cover different artifacts at different lifecycle stages. Revision-based detection is more reliable than timestamps. Timestamps are explicitly excluded due to instability across common Git operations.

### D10: Verify decision semantics

**Decision:** PASS permits archive. PASS_WITH_WARNINGS permits archive only after human explicitly accepts/defers warnings — it SHALL NOT silently mean PASS. FAIL blocks archive unless the human explicitly overrides with reason.

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

**Decision:** Namespace isolation for collision-prone integration identifiers:
- `openspec/schemas/goulart-sdd/` — schema owned by Goulart SDD.
- `.opencode/commands/goulart-*` — commands owned by Goulart SDD.
- `.opencode/skills/goulart-*` — skills owned by Goulart SDD.
- `scripts/gates/` — CI scripts owned by Goulart SDD.
- `docs/` — methodology documentation (standard naming, no `goulart-` prefix required).
- `.opencode/commands/opsx-*` and `.opencode/skills/openspec-*` — OpenSpec-owned, never modified.

### D15: Task granularity for limited context

**Decision:** Tasks SHALL be small enough for one coding-agent session. Each task maps to one acceptance criterion or one test-plan entry.

### D16: One-task-per-invocation apply baseline

**Decision:** The portable/default behavioral contract for `goulart-apply` is one task per invocation. The adapter selects the first eligible pending task, loads relevant context, executes TDD, marks the task complete, reports the result, and STOPs. A subsequent invocation processes the next task. The adapter SHALL NOT run the entire task list in one accumulated conversational context by default.

If a harness supports true isolated subcontexts, an adapter MAY optimize by spawning one fresh subcontext per task, but this is an optimization, not the baseline.

**Rationale:** One task per invocation ensures maximum portability and context isolation. Users can start each task in a fresh session even when the harness cannot spawn isolated subagents.

### D17: Fresh-context fallback hierarchy

**Decision:** Fresh-context independence is prioritized over cross-model review. The fallback hierarchy is:

1. Harness-created fresh context (preferred)
2. User-managed fresh session (fallback)
3. Degraded review mode (last resort)

In degraded mode: review MAY proceed, limitation MUST be disclosed, human MUST acknowledge degraded independence, result MUST NOT be described as independent review.

**Rationale:** A SHALL requirement that contradicts graceful degradation is incoherent. The fallback hierarchy provides clear, honest escalation.

### D18: Review-round persistence

**Decision:** Each review artifact records `ROUND: 1 | 2` and a concise previous-round disposition/history. This prevents overwriting review evidence while remaining lightweight.

**Rationale:** Minimal approach avoids an elaborate review-history subsystem while preserving enough information to know the current round and previous outcome.

## Risks / Trade-offs

- **[Prompt-only TDD enforcement]** -> Mitigation: Test-plan tracks status for visibility; verify checks final suite; human reviews task commits. Cannot prove chronology from repo state alone.
- **[Reviewer independence is instructed, not enforced]** -> Mitigation: Fresh-context fallback hierarchy with degraded-mode disclosure; adversarial posture in review template; read-only posture. Human remains the final gate.
- **[Schema maintenance as OpenSpec evolves]** -> Mitigation: Fork from `spec-driven`; use only standard schema fields; `openspec schema fork` makes rebasing easy.
- **[OpenCode lock-in for skills]** -> Mitigation: Schema and methodology docs are agent-agnostic; skills are the first adapter, not the only one.
- **[Excessive ceremony for very small changes]** -> Mitigation: `skip_specs: true` exists for spec-less changes. Future: a light profile.
- **[Verify feels redundant with archive validation]** -> Mitigation: Verify is a pre-archive audit checking spec compliance + test integrity + review completeness. Archive only checks delta spec formatting. Different purposes.
- **[Custom schema adds maintenance burden]** -> Mitigation: Minimal — only 4 new artifacts and modified instructions. Uses standard OpenSpec schema format.
- **[Review staleness detection is imperfect]** -> Mitigation: Revision-based detection where available; conservative semantic fallback. Timestamps explicitly excluded. Documented honestly.
- **[One-task-per-invocation adds ceremony]** -> Mitigation: Portable baseline ensures context isolation even without harness support. Adapters MAY optimize with subcontexts when available.

## Migration Plan

This is a greenfield methodology — no migration needed. The first user installs by:
1. Copying `openspec/schemas/goulart-sdd/` into their project.
2. Copying Goulart OpenCode skills from `.opencode/skills/goulart-*`.
3. Setting `schema: goulart-sdd` in `openspec/config.yaml`.

## Open Questions

None remaining. All architectural decisions have been resolved per the human review findings.
