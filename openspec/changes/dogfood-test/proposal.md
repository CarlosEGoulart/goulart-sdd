## Why

The Goulart SDD lifecycle has been fully scaffolded but never exercised end-to-end
on a real change. A lightweight, non-destructive smoke fixture is needed to
validate that the goulart-plan → goulart-review → goulart-apply → goulart-verify
pipeline operates correctly before using it on production work.

## What Changes

- Introduce a temporary `/goulart-dogfood-smoke` OpenCode adapter command.
- When invoked, the command reports a deterministic successful smoke result.
- The command performs no repository mutation (no file writes, no git operations,
  no schema changes).
- The command exists only as a dogfooding fixture and will be removed after the
  smoke-test sequence completes.

## Capabilities

### New Capabilities

- `dogfood-smoke-command`: Temporary adapter command that exercises the
  Goulart SDD lifecycle with a deterministic, mutation-free smoke result.

### Modified Capabilities

(none)

## Impact

- OpenCode adapter command surface: one new temporary command added to
  `.opencode/commands/`.
- No production source code, APIs, dependencies, or systems are affected.
- No schema, template, or configuration files are modified.
