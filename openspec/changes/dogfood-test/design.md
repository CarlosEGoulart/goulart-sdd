## Context

The Goulart SDD methodology has been fully scaffolded: schema, instructions,
skills, and commands are integrated. The lifecycle has never been exercised on
a real change. A minimal, non-destructive smoke fixture is needed to validate
the full pipeline before using it on production work.

## Goals / Non-Goals

**Goals:**
- Introduce a single OpenCode adapter command that produces a deterministic
  success output.
- Validate that the command can be invoked, reviewed, verified, and archived
  through the Goulart SDD lifecycle.
- Demonstrate one-task-per-invocation TDD on a trivially testable behavior.

**Non-Goals:**
- Building a general-purpose dogfooding framework.
- Exercising every lifecycle branch (downstream tasks handle additional gates).
- Producing a reusable adapter or CLI utility.
- Modifying production source code, schemas, or configuration.

## Decisions

### Command format: markdown command file

**Decision:** Implement `/goulart-dogfood-smoke` as a markdown command file
under `.opencode/commands/`.

**Rationale:** OpenCode adapter commands follow the same `.md` convention as
existing `goulart-*` commands. This keeps the fixture consistent with the
project's adapter pattern and requires zero new infrastructure.

**Alternatives considered:**
- Shell script: Rejected because it would bypass the adapter command pattern
  and introduce a different execution model.
- Inline function in OpenCode config: Not supported by the current adapter
  surface.

### Output: deterministic fixed string

**Decision:** The command outputs exactly `dogfood-smoke: OK` as its result.

**Rationale:** A fixed, human-readable string satisfies the deterministic
requirement with zero configuration. The string is easy to verify mechanically
and hard to misinterpret.

### No-mutation enforcement: author discipline + review

**Decision:** The command performs no file writes, git operations, or schema
mutations by construction. Enforcement relies on the command's trivial
implementation and adversarial code review.

**Rationale:** For a single-line output command, the mutation surface is
effectively zero. Adding runtime guards would be disproportionate to the risk.

## Risks / Trade-offs

[Risk] The fixture may be forgotten after the smoke-test sequence.
Mitigation: The spec requires a removal tracking task. The temporary nature
is documented in the command file itself.

[Risk] The deterministic output assumption may not survive adapter changes.
Mitigation: The spec constrains the output to a fixed string; adapter
upgrades that break this would be caught by the smoke test itself.

## Open Questions

(none)
