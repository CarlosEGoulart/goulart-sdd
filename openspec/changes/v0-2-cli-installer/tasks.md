# Implementation Tasks

## 1. Monorepo Setup

- [ ] 1.1 Create monorepo structure with `packages/cli/`, `packages/core/`, and `packages/opencode-adapter/` directories. Configure npm workspaces in root `package.json`. Verify `npm install` succeeds across all workspaces.

## 2. Core Package — WorkflowEngine Interface

- [ ] 2.1 Define `WorkflowEngine` interface in `packages/core/src/workflow-engine.ts` with 6 methods (`createChange`, `listChanges`, `getStatus`, `getInstructions`, `validateArtifact`, `archiveChange`) per design.md. TDD: write test TP-045 first confirming interface exports all methods, implement interface, verify test passes.

- [ ] 2.2 Implement config reader in `packages/core/src/config.ts`: read `.goulart/config.yaml`, expose `schema`, `agent`, `adapterVersion`, `workflowEngine` fields per project-state/spec.md. TDD: write test TP-048 first confirming engine field is read, implement reader, verify test passes.

- [ ] 2.3 Implement default engine resolution: when `workflowEngine` absent, default to `openspec` per workflow-engine/spec.md. TDD: write test TP-049 first, implement resolution, verify test passes.

- [ ] 2.4 Implement engine delegation: core lifecycle operations go through configured `WorkflowEngine`, not direct `openspec` calls per workflow-engine/spec.md. TDD: write test TP-047 first confirming delegation to mock, implement wiring, verify test passes.

- [ ] 2.5 Verify path isolation via MECHANICAL check (TP-030, TP-046): run `grep -r "openspec" packages/core/src/` and confirm zero matches.

- [ ] 2.6 Implement `OpenSpecEngine` class in `packages/core/src/openspec-engine.ts` implementing `WorkflowEngine` per openspec-engine/spec.md. TDD: write test TP-027 first, implement class, verify test passes.

- [ ] 2.7 Implement OpenSpec CLI delegation in OpenSpecEngine: operations execute corresponding `openspec` commands per openspec-engine/spec.md. TDD: write test TP-028 first, implement delegation, verify test passes.

- [ ] 2.8 Implement OpenSpec-not-installed error: return clear error when `openspec` CLI unavailable per openspec-engine/spec.md. TDD: write test TP-029 first, implement error path, verify test passes.

## 3. OpenCode Adapter Package

- [ ] 3.1 Create adapter installer in `packages/opencode-adapter/src/install.ts`: copy `goulart-*.md` to `.opencode/commands/` and `goulart-*` dirs to `.opencode/skills/` per opencode-adapter/spec.md. TDD: write tests TP-023, TP-024 first, implement installer, verify tests pass.

- [ ] 3.2 Implement adapter version recording: write `adapterVersion` to `.goulart/config.yaml` matching package version per opencode-adapter/spec.md. TDD: write test TP-025 first, implement version write, verify test passes.

- [ ] 3.3 Implement modified adapter file detection: when file exists and differs from package version, prompt user per opencode-adapter/spec.md. TDD: write test TP-026 first, implement detection, verify test passes.

## 4. CLI Package — Commander.js Setup

- [ ] 4.1 Initialize `packages/cli/` with commander.js dependency. Create `goulart` binary entry point that registers the `init` subcommand per cli-contract/spec.md. TDD: write tests TP-003, TP-004 first, implement CLI shell, verify tests pass.

- [ ] 4.2 Implement error handling: unknown subcommand prints error and exits code 1; missing required input prints explanation and exits code 1 per cli-contract/spec.md. TDD: write tests TP-005, TP-006 first, implement error handling, verify tests pass.

- [ ] 4.3 Implement `--agent` flag on `init`: when provided, skip interactive selection and use specified agent per goulart-init/spec.md. TDD: write test TP-016 first, implement flag, verify test passes.

- [ ] 4.4 Implement `--dir` flag on `init`: initialize in specified directory instead of cwd per goulart-init/spec.md. TDD: write test TP-013 first, implement flag, verify test passes.

## 5. CLI Package — Init Command

- [ ] 5.1 Implement `goulart init` core flow: detect git repo (error if not), create `.goulart/config.yaml` with `schema`, `agent`, `adapterVersion`, `workflowEngine` fields, install adapter files. TDD: write tests TP-012, TP-014, TP-017 first, implement init flow, verify tests pass.

- [ ] 5.2 Implement interactive agent selection via `prompts` library per coding-agent-selection/spec.md: list OpenCode as functional, future agents as experimental placeholders. TDD: write tests TP-007, TP-008 first, implement selection, verify tests pass.

- [ ] 5.3 Implement agent auto-detection: detect `.opencode/` directory and suggest OpenCode as default per coding-agent-selection/spec.md. TDD: write tests TP-009, TP-010 first, implement detection, verify tests pass.

- [ ] 5.4 Implement agent persistence: write selected agent to `.goulart/config.yaml` under `agent` field per coding-agent-selection/spec.md. TDD: write test TP-011 first, implement persistence, verify test passes.

- [ ] 5.5 Implement re-init detection: detect existing `.goulart/` installation and prompt user before overwriting per safe-initialization/spec.md. TDD: write test TP-040 first, implement detection, verify test passes.

- [ ] 5.6 Implement config preservation on re-init: when config manually edited, prompt before overwriting and show diff per safe-initialization/spec.md. TDD: write test TP-041 first, implement preservation, verify test passes.

- [ ] 5.7 Implement adapter file preservation on re-init: when adapter file modified, prompt for that file specifically per safe-initialization/spec.md. TDD: write test TP-042 first, implement preservation, verify test passes.

- [ ] 5.8 Implement idempotent re-init: when no files modified, second run completes without prompts and produces identical state per safe-initialization/spec.md. TDD: write test TP-043 first, implement idempotency, verify test passes.

- [ ] 5.9 Implement config schema evolution handling: detect missing required fields and prompt user before overwriting per safe-initialization/spec.md. TDD: write test TP-044 first, implement detection, verify test passes.

## 6. Project State Verification

- [ ] 6.1 Verify `.goulart/` directory created on init and not created outside init per project-state/spec.md. TDD: write tests TP-036, TP-037 first, implement verification, verify tests pass.

- [ ] 6.2 SEMANTIC evaluation (TP-039): human reviewer opens `.goulart/config.yaml`, confirms readability, edits fields, confirms subsequent commands respect edits.

## 7. E2E Installation Tests

- [ ] 7.1 Implement E2E test in temp repo per installation-tests/spec.md: create temp git repo, run `goulart init --agent opencode`, assert config and adapter files exist. TDD: write tests TP-019, TP-020, TP-021, TP-022 first, implement E2E harness, verify tests pass.

- [ ] 7.2 Implement documented commands verification (TP-035): confirm every documented command exists and produces documented output.

## 8. Documentation

- [ ] 8.1 Rewrite root README.md with required sections per product-documentation/spec.md. MECHANICAL check (TP-031): verify all sections present.

- [ ] 8.2 Add workflow diagram to README showing lifecycle (plan → review → apply → verify → archive) in ASCII or Mermaid per product-documentation/spec.md. SEMANTIC evaluation (TP-032): human reviewer confirms readability.

- [ ] 8.3 SEMANTIC evaluation (TP-033): human evaluator follows Quick Start, confirms initialization completes within 5 minutes.

- [ ] 8.4 Create/update Getting Started guide for external repositories per product-documentation/spec.md. SEMANTIC evaluation (TP-034): human evaluator follows guide in fresh repo, confirms success.
