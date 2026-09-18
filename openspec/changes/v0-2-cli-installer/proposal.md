## Why

Goulart SDD v0.1 established the methodology on top of OpenSpec and validated the lifecycle through dogfooding, but it lives only as a repository-hosted reference implementation. Developers cannot install it with a single command, must manually copy files, and cannot use it outside the bootstrap repository. v0.2 transforms Goulart SDD into an installable developer tool with a clean CLI, agent-adaptive installation, and a workflow-engine boundary that keeps OpenSpec as a replaceable backend.

## What Changes

- Introduce a `goulart-sdd` / `goulart` CLI published to npm with an `init` command.
- Add interactive coding-agent selection during initialization.
- Install OpenCode adapter files (commands, skills) into the target repository.
- Define a Goulart-owned project configuration and state layout separate from agent-specific files.
- Make initialization safe and idempotent — re-running must not silently overwrite user changes.
- Introduce a `WorkflowEngine` abstraction so core/CLI/adapters do not call OpenSpec directly.
- Implement `OpenSpecEngine` as the v0.2 workflow backend behind the engine boundary.
- Add automated installation tests, including temporary-repository E2E paths.
- Rewrite the root README into a usable product entry point (installation, quick start, architecture, roadmap).
- Update Getting Started so a new user can initialize Goulart in another repository.

## Capabilities

### New Capabilities

- `cli-contract`: Public CLI interface definition — command names, flags, output format, and error behavior for v0.2.
- `goulart-init`: The `goulart init` command — interactive initialization flow, target repository detection, and file installation.
- `coding-agent-selection`: Interactive coding-agent selection and auto-detection during initialization.
- `opencode-adapter`: OpenCode adapter installation — commands, skills, and configuration placement for the target repository.
- `project-state`: Goulart-owned project configuration and state layout — `.goulart/` structure, schema references, and configuration format.
- `safe-initialization`: Idempotent initialization — re-run behavior, user-change detection, conflict resolution, and overwrite protection.
- `workflow-engine`: `WorkflowEngine` abstraction — interface definition separating core lifecycle from workflow-backend execution.
- `openspec-engine`: `OpenSpecEngine` backend — implementation of `WorkflowEngine` that delegates to OpenSpec for v0.2.
- `installation-tests`: Automated installation tests — temporary-repository E2E paths and initialization behavior validation.
- `product-documentation`: README rewrite and Getting Started updates — installation, quick start, workflow diagram, architecture, and agent support documentation.

### Modified Capabilities

(none — this is the first spec-bearing change; all capabilities are new)

## Impact

- New `packages/cli/` directory (or equivalent) containing the CLI package.
- New `packages/core/` directory for workflow-engine abstraction and core lifecycle.
- New `packages/opencode-adapter/` (or equivalent) for OpenCode-specific adapter code.
- npm package publishing: `goulart-sdd` becomes an installable package.
- Root README rewritten from 2-line placeholder to full product documentation.
- Existing Getting Started documentation updated with initialization guide for external repositories.
- No production source code, APIs, or external dependencies are affected beyond the new packages.
- OpenSpec remains a dependency but is isolated behind the engine boundary.
