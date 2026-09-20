# Implementation Tasks

## 1. Monorepo Setup

- [ ] 1.1 Create monorepo structure with `packages/cli/`, `packages/core/`, `packages/opencode-adapter/`, and `packages/openspec-engine/` directories. Configure npm workspaces in root `package.json`. Verify `npm install` succeeds across all workspaces.

## 2. Core Package — WorkflowEngine Interface

- [ ] 2.1 Define `WorkflowEngine` interface in `packages/core/src/workflow-engine.ts` with 6 methods (`createChange`, `listChanges`, `getStatus`, `getInstructions`, `validateArtifact`, `archiveChange`) per design.md. TDD: write test TP-045 first confirming interface exports all methods, implement interface, verify test passes.

- [ ] 2.2 Implement config reader in `packages/core/src/config.ts`: read `.goulart/config.yaml`, expose `schema`, `agent`, `adapterVersion`, `workflowEngine` fields per project-state/spec.md. TDD: write test TP-048 first confirming engine field is read, implement reader, verify test passes.

- [ ] 2.3 Implement default engine resolution: when `workflowEngine` absent, default to `openspec` per workflow-engine/spec.md. TDD: write test TP-049 first, implement resolution, verify test passes.

- [ ] 2.4 Implement engine delegation: core lifecycle operations go through configured `WorkflowEngine`, not direct `openspec` calls per workflow-engine/spec.md. TDD: write test TP-047 first confirming delegation to mock, implement wiring, verify test passes.

- [ ] 2.5 Verify architecture boundary (TP-030): AUTOMATED test in `packages/core/test/architecture.test.ts` confirms no direct OpenSpec execution or OpenSpec-internal imports exist in `packages/core/src/`, `packages/cli/src/`, or `packages/opencode-adapter/src/`. TDD: write test first, confirm it passes against clean source.

- [ ] 2.6 Verify architecture boundary (TP-046): AUTOMATED test in `packages/core/test/architecture.test.ts` confirms core, CLI, and adapter packages do not import OpenSpec-specific modules; only `packages/openspec-engine/` may contain OpenSpec execution. TDD: write test first, confirm it passes against clean source.

## 3. OpenCode Adapter Package

- [ ] 3.1 Create adapter installer in `packages/opencode-adapter/src/install.ts` for copying command files. TDD: write test TP-023 first confirming `goulart-*.md` files are installed to `.opencode/commands/`, implement installer, verify test passes.

- [ ] 3.2 Extend adapter installer for skill directory installation. TDD: write test TP-024 first confirming `goulart-*` directories are installed to `.opencode/skills/`, implement skill copy, verify test passes.

- [ ] 3.3 Implement adapter version recording: write `adapterVersion` to `.goulart/config.yaml` matching package version per opencode-adapter/spec.md. TDD: write test TP-025 first, implement version write, verify test passes.

- [ ] 3.4 Implement modified adapter file detection: when file exists and differs from package version, prompt user per opencode-adapter/spec.md. TDD: write test TP-026 first, implement detection, verify test passes.

## 4. CLI Package — Commander.js Setup

- [ ] 4.1 Initialize `packages/cli/` with commander.js dependency. Create `goulart` binary entry point that registers the `init` subcommand per cli-contract/spec.md. TDD: write test TP-003 first confirming help output shows subcommands, implement CLI shell, verify test passes.

- [ ] 4.2 Implement version output: `goulart --version` prints the current version number per cli-contract/spec.md. TDD: write test TP-004 first, implement version flag, verify test passes.

- [ ] 4.3 Implement unknown subcommand error: prints error listing available commands and exits code 1 per cli-contract/spec.md. TDD: write test TP-005 first, implement error handling, verify test passes.

- [ ] 4.4 Implement missing required input error: prints explanation and exits code 1 per cli-contract/spec.md. TDD: write test TP-006 first, implement error handling, verify test passes.

- [ ] 4.5 Implement `--agent` flag on `init`: when provided, skip interactive selection and use specified agent per goulart-init/spec.md. TDD: write test TP-016 first, implement flag, verify test passes.

- [ ] 4.6 Implement `--dir` flag on `init`: initialize in specified directory instead of cwd per goulart-init/spec.md. TDD: write test TP-013 first, implement flag, verify test passes.

## 5. CLI Package — Init Command

- [ ] 5.1 Implement `goulart init` git repo detection: error and exit code 1 when not in a git repository per goulart-init/spec.md. TDD: write test TP-014 first, implement detection, verify test passes.

- [ ] 5.2 Implement `goulart init` core scaffolding: create `.goulart/config.yaml` with `schema`, `agent`, `adapterVersion`, `workflowEngine` fields per project-state/spec.md. TDD: write test TP-017 first, implement config creation, verify test passes.

- [ ] 5.3 Implement `goulart init` current directory flow: when run in a git repo, creates `.goulart/` and installs adapter files per goulart-init/spec.md. TDD: write test TP-012 first, implement init flow, verify test passes.

- [ ] 5.4 Implement interactive agent selection via `prompts` library per coding-agent-selection/spec.md: list OpenCode as functional, future agents as experimental placeholders. TDD: write test TP-007 first, implement agent list, verify test passes.

- [ ] 5.5 Implement unimplemented agent error: selecting agent without installed adapter prints error and exits code 1 per coding-agent-selection/spec.md. TDD: write test TP-008 first, implement error, verify test passes.

- [ ] 5.6 Implement agent auto-detection: detect `.opencode/` directory and suggest OpenCode as default per coding-agent-selection/spec.md. TDD: write test TP-009 first, implement detection, verify test passes.

- [ ] 5.7 Implement no-agent-detected path: when no agent config found, present full selection list without default per coding-agent-selection/spec.md. TDD: write test TP-010 first, implement path, verify test passes.

- [ ] 5.8 Implement agent persistence: write selected agent to `.goulart/config.yaml` under `agent` field per coding-agent-selection/spec.md. TDD: write test TP-011 first, implement persistence, verify test passes.

- [ ] 5.9 Implement re-init detection: detect existing `.goulart/` installation and prompt user before overwriting per safe-initialization/spec.md. TDD: write test TP-040 first, implement detection, verify test passes.

- [ ] 5.10 Implement config preservation on re-init: when config manually edited, prompt before overwriting and show diff per safe-initialization/spec.md. TDD: write test TP-041 first, implement preservation, verify test passes.

- [ ] 5.11 Implement adapter file preservation on re-init: when adapter file modified, prompt for that file specifically per safe-initialization/spec.md. TDD: write test TP-042 first, implement preservation, verify test passes.

- [ ] 5.12 Implement idempotent re-init: when no files modified, second run completes without prompts and produces identical state per safe-initialization/spec.md. TDD: write test TP-043 first, implement idempotency, verify test passes.

- [ ] 5.13 Implement config schema evolution handling: detect missing required fields and prompt user before overwriting per safe-initialization/spec.md. TDD: write test TP-044 first, implement detection, verify test passes.

- [ ] 5.14 Implement re-init config preservation prompt: config file preserved on re-init per goulart-init/spec.md. TDD: write test TP-018 first, implement prompt, verify test passes.

## 6. Project State Verification

- [ ] 6.1 Verify `.goulart/` directory created on init per project-state/spec.md. TDD: write test TP-036 first, implement verification, verify test passes.

- [ ] 6.2 Verify `.goulart/` directory not created outside init per project-state/spec.md. TDD: write test TP-037 first, implement verification, verify test passes.

- [ ] 6.3 SEMANTIC evaluation (TP-039): human reviewer opens `.goulart/config.yaml`, confirms readability, edits fields, confirms subsequent commands respect edits.

## 7. OpenSpec Engine Package

- [ ] 7.1 Implement `OpenSpecEngine` class in `packages/openspec-engine/src/openspec-engine.ts` implementing `WorkflowEngine` per openspec-engine/spec.md. TDD: write test TP-027 first confirming instantiation from config, implement class, verify test passes.

- [ ] 7.2 Implement OpenSpec CLI delegation in OpenSpecEngine: operations execute corresponding `openspec` commands per openspec-engine/spec.md. TDD: write test TP-028 first confirming delegation to mock CLI, implement delegation, verify test passes.

- [ ] 7.3 Implement OpenSpec-not-installed error: return clear error when `openspec` CLI unavailable per openspec-engine/spec.md. TDD: write test TP-029 first, implement error path, verify test passes.

## 8. E2E Installation Tests

- [ ] 8.1 Implement E2E init in temp repo: create temp git repo, run `goulart init --agent opencode`, assert `.goulart/config.yaml` and adapter files exist per installation-tests/spec.md. TDD: write test TP-019 first, implement E2E test, verify test passes.

- [ ] 8.2 Implement E2E idempotent re-init: run init twice in temp repo, assert byte-identical state per installation-tests/spec.md. TDD: write test TP-020 first, implement idempotency check, verify test passes.

- [ ] 8.3 Implement E2E config validation: after init, assert `.goulart/config.yaml` contains `schema`, `agent`, `adapterVersion`, `workflowEngine` with non-empty values per installation-tests/spec.md. TDD: write test TP-021 first, implement validation, verify test passes.

- [ ] 8.4 Implement E2E file integrity: after init, assert installed adapter files match package source per installation-tests/spec.md. TDD: write test TP-022 first, implement integrity check, verify test passes.

- [ ] 8.5 Implement documented commands verification (TP-035): confirm every command documented in README exists and produces documented output.

## 9. Documentation

- [ ] 9.1 Rewrite root README.md with required sections per product-documentation/spec.md. MECHANICAL check (TP-031): run `scripts/check-readme-headings.sh` to verify all 11 required section headings are present.

- [ ] 9.2 Add workflow diagram to README showing lifecycle (plan → review → apply → verify → archive) in ASCII or Mermaid per product-documentation/spec.md. SEMANTIC evaluation (TP-032): human reviewer confirms readability.

- [ ] 9.3 SEMANTIC evaluation (TP-033): human evaluator follows Quick Start, confirms initialization completes within 5 minutes.

- [ ] 9.4 Create/update Getting Started guide for external repositories per product-documentation/spec.md. SEMANTIC evaluation (TP-034): human evaluator follows guide in fresh repo, confirms success.
