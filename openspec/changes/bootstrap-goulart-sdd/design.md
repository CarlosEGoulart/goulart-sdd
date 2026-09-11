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
    gate permits continuation?
     /     \
   no       yes
    |        |
  revise     v
          later goulart-plan invocation
              |
          test-plan (OpenSpec artifact)
              |
             tasks (OpenSpec artifact)
              |
          report planning complete → STOP
          next Goulart-compliant step: goulart-apply
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
         required material fixes → stale review → re-review
         round limit reached → STOP / human escalation
              |
              v
           goulart-verify (adapter command)
         check prerequisites → verdict permits, triage complete
         all tasks complete, test-plan entries evaluable
         review freshness assessed, permitted overrides recorded
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

Goulart-compliant execution uses `goulart-*` lifecycle entry points and honors their gates and human dispositions. Degraded reviews and overrides must retain their disclosed limitations; they do not supply the guarantees they waive. Raw OpenSpec execution remains available as an escape hatch but MAY bypass Goulart execution gates. In OpenSpec 1.13, raw `/opsx-propose` creates a change and may generate its entire schema-required planning set, bypassing Goulart sequencing, review handoff, or human gates. That result SHALL NOT be represented as Goulart-compliant planning.

This is documented honestly in the methodology. We do not attempt to modify or disable upstream OpenSpec commands.

## Gate Categories

### Artifact gates

Provided by OpenSpec's schema/dependency graph (`requires:` field).

- Enforce artifact file existence and ordering.
- Do NOT prove artifact semantic quality.
- Do NOT prove implementation completeness.

### Execution gates

Provided by Goulart adapter commands and skills.

- `goulart-plan`: state-aware initial planning and later continuation through test-plan/tasks, with the plan-review and human-disposition gate between them (D3).
- `goulart-apply`: one task per invocation, checks pre-implementation gates (plan-review verdict, human acceptance, test-plan existence), executes TDD, instructs user to run `goulart-review code` when all tasks complete.
- `goulart-verify`: independently checks implementation-completeness and review prerequisites (D7), performs verification, produces verify artifact with DECISION.
- `goulart-archive`: checks verify decision, delegates to OpenSpec archive (PASS), requires human warning dispositions (PASS_WITH_WARNINGS), blocks on FAIL (human override explicit with reason).
- These are not necessarily provable from Git history.

### Repository gates

Provided by scripts, tests, CI.

- OpenSpec validation, automated tests, lint, schema validity, structural artifact checks.
- Validate repository state but generally cannot prove agent cognition or process chronology.

### Human decisions

Explicitly NOT a mechanical gate. The human retains final authority to accept, reject, defer, or override at any point.

- **plan-review**: explicit human disposition (ACCEPTED/REVISE/OVERRIDDEN) is mandatory; only the verdict/disposition combinations in D5 permit progression.
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

- `goulart-plan`: while initial planning is incomplete, creates/continues proposal/specs/design, then STOPs and instructs the user to run `goulart-review plan` in a fresh session. On a later invocation, checks the review verdict, freshness, human disposition, and any permitted override. If the gate permits continuation, creates/continues test-plan then tasks, reports planning complete, STOPs, and identifies `goulart-apply` as the next Goulart-compliant step. Otherwise it STOPs and identifies each unresolved gate. Already-complete artifacts are retained. This uses the same command; no additional lifecycle entry point is needed.
- `goulart-review`: supports plan/code stages using the separate-context hierarchy in D17, with explicitly disclosed and human-acknowledged degraded mode only as a last resort. Author/implementer adapters stop for review handoff; automation may supply a separate/fresh review context but cannot prove cognitive isolation.
- `goulart-apply`: one task per invocation. Checks pre-implementation gates, executes TDD, marks task complete, stops. Instructs user to run `goulart-review code` when all tasks complete.
- `goulart-verify`: independently checks all prerequisites in D7, performs verification, produces verify artifact with DECISION. It consumes code-review state and produces the verify decision; `goulart-archive` consumes that decision.
- `goulart-archive`: checks verify decision. PASS → delegate to OpenSpec archive. PASS_WITH_WARNINGS → require human warning dispositions. FAIL → block archive.

**Rationale:** OpenSpec owns artifact creation mechanics; Goulart adapters own methodology sequencing. This is an execution/workflow guarantee, not a new OpenSpec engine feature.

**Alternatives considered:**
- Rely only on agent instructions to follow the sequence: rejected because instructions alone are too easily bypassed.
- Create a new OpenSpec engine feature: rejected because it requires forking/modifying OpenSpec.

### D4: Split review templates

**Decision:** Two separate review templates: `templates/plan-review.md` and `templates/code-review.md`, reflecting structurally different artifacts. One Reviewer role, two artifact contracts.

- `plan-review.md`: Review Metadata, Round and previous outcome, Reviewed Inputs (revision/paths where available), Findings, Verdict, Required Changes with materiality classification/rationale, Human Decision, Human Reason including any explicit override.
- `code-review.md`: Review Metadata, Round and previous outcome, Reviewed Implementation (revision/paths where available), Findings, Verdict, Per-Finding Human Triage, Required Changes with materiality justification and any explicit override/reason. Both contracts record degraded-review disclosure/acknowledgement where applicable.

**Rationale:** A single generic template is insufficient for two structurally different review stages with different human interaction patterns.

### D5: Human decision after plan-review

**Decision:** After plan-review, the human records a disposition: ACCEPTED, REVISE, or OVERRIDDEN. Reviewer approval alone is NOT sufficient to proceed.

| Verdict/state | Permitted continuation |
| --- | --- |
| APPROVE | Human STATUS: ACCEPTED permits downstream planning, provided the review is current. |
| APPROVE_WITH_CHANGES, unapplied Required Changes | Block; ACCEPTED cannot waive unapplied changes. |
| APPROVE_WITH_CHANGES, material corrections applied | Re-review required for normal progression, or explicit STATUS: OVERRIDDEN with mandatory reason identifying the waived re-review. |
| APPROVE_WITH_CHANGES, demonstrably non-material corrections applied | Human MAY record ACCEPTED without re-review only with materiality rationale in the review artifact. |
| REVISE or human STATUS: REVISE | Normally revise and re-review; an explicit STATUS: OVERRIDDEN with mandatory reason may permit progression. |

Non-material corrections do not change requirements, scope, architecture, acceptance criteria, or implementation assumptions. An override records an exception; it does not make a stale review fresh. Required re-reviews count toward the two-round limit in D18.

**Rationale:** This ensures the human remains the final gate. An override MUST include a reason for traceability.

### D6: Three validation types in test-plan

**Decision:** Each test-plan entry declares one of: AUTOMATED, MECHANICAL, or SEMANTIC.

- AUTOMATED: unit/integration/E2E tests with file path and function name.
- MECHANICAL: tool-based checks (lint, typecheck, schema validation) with command.
- SEMANTIC: when automation is unsuitable, with justification and evaluator identification.

**Rationale:** The previous design forced every scenario into an automated test, which is too rigid for methodology-level and semantic guarantees (e.g., reviewer independence, architectural judgment).

### D7: Verify depends on code-review

**Decision:** `verify` requires `code-review` in the artifact graph. Before producing verify.md, `goulart-verify` independently checks:

- code-review exists and its verdict/dispositions permit progression;
- mandatory human triage is complete;
- all tasks in tasks.md are complete;
- every required test-plan entry has a final evaluable state (completed AUTOMATED/MECHANICAL check and recorded result, or documented SEMANTIC evaluation);
- review freshness, with any explicitly permitted override recorded rather than presented as fresh review;
- degraded-review disclosure and human acknowledgement where applicable.

Missing review, incomplete tasks, incomplete mandatory triage, or unevaluable entries block verification and identify the gap. These are not waived by a review override. Completed but failing validations remain evaluable and result in FAIL during the audit. The audit then checks compliance, test integrity, and review completeness before emitting DECISION.

**Rationale:** This is defense in depth: artifact existence/order does not prove semantic implementation completion. A qualifying review may be independent or explicitly degraded under D17; verification preserves that distinction.

### D8: Behavioral test integrity

**Decision:** Tests SHALL NOT be removed, weakened, skipped, or replaced in a way that reduces required behavioral coverage without an explicitly justified specification change. Verification focuses on retained behavior/scenario coverage, not file identity.

**Rationale:** The previous rule ("no test removal without REMOVED requirement") was too rigid. Tests may legitimately be renamed, consolidated, replaced, rewritten, or moved as long as behavioral coverage is preserved.

### D9: Review staleness — two lineages, revision-based detection

**Decision:** Verify distinguishes two review staleness lineages:

- **Plan-review staleness**: any material change to proposal/specs/design after review makes it stale, including reviewer-requested Required Changes. Normal progression requires re-review; the explicit plan override in D5 remains an exception, not evidence of freshness. Only non-material corrections preserve freshness without another review, with the rationale recorded in the review artifact and checked during verification.
- **Code-review staleness**: material changes to source code/tests/implementation-relevant config after review make it stale, including changes addressing accepted findings. Re-review is required before normal progression to verify; non-material corrections may avoid re-review only with a clear recorded justification. The explicit override in D19 records a waived review condition without claiming review of the changed implementation.

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

**Decision:** Independent review SHALL use a context/session separate from the author/implementer context. Fresh-context independence is prioritized over optional cross-model review. The fallback hierarchy is:

1. Separate/fresh context supplied by the harness (preferred)
2. User-managed separate clean session (fallback; still independent)
3. Degraded review mode (last resort)

Only when neither separate-context option is available MAY review proceed without a separate context in degraded mode. The limitation MUST be disclosed, the human MUST acknowledge it, and the result MUST NOT be described as independent review. Separate context is a process practice, not mechanical proof of cognitive isolation.

**Rationale:** A SHALL requirement that contradicts graceful degradation is incoherent. The fallback hierarchy provides clear, honest escalation.

### D18: Review-round persistence

**Decision:** Each review artifact records `ROUND: 1 | 2` and a concise previous-round disposition/history. This prevents overwriting review evidence while remaining lightweight.

Both review stages have a maximum of two review/revision rounds, including re-review after APPROVE_WITH_CHANGES and other material post-review changes. Round 2 with unresolved blocking/required changes or a material change requiring further review SHALL STOP and escalate to the human. No automatic third round or history reset is allowed. Explicit human override, where permitted by the relevant verdict contract, must identify the waived condition and reason.

**Rationale:** Minimal approach avoids an elaborate review-history subsystem while preserving enough information to know the current round and previous outcome.

### D19: Code-review verdict transitions

**Decision:** APPROVE means no blocking implementation changes are required. Any findings still require human triage; a clean review needs no additional approval. After triage, verify may proceed if the review remains current and implementation-completeness prerequisites pass.

APPROVE_WITH_CHANGES identifies at least one required implementation change, subject to explicit human triage. Accepted/required changes must be addressed. Material fixes make the review stale and require another round before normal progression; clearly justified non-material corrections may avoid re-review. REJECT or DEFER requires justification and does not permit normal progression while an unresolved blocking/Critical condition remains.

REVISE blocks normal progression: revise and re-review. A human may explicitly override only with a recorded reason identifying the waived review condition. Triage alone is not an override of an unresolved blocking condition, and overrides do not substitute for task completion or test-plan readiness. D18's round limit applies to every required re-review.

**Rationale:** Explicit transitions connect finding disposition, implementation changes, review freshness, and verification without adding an unconditional second approval gate.

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
