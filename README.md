# Goulart SDD

An opinionated Spec-Driven Development workflow for AI coding agents, with explicit planning gates, independent review, human decisions, TDD, focused implementation tasks, and final verification.

> **Project status**
>
> v0.1 methodology bootstrap is complete.
> v0.2 is now focused on making Goulart SDD installable, agent-aware, and less coupled to its current OpenSpec backend.
>
> The planned `npx goulart-sdd init` installer is **not released yet**.

## Why Goulart SDD?

AI coding agents can implement software quickly, but speed alone does not guarantee:

- stable requirements before implementation;
- independent review of plans and code;
- explicit human approval at important decision points;
- test-first implementation where automation is appropriate;
- small, reviewable implementation units;
- reliable final verification before completion.

Goulart SDD turns those practices into an explicit development lifecycle.

## Core workflow

```text
/goulart-plan
      |
      v
 proposal -> specs -> design
      |
      v
 STOP #1: plan review required
      |
      v
/goulart-review plan
      |
      v
 human disposition
 ACCEPTED | REVISE | OVERRIDDEN
      |
      v
/goulart-plan
      |
      v
 test-plan -> tasks
      |
      v
 STOP #2: planning complete
      |
      v
/goulart-apply
      |
      v
 one focused task
 RED -> GREEN -> REFACTOR
      |
      v
 STOP and repeat per task
      |
      v
/goulart-review code
      |
      v
/goulart-verify
      |
      v
/goulart-archive
```

The lifecycle is intentionally not a single uninterrupted agent session. Goulart SDD uses STOP boundaries to separate planning, review, implementation, and verification responsibilities.

## Command reference

| Command | Purpose |
|---|---|
| `/goulart-plan <change>` | Creates or continues planning. Initial planning stops after proposal/specs/design; later planning creates test-plan/tasks after the review and human gate permit progression. |
| `/goulart-review plan <change>` | Performs plan review in a fresh or separate review context when possible. |
| `/goulart-apply <change>` | Implements one eligible focused task per invocation and then stops. |
| `/goulart-review code <change>` | Reviews the completed implementation against the approved plan and test evidence. |
| `/goulart-verify <change>` | Performs final prerequisite checks and verification audit. |
| `/goulart-archive <change>` | Checks archive eligibility from the completed verification result and delegates archive mechanics when permitted. |

## Core principles

Goulart SDD is built around these rules:

- **Spec before implementation** — implementation follows explicit planning artifacts.
- **Human-in-the-loop** — reviewer verdicts do not silently become human decisions.
- **Independent review** — review prefers a fresh or separate context from author/implementer work.
- **One focused task per apply invocation** — implementation context is intentionally bounded.
- **TDD where appropriate** — automated behavioral work follows RED -> GREEN -> REFACTOR.
- **No fake tests** — mechanical and semantic validation remain mechanical/semantic when that is the correct validation type.
- **STOP on spec drift** — implementation must not silently rewrite requirements to make code pass.
- **Bounded review loops** — repeated review/revision does not continue indefinitely without escalation.
- **Evidence over claims** — test status, review freshness, human decisions, and verification results must not be fabricated.
- **Verify before archive** — archive consumes verification; it does not replace verification.

## Current usage: v0.1 reference implementation

v0.1 is currently a repository-hosted reference implementation. It is not yet distributed as a standalone installer.

### Prerequisites

Install the official OpenSpec CLI:

```bash
npm install -g @fission-ai/openspec@latest
```

Install OpenCode using its supported installation method.

Then clone this repository and work from its root:

```bash
git clone https://github.com/CarlosEGoulart/goulart-sdd.git
cd goulart-sdd
```

The repository already contains:

```text
openspec/
  config.yaml
  schemas/goulart-sdd/

.opencode/
  commands/goulart-*
  skills/goulart-*

docs/
  methodology.md
  getting-started.md
```

### Start a change

Inside OpenCode:

```text
/goulart-plan add-example-feature
```

The first invocation creates or continues:

```text
proposal
specs
design
```

and then **must stop**.

Next, run the plan review from a fresh or separate review context:

```text
/goulart-review plan add-example-feature
```

After the human records a permitting disposition, run the same planning command again:

```text
/goulart-plan add-example-feature
```

The later invocation creates or continues:

```text
test-plan
tasks
```

and then stops before implementation.

Implementation proceeds one task at a time:

```text
/goulart-apply add-example-feature
```

When all tasks are complete:

```text
/goulart-review code add-example-feature
/goulart-verify add-example-feature
/goulart-archive add-example-feature
```

For the full lifecycle semantics, see [docs/methodology.md](docs/methodology.md).

For the current detailed walkthrough, see [docs/getting-started.md](docs/getting-started.md).

## Supported coding agents

| Coding agent | Status |
|---|---|
| OpenCode | Reference adapter available |
| Claude Code | Planned |
| Codex | Planned |
| Cursor | Planned |
| Generic adapter | Planned |

OpenCode is the first reference adapter. The methodology is intended to remain agent-agnostic.

## Goulart-compliant vs raw OpenSpec

Normal Goulart usage should use the `goulart-*` entry points.

Raw OpenSpec commands remain available because OpenSpec is currently the underlying workflow engine, but raw execution may bypass Goulart sequencing and gates.

| Entry point | Goulart-compliant? |
|---|---|
| `/goulart-plan` | Yes |
| `/goulart-review` | Yes |
| `/goulart-apply` | Yes |
| `/goulart-verify` | Yes |
| `/goulart-archive` | Yes |
| `/opsx-propose` | No — raw OpenSpec path |
| `/opsx-apply` | No — may bypass Goulart apply gates |
| `/opsx-archive` | No — may bypass Goulart verification/archive gates |
| `openspec ...` | Raw engine operation |

Raw OpenSpec is an implementation escape hatch, not the normal Goulart workflow.

## Architecture today

```text
OpenSpec engine
      |
      v
Goulart methodology
schema + templates + policies + lifecycle rules
      |
      v
OpenCode adapter
goulart-* commands and skills
```

OpenSpec currently provides:

- change lifecycle storage;
- schema resolution and artifact dependency graph;
- artifact instructions and templates;
- validation;
- status/context operations;
- archive mechanics.

Goulart SDD adds the opinionated engineering workflow on top:

- mandatory planning STOPs;
- independent/degraded review semantics;
- explicit human dispositions;
- test-plan coverage rules;
- focused one-task-per-invocation implementation;
- TDD policy;
- spec-drift STOP behavior;
- final verification and archive eligibility gates.

## v0.2 direction

The next release is intended to turn Goulart SDD into an installable developer tool.

Target user experience:

```bash
npx goulart-sdd init
```

Planned initialization flow:

```text
? Select your coding agent
> OpenCode
  Claude Code
  Codex
  Cursor
  Generic
```

The intended v0.2 architecture is:

```text
Goulart CLI
    |
    +-- Core
    |    +-- lifecycle
    |    +-- policies
    |    +-- artifact contracts
    |    +-- WorkflowEngine interface
    |
    +-- Agent adapters
    |    +-- OpenCode
    |    +-- future adapters
    |
    +-- Workflow engines
         +-- OpenSpecEngine
         +-- future NativeGoulartEngine
```

The important boundary is that the public Goulart workflow should not depend directly on OpenSpec-specific commands.

During v0.2, OpenSpec remains the backend, but OpenSpec-specific execution should become isolated behind an engine implementation. Core, CLI, and agent adapters should be designed so that a different engine can be introduced later without rewriting the public workflow.

Tracking issue: [#56 — v0.2: installable CLI, agent adapters, and engine abstraction](https://github.com/CarlosEGoulart/goulart-sdd/issues/56)

## Planned project installation layout

The final layout is part of the v0.2 design work, but the intended separation is conceptually:

```text
.goulart/
  config / methodology-owned project state

.opencode/
  OpenCode-specific commands and skills
```

Future adapters should own their own agent-specific files without forcing those details into the Goulart core.

## Roadmap

| Version | Focus |
|---|---|
| v0.1 | Methodology bootstrap, strict lifecycle, OpenCode reference adapter, dogfooding |
| v0.2 | CLI installer, agent selection, adapter separation, workflow-engine abstraction, installation E2E, product documentation |
| v0.3 | Evaluate native workflow-engine capabilities and reduce OpenSpec dependence where justified |
| v1.0 | Stable installable workflow with a documented public contract |

Removing OpenSpec is intentionally **not** a v0.2 requirement. The first goal is to isolate it behind a clean boundary and validate the installer/adapters in real use.

## Documentation

- [Methodology](docs/methodology.md) — canonical lifecycle, roles, gates, review, verification, and archive semantics.
- [Getting Started](docs/getting-started.md) — detailed walkthrough for the current v0.1 repository setup.
- [Goulart SDD schema](openspec/schemas/goulart-sdd/) — current OpenSpec-backed artifact contracts.
- [v0.2 tracking issue](https://github.com/CarlosEGoulart/goulart-sdd/issues/56) — installer and decoupling work.

## Development status

The v0.1 bootstrap is complete and the project is now beginning v0.2 development by dogfooding the Goulart SDD lifecycle itself.

Until the v0.2 installer is released, treat this repository as the reference implementation rather than an npm-ready end-user package.
