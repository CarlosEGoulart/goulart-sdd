# Test Plan

## Coverage Ledger

| ID | Spec Path | Requirement | Scenario | Validation Type | Test File/Command | Test Name | Status |
|---|---|---|---|---|---|---|---|
| TP-001 | specs/cli-contract/spec.md | CLI package is installable via npm | Global install | AUTOMATED | packages/cli/test/install.test.ts | globalInstallShowsCommand | NOT_STARTED |
| TP-002 | specs/cli-contract/spec.md | CLI package is installable via npm | npx execution | AUTOMATED | packages/cli/test/install.test.ts | npxExecutesInit | NOT_STARTED |
| TP-003 | specs/cli-contract/spec.md | CLI exposes goulart command | Help output | AUTOMATED | packages/cli/test/cli.test.ts | helpOutputShowsSubcommands | NOT_STARTED |
| TP-004 | specs/cli-contract/spec.md | CLI exposes goulart command | Version output | AUTOMATED | packages/cli/test/cli.test.ts | versionOutputShowsVersion | NOT_STARTED |
| TP-005 | specs/cli-contract/spec.md | CLI reports errors clearly | Unknown subcommand | AUTOMATED | packages/cli/test/cli.test.ts | unknownSubcommandExitsWithError | NOT_STARTED |
| TP-006 | specs/cli-contract/spec.md | CLI reports errors clearly | Missing required input | AUTOMATED | packages/cli/test/cli.test.ts | missingInputExitsWithError | NOT_STARTED |
| TP-007 | specs/coding-agent-selection/spec.md | Agent selection presents supported options | Supported agents listed | AUTOMATED | packages/cli/test/agent-selection.test.ts | agentListShowsOpenCodeAndPlaceholders | NOT_STARTED |
| TP-008 | specs/coding-agent-selection/spec.md | Agent selection presents supported options | Unimplemented agent selected | AUTOMATED | packages/cli/test/agent-selection.test.ts | unimplementedAgentExitsWithError | NOT_STARTED |
| TP-009 | specs/coding-agent-selection/spec.md | Auto-detect existing agent configuration | OpenCode detected | AUTOMATED | packages/cli/test/agent-selection.test.ts | opencodeDetectedAsDefault | NOT_STARTED |
| TP-010 | specs/coding-agent-selection/spec.md | Auto-detect existing agent configuration | No agent detected | AUTOMATED | packages/cli/test/agent-selection.test.ts | noAgentDetectedShowsFullList | NOT_STARTED |
| TP-011 | specs/coding-agent-selection/spec.md | Selected agent is recorded | Agent persisted | AUTOMATED | packages/cli/test/agent-selection.test.ts | agentPersistedToConfig | NOT_STARTED |
| TP-012 | specs/goulart-init/spec.md | init command scaffolds Goulart SDD | Initialize in current directory | AUTOMATED | packages/cli/test/init.test.ts | initInCurrentDirCreatesGoulart | NOT_STARTED |
| TP-013 | specs/goulart-init/spec.md | init command scaffolds Goulart SDD | Initialize in specified directory | AUTOMATED | packages/cli/test/init.test.ts | initInSpecifiedDirCreatesGoulart | NOT_STARTED |
| TP-014 | specs/goulart-init/spec.md | init command scaffolds Goulart SDD | Not a git repository | AUTOMATED | packages/cli/test/init.test.ts | initOutsideGitRepoExitsWithError | NOT_STARTED |
| TP-015 | specs/goulart-init/spec.md | init is interactive by default | Interactive selection | SEMANTIC | packages/cli/test/init.test.ts | initPresentsSelectionPrompt | NOT_STARTED |
| TP-016 | specs/goulart-init/spec.md | init is interactive by default | Agent specified via flag | AUTOMATED | packages/cli/test/init.test.ts | initWithAgentFlagSkipsPrompt | NOT_STARTED |
| TP-017 | specs/goulart-init/spec.md | init creates project configuration | Config file created | AUTOMATED | packages/cli/test/init.test.ts | configFileCreatedWithFields | NOT_STARTED |
| TP-018 | specs/goulart-init/spec.md | init creates project configuration | Config file preserved on re-init | AUTOMATED | packages/cli/test/init.test.ts | reInitPromptsBeforeOverwrite | NOT_STARTED |
| TP-019 | specs/installation-tests/spec.md | Init works in a temporary repository | E2E init in temp repo | AUTOMATED | packages/cli/test/e2e.test.ts | e2eInitInTempRepo | NOT_STARTED |
| TP-020 | specs/installation-tests/spec.md | Init is idempotent in tests | Idempotent re-init | AUTOMATED | packages/cli/test/e2e.test.ts | idempotentReInitProducesIdenticalState | NOT_STARTED |
| TP-021 | specs/installation-tests/spec.md | Config structure is valid | Config validation | AUTOMATED | packages/cli/test/e2e.test.ts | configContainsRequiredFields | NOT_STARTED |
| TP-022 | specs/installation-tests/spec.md | Adapter files match package source | File integrity | AUTOMATED | packages/cli/test/e2e.test.ts | adapterFilesMatchPackageSource | NOT_STARTED |
| TP-023 | specs/opencode-adapter/spec.md | OpenCode adapter files are installed | Commands installed | AUTOMATED | packages/opencode-adapter/test/install.test.ts | commandsInstalledToOpencodeDir | NOT_STARTED |
| TP-024 | specs/opencode-adapter/spec.md | OpenCode adapter files are installed | Skills installed | AUTOMATED | packages/opencode-adapter/test/install.test.ts | skillsInstalledToOpencodeDir | NOT_STARTED |
| TP-025 | specs/opencode-adapter/spec.md | Adapter files are versioned | Version recorded | AUTOMATED | packages/opencode-adapter/test/install.test.ts | adapterVersionRecordedInConfig | NOT_STARTED |
| TP-026 | specs/opencode-adapter/spec.md | Existing adapter files are handled safely | Modified file detected | AUTOMATED | packages/opencode-adapter/test/install.test.ts | modifiedFileDetectedPromptsUser | NOT_STARTED |
| TP-027 | specs/openspec-engine/spec.md | OpenSpecEngine implements WorkflowEngine | Engine instantiation | AUTOMATED | packages/openspec-engine/test/engine.test.ts | openSpecEngineInstantiatesFromConfig | NOT_STARTED |
| TP-028 | specs/openspec-engine/spec.md | OpenSpecEngine delegates to OpenSpec CLI | Plan delegation | AUTOMATED | packages/openspec-engine/test/engine.test.ts | openSpecEngineDelegatesToCli | NOT_STARTED |
| TP-029 | specs/openspec-engine/spec.md | OpenSpecEngine delegates to OpenSpec CLI | OpenSpec not installed | AUTOMATED | packages/openspec-engine/test/engine.test.ts | openSpecNotInstalledReturnsError | NOT_STARTED |
| TP-030 | specs/openspec-engine/spec.md | OpenSpec-specific paths are isolated | Path isolation | AUTOMATED | packages/core/test/architecture.test.ts | noDirectOpenspecExecutionOutsideEngine | NOT_STARTED |
| TP-031 | specs/product-documentation/spec.md | README serves as product landing page | README sections present | MECHANICAL | scripts/check-readme-headings.sh | verifyAllRequiredSections | NOT_STARTED |
| TP-032 | specs/product-documentation/spec.md | README serves as product landing page | Workflow diagram | SEMANTIC | README.md | workflowDiagramReview | NOT_STARTED |
| TP-033 | specs/product-documentation/spec.md | README serves as product landing page | Quick start works | SEMANTIC | README.md | quickStartEvaluation | NOT_STARTED |
| TP-034 | specs/product-documentation/spec.md | Getting Started works for external repositories | External repo initialization | SEMANTIC | docs/getting-started.md | externalRepoInitEvaluation | NOT_STARTED |
| TP-035 | specs/product-documentation/spec.md | Documentation reflects actual implementation | Commands match implementation | AUTOMATED | packages/cli/test/docs-commands.test.ts | documentedCommandsExist | NOT_STARTED |
| TP-036 | specs/project-state/spec.md | Goulart configuration directory | Directory created on init | AUTOMATED | packages/cli/test/project-state.test.ts | goulartDirCreatedOnInit | NOT_STARTED |
| TP-037 | specs/project-state/spec.md | Goulart configuration directory | Directory not created outside init | AUTOMATED | packages/cli/test/project-state.test.ts | goulartDirNotCreatedOutsideInit | NOT_STARTED |
| TP-038 | specs/project-state/spec.md | Configuration file format | Valid config structure | AUTOMATED | packages/cli/test/project-state.test.ts | configStructureMatchesSpec | NOT_STARTED |
| TP-039 | specs/project-state/spec.md | Configuration is human-readable | Hand-editable | SEMANTIC | .goulart/config.yaml | handEditableReview | NOT_STARTED |
| TP-040 | specs/safe-initialization/spec.md | Re-init detects existing installation | Already initialized | AUTOMATED | packages/cli/test/safe-init.test.ts | reInitDetectsExistingInstall | NOT_STARTED |
| TP-041 | specs/safe-initialization/spec.md | User changes are preserved | Modified config preserved | AUTOMATED | packages/cli/test/safe-init.test.ts | modifiedConfigPreserved | NOT_STARTED |
| TP-042 | specs/safe-initialization/spec.md | User changes are preserved | Modified adapter files preserved | AUTOMATED | packages/cli/test/safe-init.test.ts | modifiedAdapterFilesPreserved | NOT_STARTED |
| TP-043 | specs/safe-initialization/spec.md | Init is idempotent for unmodified state | Clean re-init | AUTOMATED | packages/cli/test/safe-init.test.ts | cleanReInitIdempotent | NOT_STARTED |
| TP-044 | specs/safe-initialization/spec.md | Config schema evolution is acknowledged | Incompatible config triggers re-init prompt | AUTOMATED | packages/cli/test/safe-init.test.ts | incompatibleConfigTriggersPrompt | NOT_STARTED |
| TP-045 | specs/workflow-engine/spec.md | WorkflowEngine interface exists | Interface defined | AUTOMATED | packages/core/test/workflow-engine.test.ts | interfaceExportsAllMethods | NOT_STARTED |
| TP-046 | specs/workflow-engine/spec.md | Core does not call OpenSpec directly | No direct openspec calls in core | AUTOMATED | packages/core/test/architecture.test.ts | noOpenspecImportsInCoreCliAdapter | NOT_STARTED |
| TP-047 | specs/workflow-engine/spec.md | Core does not call OpenSpec directly | Engine delegation | AUTOMATED | packages/core/test/workflow-engine.test.ts | operationsDelegatedToEngine | NOT_STARTED |
| TP-048 | specs/workflow-engine/spec.md | Engine is configurable | Engine field present | AUTOMATED | packages/core/test/workflow-engine.test.ts | engineFieldReadFromConfig | NOT_STARTED |
| TP-049 | specs/workflow-engine/spec.md | Engine is configurable | Default engine | AUTOMATED | packages/core/test/workflow-engine.test.ts | defaultEngineIsOpenspec | NOT_STARTED |

## Validation Types

- **AUTOMATED**: Scenario verified by an automated test (unit, integration, E2E). Test file path and test function name recorded.
- **MECHANICAL**: Scenario verified by a deterministic command or tool (lint, schema validation, build, script). Command or script path recorded.
- **SEMANTIC**: Scenario cannot reasonably be validated automatically. Rationale, evaluator, and evaluation description recorded.

## Semantic Entry Detail

| ID | Semantic Rationale | Semantic Evaluator | Evaluation Description |
|---|---|---|---|
| TP-015 | Interactive prompts require a TTY/terminal session to validate the prompts library renders the selection UI correctly; automated test can verify the function is called but not the visual UX | Human evaluator | Run `goulart init` in a terminal, verify the agent selection prompt appears with OpenCode and placeholder agents listed |
| TP-032 | Workflow diagram is a visual artifact in the README; automated text matching cannot assess readability or correctness of a diagram | Human reviewer | Review the workflow section of README.md, confirm the Goulart lifecycle (plan → review → apply → verify → archive) is presented in ASCII or Mermaid format and is readable |
| TP-033 | Quick start time-to-value involves human comprehension and execution speed; cannot be measured by automated tests | Human evaluator | Follow the Quick Start section in README.md from a fresh clone, measure time to successful `goulart init`, confirm it completes within 5 minutes |
| TP-034 | External repo initialization requires manual verification that the Getting Guide works for a user unfamiliar with the bootstrap repository | Human evaluator | Follow Getting Started guide in a fresh git repository (not goulart-sdd), confirm successful initialization without referencing bootstrap repo |
| TP-039 | Hand-editability is a human ergonomics judgment about YAML readability; cannot be assessed by automated means | Human reviewer | Open .goulart/config.yaml in a text editor, confirm fields are readable, comments are clear, and manual edits are respected by subsequent commands |

## Coverage Summary

Total Scenarios: 49
Mapped Scenarios: 49
Unmapped Scenarios: 0

## Coverage Integrity

| Entry | Change | Coverage Impact | Spec Amendment | Rationale |
|---|---|---|---|---|
| TP-027, TP-028, TP-029 | Test paths moved from packages/core/test/ to packages/openspec-engine/test/ | None — same scenarios, corrected package location | None | OpenSpecEngine implementation belongs in packages/openspec-engine per architecture boundary |
| TP-030 | Changed from MECHANICAL grep to AUTOMATED architecture-boundary test | None — same scenario, corrected validation approach | None | Raw grep catches false positives from config values; architecture test checks actual execution coupling |
| TP-031 | Changed from grep count to MECHANICAL script checking each required heading | None — same scenario, corrected validation approach | None | Heading count does not prove required sections exist; script verifies each individually |
| TP-046 | Changed from MECHANICAL grep to AUTOMATED architecture-boundary test | None — same scenario, corrected validation approach | None | Same rationale as TP-030; each entry validates its own distinct scenario |
