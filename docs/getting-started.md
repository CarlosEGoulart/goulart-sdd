# Getting Started with Goulart SDD

A practical guide to using the current strict Goulart SDD workflow in this repository.

## What You Need

**OpenSpec CLI** — the official OpenSpec engine. Install globally:

```
npm install -g @fission-ai/openspec@latest
```

**OpenCode** — the current first reference coding-agent adapter. OpenCode is the first adapter; the methodology and schema are agent-agnostic. Install OpenCode using its official supported installation method.

**This repository** — already configured with the `goulart-sdd` schema, adapter entry points, and methodology documentation.

## Current Repository Setup

This repository contains all required Goulart SDD pieces:

| Path | Purpose |
|---|---|
| `openspec/config.yaml` | Project config selecting `schema: goulart-sdd` |
| `openspec/schemas/goulart-sdd/` | Custom schema, templates, and instructions |
| `.opencode/commands/goulart-*` | Command entry points (thin delegation to skills) |
| `.opencode/skills/goulart-*` | Skills implementing methodology sequencing |
| `docs/methodology.md` | Canonical human-readable methodology reference |

The project config selects `schema: goulart-sdd`. Run examples from the repository root where OpenSpec can resolve the `openspec/` directory.

## First Change Walkthrough

This walkthrough uses the change name `add-example-feature` to show the full planning lifecycle.

### 1. Initial Planning

```
/goulart-plan add-example-feature
```

The initial planning phase creates or continues:

1. proposal
2. specs
3. design

Then it **MUST STOP**.

> **STOP #1 — PLAN REVIEW REQUIRED**
>
> After this stop, test-plan and tasks are NOT yet generated. Implementation must NOT begin. The next step is independent plan review.

### 2. Independent Plan Review

```
/goulart-review plan add-example-feature
```

Run this in a fresh or separate review context when possible. The review evaluates proposal, specs, and design for consistency, compliance, quality, feasibility, scope, and testability.

The reviewer emits one verdict:

| Verdict | Meaning |
|---|---|
| APPROVE | Plan is acceptable as-is. |
| APPROVE_WITH_CHANGES | Required changes must be applied. |
| REVISE | Normal progression is blocked. |

The reviewer verdict alone does NOT resume planning. A human disposition is required.

### 3. Human Disposition

The human must provide a decision after the review:

| Disposition | Effect |
|---|---|
| ACCEPTED | Permits downstream planning, provided the review is current and all required changes are applied. |
| REVISE | Human requests revision (Human Reason mandatory). Artifacts revised and re-reviewed. |
| OVERRIDDEN | Explicit exception with mandatory reason. Does not make a stale review fresh. |

APPROVE + current review + ACCEPTED → downstream planning may proceed.

Do not treat reviewer APPROVE as automatic human acceptance. The human decision is a separate, mandatory step.

### 4. Continue Planning with goulart-plan

```
/goulart-plan add-example-feature
```

**This is the same command.** It is state-aware. When the plan-review gate permits continuation, it creates or continues:

1. test-plan
2. tasks

Then it **MUST STOP**.

> **STOP #2 — PLANNING COMPLETE**
>
> Do NOT implement in the same `goulart-plan` invocation. Planning reports `goulart-apply` as the next Goulart-compliant step.

### 5. Implement One Task

```
/goulart-apply add-example-feature
```

Each invocation handles **one task**:

1. Selects the first eligible pending task
2. Loads relevant context
3. Implements: RED → GREEN → REFACTOR (where AUTOMATED behavioral testing is appropriate)
4. Marks only that task complete
5. Reports the result
6. Stops

Run `/goulart-apply add-example-feature` again for the next task. When all tasks are complete, `goulart-apply` stops and directs you to `/goulart-review code add-example-feature`.

## What Happens After Implementation

After all tasks are complete, the remaining lifecycle stages are:

```
/goulart-review code add-example-feature
```

Independent code review with findings, verdicts, and human triage where findings exist.

```
/goulart-verify add-example-feature
```

Final verification audit producing a DECISION: PASS, PASS_WITH_WARNINGS, or FAIL.

```
/goulart-archive add-example-feature
```

Archive eligibility check consuming the verify result. Delegates to upstream OpenSpec archive when eligible.

Each stage has its own gates and STOP conditions. See [docs/methodology.md](methodology.md) for full review, verification, and archive semantics.

## Walkthrough Diagram

```
/goulart-plan add-example-feature
        |
        v
 proposal → specs → design
        |
        v
   STOP #1 ─── PLAN REVIEW REQUIRED
        |
        v
/goulart-review plan add-example-feature
        |
        v
 HUMAN DISPOSITION (ACCEPTED / REVISE / OVERRIDDEN)
        |
        v
/goulart-plan add-example-feature  (same command, state-aware)
        |
        v
 test-plan → tasks
        |
        v
   STOP #2 ─── PLANNING COMPLETE
        |
        v
/goulart-apply add-example-feature
        |
        v
 ONE TASK → STOP  (repeat per invocation)
        |
        v
 when all tasks complete
        |
        v
/goulart-review code → /goulart-verify → /goulart-archive
```

## Goulart-Compliant vs Raw OpenSpec

**Goulart-compliant execution** uses the `goulart-*` entry points and honors their STOPs, review gates, human decisions, and verification rules.

| Command | Goulart-compliant |
|---|---|
| `/goulart-plan` | Yes |
| `/goulart-review` | Yes |
| `/goulart-apply` | Yes |
| `/goulart-verify` | Yes |
| `/goulart-archive` | Yes |

**Raw OpenSpec commands** remain available as escape hatches:

| Command | Goulart-compliant |
|---|---|
| `/opsx-propose` | No — may bypass Goulart sequencing |
| `/opsx-apply` | No — may bypass Goulart gates |
| `/opsx-archive` | No — may bypass Goulart verification |
| `openspec ...` | No — raw execution |

Raw OpenSpec is not disabled. It remains an escape hatch. However, raw execution that bypasses Goulart gates must NOT be represented as Goulart-compliant execution. For example, raw `/opsx-propose` may generate the entire planning artifact set at once, bypassing the mandatory first STOP and plan-review handoff.

## Common Mistakes

| Mistake | Why it is wrong |
|---|---|
| Running `/goulart-plan` once and expecting implementation to start | The first invocation stops after design. Plan review and human disposition are required before continuing. |
| Skipping plan-review after the first STOP | Plan review is mandatory. Test-plan and tasks are not generated without a permitting review and human gate. |
| Treating reviewer APPROVE as human ACCEPTED | The reviewer verdict and human disposition are separate steps. APPROVE alone does not authorize continuation. |
| Inventing `/goulart-continue` or `/goulart-resume` | There is no such command. The same `goulart-plan` entry point is state-aware and is invoked again after the review/human gate. |
| Expecting the second `goulart-plan` call to implement code | Planning creates test-plan and tasks, then stops. Implementation uses `goulart-apply`. |
| Running one `goulart-apply` and expecting all tasks to finish | Each invocation handles one task. Run it again for the next task. |
| Using raw `/opsx-propose` and calling the result Goulart-compliant | Raw OpenSpec may bypass Goulart sequencing and gates. The result is not Goulart-compliant. |
| Treating a different model alone as independent review | Cross-model review is optional. Different model does not equal independence. Separate context does. |

## Fresh Review Context

Independent review prefers:

1. Harness-supplied fresh or separate context (preferred)
2. User-managed separate clean session (fallback, still independent)
3. Degraded review only when the first two are unavailable

Cross-model review is optional. A different model alone does not establish independence. See [docs/methodology.md](methodology.md) for the full independent review and degraded-mode contract.

## Further Reading

- [docs/methodology.md](methodology.md) — canonical methodology reference
- [docs/getting-started.md](getting-started.md) — this guide
- [openspec/config.yaml](../openspec/config.yaml) — project configuration
- [openspec/schemas/goulart-sdd/](../openspec/schemas/goulart-sdd/) — custom schema
