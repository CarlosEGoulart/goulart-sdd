## Context

Goulart SDD v0.1 is a repository-hosted reference implementation. All methodology files (schema, commands, skills) live in `.opencode/` and `openspec/` within the bootstrap repository. There is no CLI, no npm package, no way for a developer to install Goulart in another project without manually copying files.

The v0.2 design must transform this into an installable tool while preserving the OpenSpec backend as a replaceable implementation detail. See proposal.md — Why.

## Goals / Non-Goals

**Goals:**
- Deliver a `goulart` CLI published to npm with an `init` command.
- Install OpenCode adapter files into the target repository.
- Separate Goulart-owned state (`.goulart/`) from agent-specific files (`.opencode/`).
- Define a `WorkflowEngine` interface that isolates core lifecycle from backend execution.
- Implement `OpenSpecEngine` as the v0.2 backend behind that interface.
- Ensure re-initialization is safe and idempotent.
- Provide automated E2E tests for installation.

**Non-Goals:**
- Remove OpenSpec or replace it with a native engine in v0.2.
- Implement Claude Code, Codex, Cursor, or Generic adapters.
- Provide a full package manager (yarn/pnpm) adapter system.
- Build a GUI or web-based configuration interface.
- Migrate existing v0.1 bootstrap users automatically.

## Decisions

### Package structure: monorepo with workspaces

**Decision:** Use a monorepo layout with separate packages for CLI, core, and the OpenCode adapter.

**Rationale:** Separates concerns (CLI parsing vs lifecycle logic vs agent-specific file installation), allows independent testing, and keeps the door open for future adapter packages. Uses npm workspaces for simplicity.

**Alternatives considered:**
- Single package: Rejected because it would mix CLI parsing, core lifecycle, and adapter-specific file copying in one module, making it harder to enforce the engine boundary.
- Separate repositories: Rejected because it would create versioning and release coordination overhead for a small project.

### CLI framework: commander.js

**Decision:** Use `commander.js` for CLI argument parsing.

**Rationale:** Lightweight, well-maintained, zero-config, supports subcommands and interactive prompts via `inquirer`/`prompts`. No build step needed for the CLI layer itself.

**Alternatives considered:**
- yargs: Slightly heavier, more features than needed for v0.2.
- oclif: Salesforce ecosystem, heavy scaffolding, overkill for this scope.
- Custom parser: Fragile and wastes time reinventing basic parsing.

### Interactive prompts: prompts (term-kit alternative)

**Decision:** Use the `prompts` library for interactive coding-agent selection.

**Rationale:** Lightweight, no native dependencies, works in all terminals, supports list/confirm/input types. Avoids the native compilation issues of `inquirer` or `term-kit`.

**Alternatives considered:**
- inquirer: Heavier, has optional native deps.
- readline directly: Too low-level for a pleasant UX.
- Non-interactive only: Rejected because the user requirement explicitly calls for agent selection.

### Config location: .goulart/

**Decision:** Use `.goulart/` as the project-owned configuration directory.

**Rationale:** Follows the dot-directory convention (`.git/`, `.github/`, `.vscode/`), clearly separates Goulart state from agent files, and is distinct from `.opencode/` which remains agent-owned.

**Alternatives considered:**
- Root `goulart.yaml`: Would mix Goulart config with other project config files.
- `openspec/` reuse: Would blur the boundary between Goulart and OpenSpec.
- Inside `.opencode/`: Would make Goulart state agent-dependent, violating the architecture constraint.

### Engine abstraction: strategy pattern

**Decision:** Implement `WorkflowEngine` as a strategy/interface with method signatures matching the core lifecycle operations (plan, review, apply, verify, archive).

**Rationale:** Classic strategy pattern allows swapping backends without changing core logic. The interface stays small (5-6 methods) and focused on lifecycle operations.

**Alternatives considered:**
- Plugin system: Over-engineered for v0.2 when there's only one backend.
- Event-driven: Adds complexity without clear benefit at this scale.
- Direct function imports: Would defeat the purpose of the engine boundary.

### Testing strategy: temporary repository E2E

**Decision:** Test installation by creating temporary git repositories, running `goulart init`, and asserting file existence and content.

**Rationale:** Directly validates the user-facing workflow. Temp repos provide isolation. No mocking needed for the most critical path.

**Alternatives considered:**
- Unit tests only: Would miss integration issues with file system, git, and config parsing.
- Mock-based tests: Would not catch real file installation problems.
- Docker-based tests: Too heavy for CI, slow feedback loop.

## Risks / Trade-offs

[Risk] OpenSpec CLI dependency creates a runtime requirement.
Mitigation: Document clearly that OpenSpec is required for v0.2. The engine boundary allows replacing it in v0.3+.

[Risk] Commander.js may not cover all future CLI needs (auto-update, plugins).
Mitigation: v0.2 scope is intentionally limited. Can migrate to oclif or custom parser later if needed.

[Risk] `.goulart/` directory may conflict with other tools.
Mitigation: Dot-directories are conventionally tool-specific. No known conflicts exist.

[Risk] Monorepo adds complexity over a single package.
Mitigation: npm workspaces are simple and well-understood. The complexity is justified by the clean separation.

[Risk] Re-init safety depends on accurate file fingerprinting.
Mitigation: Use content hashing rather than timestamps. Test with explicit diff detection.

## Open Questions

(none — all build-changing questions are resolved in the Decisions section above)
