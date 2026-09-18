# Plan Review

## Review Metadata

**Review Mode:** Degraded

**Reviewer Context/Session:** Degraded — same session as author (no fresh context available)

**Degraded Review Disclosure:**
This review was conducted in the same session that authored the planning artifacts. No separate/fresh context or user-managed clean session was available. The review is therefore degraded and MUST NOT be described as independent.

- [x] Degraded review limitation disclosed (required if Degraded mode)
- [ ] Human acknowledgement of degraded-review limitation recorded (required if Degraded mode)

---

## Review Round

**ROUND:** 1

**Previous Round Outcome:**

```
N/A — first round
```

---

## Reviewed Inputs

**Review Timestamp:** 2026-09-18

**Proposal:**

- Path: `openspec/changes/v0-2-cli-installer/proposal.md`
- Revision/Commit: b093d53

**Specs:**

| Spec Path | Revision/Commit |
|---|---|
| `openspec/changes/v0-2-cli-installer/specs/cli-contract/spec.md` | b093d53 |
| `openspec/changes/v0-2-cli-installer/specs/goulart-init/spec.md` | b093d53 |
| `openspec/changes/v0-2-cli-installer/specs/coding-agent-selection/spec.md` | b093d53 |
| `openspec/changes/v0-2-cli-installer/specs/opencode-adapter/spec.md` | b093d53 |
| `openspec/changes/v0-2-cli-installer/specs/project-state/spec.md` | b093d53 |
| `openspec/changes/v0-2-cli-installer/specs/safe-initialization/spec.md` | b093d53 |
| `openspec/changes/v0-2-cli-installer/specs/workflow-engine/spec.md` | b093d53 |
| `openspec/changes/v0-2-cli-installer/specs/openspec-engine/spec.md` | b093d53 |
| `openspec/changes/v0-2-cli-installer/specs/installation-tests/spec.md` | b093d53 |
| `openspec/changes/v0-2-cli-installer/specs/product-documentation/spec.md` | b093d53 |

**Design:**

- Path: `openspec/changes/v0-2-cli-installer/design.md`
- Revision/Commit: b093d53

**Additional Reviewed Paths:**

| Path | Revision/Commit | Reason |
|---|---|---|
| `openspec/config.yaml` | b093d53 | Verify schema/context constraints |

---

## Findings

### Finding: F-001

**Severity:** Critical
**Category:** compliance

**Description:**
`project-state/spec.md` defines `.goulart/config.yaml` as containing three required fields: `schema`, `agent`, and `adapterVersion`. However, `workflow-engine/spec.md` adds a `workflowEngine` field to the same config file. `project-state/spec.md` does not mention `workflowEngine`, creating a schema conflict. Two specs disagree on the shape of the same configuration file.

**Evidence/Context:**
- `project-state/spec.md` Scenario "Valid config structure": "it contains at minimum: `schema`, `agent`, and `adapterVersion` fields"
- `workflow-engine/spec.md` Scenario "Engine field present": "it contains a `workflowEngine` field identifying the active engine"

---

### Finding: F-002

**Severity:** Moderate
**Category:** quality

**Description:**
`design.md` CLI framework decision states: "supports subcommands and interactive prompts via `inquirer`/`prompts`". The actual decision is the `prompts` library. The `inquirer` reference is misleading — it implies inquirer is a co-equal alternative or dependency, but the prompts decision section explicitly rejects inquirer ("Heavier, has optional native deps").

**Evidence/Context:**
- `design.md` line 41: "supports subcommands and interactive prompts via `inquirer`/`prompts`"
- `design.md` line 55: "inquirer: Heavier, has optional native deps." (listed as rejected alternative)

---

### Finding: F-003

**Severity:** Critical
**Category:** quality

**Description:**
`design.md` states the WorkflowEngine interface "stays small (5-6 methods) and focused on lifecycle operations." Neither `design.md` nor `workflow-engine/spec.md` enumerate what these methods are. The spec requires "methods for change lifecycle operations" without defining what those operations are. An implementer cannot build the interface without guessing the method signatures, which risks architecture drift between spec and implementation.

**Evidence/Context:**
- `design.md` line 74: "The interface stays small (5-6 methods) and focused on lifecycle operations."
- `workflow-engine/spec.md` Scenario "Interface defined": "a `WorkflowEngine` type/interface is exported with methods for change lifecycle operations"

---

### Finding: F-004

**Severity:** Moderate
**Category:** scope

**Description:**
`product-documentation/spec.md` requires a Getting Started guide: "The Getting Started guide SHALL allow a user to initialize Goulart in a repository other than `goulart-sdd`." The proposal says "Update Getting Started" (existing doc) and separately mentions "New `docs/getting-started.md` or equivalent." The scope is ambiguous — is this an update to an existing file or a new file? The design does not address documentation structure.

**Evidence/Context:**
- `proposal.md` line 16: "Update Getting Started so a new user can initialize Goulart in another repository."
- `proposal.md` line 44: "New `docs/getting-started.md` or equivalent for initialization guide."
- `product-documentation/spec.md` Requirement "Getting Started works for external repositories"

---

### Finding: F-005

**Severity:** Moderate
**Category:** compliance

**Description:**
`coding-agent-selection/spec.md` says agents beyond OpenCode should be "marked as experimental" in the selection list. `design.md` non-goals explicitly state: "Implement Claude Code, Codex, Cursor, or Generic adapters." These are contradictory — the spec wants to list unimplemented agents as selectable options, while the design says not to implement them. The spec should clarify these are display-only placeholders, not functional agents.

**Evidence/Context:**
- `coding-agent-selection/spec.md` Scenario "Supported agents listed": "the CLI displays OpenCode as a supported option and any future agents marked as experimental"
- `design.md` line 20: "Implement Claude Code, Codex, Cursor, or Generic adapters." (non-goal)

---

### Finding: F-006

**Severity:** Suggestion
**Category:** quality

**Description:**
`cli-contract/spec.md` says "The CLI SHALL expose a `goulart` command (or `goulart-sdd` alias)." The proposal uses `goulart-sdd` as the npm package name. The relationship between the npm package name (`goulart-sdd`) and the CLI binary name (`goulart`) is not explicitly documented in any spec or design artifact.

**Evidence/Context:**
- `cli-contract/spec.md` Requirement "CLI exposes goulart command"
- `proposal.md` line 7: "Introduce a `goulart-sdd` / `goulart` CLI published to npm"

---

### Finding: F-007

**Severity:** Suggestion
**Category:** quality

**Description:**
`workflow-engine/spec.md` requires "methods for change lifecycle operations" but does not enumerate what those operations are. This overlaps with F-003 but is a spec-level completeness issue — the spec itself should list the lifecycle operations it covers (plan, review, apply, verify, archive) rather than leaving them implicit.

**Evidence/Context:**
- `workflow-engine/spec.md` Scenario "Interface defined": "methods for change lifecycle operations"

---

### Finding: F-008

**Severity:** Suggestion
**Category:** quality

**Description:**
`product-documentation/spec.md` requires a "workflow" section in the README (Scenario "README sections present"), implying a workflow diagram. `design.md` does not specify what the workflow diagram should contain or how it should be produced (ASCII art, Mermaid, image, etc.).

**Evidence/Context:**
- `product-documentation/spec.md` Scenario "README sections present": "it contains: overview, quick start, installation, workflow, command reference..."

---

### Finding: F-009

**Severity:** Suggestion
**Category:** feasibility

**Description:**
`safe-initialization/spec.md` requires re-init to preserve user-modified config files. If `.goulart/config.yaml` schema changes between package versions (e.g., new required field added), old config files may become invalid. No version migration strategy is defined. This is acceptable for v0.2 given the early stage, but should be acknowledged.

**Evidence/Context:**
- `safe-initialization/spec.md` Scenario "Modified config preserved": "re-init prompts the user before overwriting and shows the diff"

---

### Finding: F-010

**Severity:** Suggestion
**Category:** quality

**Description:**
`cli-contract/spec.md` says the CLI "accepts subcommands" (plural) but only `init` is specified for v0.2. The help output scenario says it "prints available subcommands" but only one exists. This is not a blocking issue but creates a slightly misleading impression of scope.

**Evidence/Context:**
- `cli-contract/spec.md` Requirement "CLI exposes goulart command": "accepts subcommands"
- `cli-contract/spec.md` Scenario "Help output": "prints available subcommands"

---

**Finding Summary:**

| ID | Severity | Category | Status |
|---|---|---|---|
| F-001 | Critical | compliance | Open |
| F-002 | Moderate | quality | Open |
| F-003 | Critical | quality | Open |
| F-004 | Moderate | scope | Open |
| F-005 | Moderate | compliance | Open |
| F-006 | Suggestion | quality | Open |
| F-007 | Suggestion | quality | Open |
| F-008 | Suggestion | quality | Open |
| F-009 | Suggestion | feasibility | Open |
| F-010 | Suggestion | quality | Open |

---

## Verdict

**APPROVE_WITH_CHANGES**

---

## Required Changes

### Change: RC-001

**Requested Change:**
Reconcile `project-state/spec.md` and `workflow-engine/spec.md` on `.goulart/config.yaml` fields. `project-state/spec.md` must include `workflowEngine` in its required fields list (or both specs must agree on a single canonical field list).

**Applied:** No

**Materiality:** MATERIAL

**Materiality Rationale:**
Affects the configuration schema — a core data contract. Two specs currently disagree on the shape of the same file, which would cause implementation ambiguity.

---

### Change: RC-002

**Requested Change:**
Remove the `inquirer` reference from `design.md` CLI framework decision. The line "supports subcommands and interactive prompts via `inquirer`/`prompts`" should read "supports subcommands and interactive prompts via `prompts`" to be consistent with the prompts decision section that rejects inquirer.

**Applied:** No

**Materiality:** NON_MATERIAL

**Materiality Rationale:**
Editorial correction only. The actual decision already correctly specifies `prompts`. The `inquirer` reference is a leftover that does not change requirements, scope, or architecture.

---

### Change: RC-003

**Requested Change:**
Enumerate the WorkflowEngine interface methods in `design.md` (and optionally in `workflow-engine/spec.md`). List the specific lifecycle operations the interface covers (e.g., `plan`, `review`, `apply`, `verify`, `archive`) so the interface contract is unambiguous for implementers.

**Applied:** No

**Materiality:** MATERIAL

**Materiality Rationale:**
Affects architecture clarity. The design claims "5-6 methods" without naming them, and the spec requires "methods for change lifecycle operations" without enumeration. An implementer would have to guess the interface, risking architecture drift.

---

**Required Changes Summary:**

| ID | Applied | Materiality |
|---|---|---|
| RC-001 | No | MATERIAL |
| RC-002 | No | NON_MATERIAL |
| RC-003 | No | MATERIAL |

---

## Human Decision

**STATUS:** REVISE

**Human Reason:**

Apply RC-001, RC-002, and RC-003 in a fresh Author/planning session. RC-001 and RC-003 are MATERIAL and require re-review after correction. RC-002 is NON_MATERIAL but should also be corrected in the same revision. Do not modify proposal.md, specs, design.md, test-plan, or tasks in this reviewer session.

---

## Re-review / Override Evidence

**Affected Inputs (if applicable):**

| Input Path | Change Description |
|---|---|
| | |

**Re-review Waived:** No

**Waiver Reason:**

---

## Escalation Evidence

**Escalation Required:** No

**Escalation Reason:**

**Unresolved Issues Summary:**
