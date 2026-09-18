## 1. Custom Schema Foundation

- [x] 1.1 Fork `spec-driven` schema into `openspec/schemas/goulart-sdd/` using `openspec schema fork spec-driven goulart-sdd` and verify the directory structure contains `schema.yaml` and `templates/`
- [x] 1.2 Review the forked `schema.yaml` to understand the base artifact graph and verify the file is valid YAML with `openspec schema validate goulart-sdd`
- [x] 1.3 Add the `plan-review` artifact to `schema.yaml` with `requires: [proposal, specs, design]` and verify `openspec schema validate goulart-sdd` passes
- [x] 1.4 Add the `test-plan` artifact to `schema.yaml` with `requires: [specs, plan-review]` and verify validation passes
- [x] 1.5 Add the `code-review` artifact to `schema.yaml` with `requires: [tasks]` and verify validation passes
- [x] 1.6 Add the `verify` artifact to `schema.yaml` with `requires: [code-review]` and verify validation passes
- [x] 1.7 Update the `tasks` artifact `requires` to `[test-plan, design, plan-review]` and verify the full dependency graph resolves correctly with `openspec schema validate goulart-sdd`
- [x] 1.8 Set `goulart-sdd` as the default schema in `openspec/config.yaml` by updating the `schema:` field and verify `openspec schemas --json` lists it

## 2. Schema Templates

- [x] 2.1 Create `openspec/schemas/goulart-sdd/templates/plan-review.md` with Review Metadata (including degraded disclosure/acknowledgement), ROUND: 1 | 2 and previous outcome, Reviewed Inputs (revision/paths), Findings (severity + category), Verdict, Required Changes with materiality rationale, Human Decision and Human Reason (including explicit overrides); verify the template supports each plan-review verdict/disposition transition
- [x] 2.2 Create `openspec/schemas/goulart-sdd/templates/code-review.md` with Review Metadata (including degraded disclosure/acknowledgement), ROUND: 1 | 2 and previous outcome, Reviewed Implementation (revision/paths), Findings (severity + category), Verdict, Per-Finding Human Triage (ACCEPT/REJECT/DEFER), Required Changes with materiality justification, and any explicit override/reason; verify it supports each code-review transition without requiring approval for a clean review
- [x] 2.3 Create `openspec/schemas/goulart-sdd/templates/test-plan.md` with table format for: Requirement, Scenario, Validation Type (AUTOMATED/MECHANICAL/SEMANTIC), Test File/Command, Test Name, Status, and verify the file exists
- [x] 2.4 Create `openspec/schemas/goulart-sdd/templates/verify.md` with sections for: Prerequisites Checked, Reviewed Revision, Reviewed Paths, Task Completion, Test Integrity, Review Staleness (plan-review and code-review), Scope Drift, Decision (PASS/PASS_WITH_WARNINGS/FAIL), and verify the file exists
- [x] 2.5 Modify `openspec/schemas/goulart-sdd/templates/tasks.md` to include TDD ordering instructions (write failing test, implement to pass, refactor) and verify the file exists
- [x] 2.6 Run `openspec schema validate goulart-sdd` and confirm all templates resolve without errors

## 3. Schema Instructions

- [x] 3.1 Write the `plan-review` instruction in `schema.yaml` — independent versus degraded review, fallback hierarchy, adversarial attack surface, complete verdict/disposition matrix, applied Required Changes with material re-review versus justified non-material acceptance, explicit OVERRIDDEN/reason, severity rules, two-round persistence/escalation including APPROVE_WITH_CHANGES, and read-only posture; verify instructions match the plan-review spec scenarios
- [x] 3.2 Write the `test-plan` instruction in `schema.yaml` — three validation types (AUTOMATED/MECHANICAL/SEMANTIC), scenario-to-validation mapping, coverage ledger format, behavioral integrity rule, and verify the instruction field is populated
- [x] 3.3 Write the `code-review` instruction in `schema.yaml` — independent/degraded fallback, evaluation scope, full APPROVE/APPROVE_WITH_CHANGES/REVISE transitions, finding triage with justified REJECT/DEFER, material-fix staleness and justified non-material exception, explicit override/reason, and two-round persistence/escalation; verify each transition against the code-review spec
- [x] 3.4 Write the `verify` instruction in `schema.yaml` — independently check code-review existence/permitted verdict, complete mandatory triage, all tasks complete, final evaluable test-plan entries, review freshness and any permitted explicit override; then audit compliance, tests, and both staleness lineages using revision/diff plus semantic judgment (no timestamps), and emit DECISION with warning semantics and reviewed metadata; verify incomplete prerequisites block and overrides do not imply freshness
- [x] 3.5 Update the `tasks` instruction in `schema.yaml` to enforce TDD ordering per scenario and verify the instruction field is updated
- [x] 3.6 Run `openspec schema validate goulart-sdd --verbose` and confirm all artifacts and instructions are valid

## 4. goulart-plan Adapter

- [x] 4.1 Create `.opencode/skills/goulart-plan/SKILL.md` with state-aware initial planning (proposal/specs/design → STOP → fresh-session plan review), later continuation through test-plan/tasks only when verdict/staleness/human gates permit it, exact unresolved-gate reporting, and planning-complete → STOP → goulart-apply guidance; verify all workflow state transitions are covered without another command
- [x] 4.2 Create `.opencode/commands/goulart-plan.md` command file that invokes the goulart-plan skill and verify the file exists

## 5. goulart-review Adapter

- [x] 5.1 Create `.opencode/skills/goulart-review/SKILL.md` for plan/code reviews using their separate artifact contracts, independent-context versus disclosed/acknowledged degraded fallback, severity/category findings, stage-specific verdict transitions, human plan disposition versus conditional code triage, materiality/staleness, explicit overrides, and bounded round persistence; verify behavior matches both review specs
- [x] 5.2 Create `.opencode/commands/goulart-review.md` command file that invokes the goulart-review skill and verify the file exists

## 6. goulart-apply Adapter

- [x] 6.1 Create `.opencode/skills/goulart-apply/SKILL.md` with instructions for: one-task-per-invocation baseline, select first eligible pending task, load relevant context (task/spec/design/test-plan), allow repository exploration, TDD execution (RED-GREEN-REFACTOR), mark task complete, report result, STOP. When all tasks complete, instruct user to run `goulart-review code` in a fresh session. Pre-implementation gate checks (plan-review verdict, human acceptance, test-plan existence). And verify the file exists
- [x] 6.2 Create `.opencode/commands/goulart-apply.md` command file that invokes the goulart-apply skill and verify the file exists

## 7. goulart-verify Adapter

- [x] 7.1 Create `.opencode/skills/goulart-verify/SKILL.md` that independently blocks on missing/non-permitting code-review, incomplete mandatory triage, incomplete tasks, unevaluable test-plan entries, or unresolved stale-review gates; assess recorded review exceptions without claiming freshness, then audit spec compliance, all three validation types, and two-lineage staleness (including accepted-finding fixes) and produce DECISION/reviewed metadata; verify prerequisite-blocking and result-production scenarios match the verification spec
- [x] 7.2 Create `.opencode/commands/goulart-verify.md` command file that invokes the goulart-verify skill and verify the file exists

## 8. goulart-archive Adapter

- [x] 8.1 Create `.opencode/skills/goulart-archive/SKILL.md` with instructions for: check verify decision — PASS delegates to OpenSpec archive, PASS_WITH_WARNINGS delegates only after human warning dispositions are recorded, FAIL blocks archive (human override must be explicit with reason). Do NOT modify upstream opsx-archive or OpenSpec archive behavior. And verify the file exists
- [x] 8.2 Create `.opencode/commands/goulart-archive.md` command file that invokes the goulart-archive skill and verify the file exists

## 9. Project Configuration

- [x] 9.1 Update `openspec/config.yaml` with project context describing Goulart SDD, its purpose, and constraints, and verify the file is valid YAML
- [x] 9.2 Add per-artifact rules to `openspec/config.yaml` for `specs` (require SHALL/MUST, every scenario must be testable) and `tasks` (TDD ordering, small tasks), and verify the config file parses correctly

## 10. Methodology Documentation

- [x] 10.1 Create `docs/methodology.md` explaining the workflow, roles, gates, state-aware planning continuation, raw OpenSpec full-planning escape hatch, one-task-per-invocation TDD, independent/degraded review, verdict/disposition and two-round staleness rules, and verification prerequisites; verify documentation agrees with the lifecycle contracts and deferred scope
- [x] 10.2 Create `docs/getting-started.md` with installation steps and a first-change walkthrough covering initial goulart-plan → review/human disposition → later goulart-plan → goulart-apply guidance, plus Goulart-compliant versus raw execution; verify the walkthrough includes both planning stops

## 11. Dogfooding Verification

These are lightweight adapter/schema smoke checks; documented manual evaluation is acceptable where automation is impractical. No complete fake application is required. Validate populated planning artifacts rather than treating a newly created empty change as a valid completed plan.

- [ ] 11.1 Create a test change using the `goulart-sdd` schema (`openspec new change "dogfood-test" --schema goulart-sdd`) and verify the change directory is created with the correct schema
- [ ] 11.2 Verify that the test change's `.openspec.yaml` references the `goulart-sdd` schema and that `openspec status` shows all expected artifacts (proposal, specs, design, plan-review, test-plan, tasks, code-review, verify)
- [ ] 11.3 Smoke test planning gate: run `goulart-plan` on the test change, verify it creates proposal/specs/design and stops before plan-review/test-plan/tasks
- [ ] 11.4 Smoke test apply gate: on a temporary change without valid plan-review/human acceptance, verify `goulart-apply` refuses implementation
- [ ] 11.5 Smoke test archive gate: on a temporary change without valid verify state, verify `goulart-archive` refuses archive delegation
- [ ] 11.6 Smoke test planning continuation: prepare a valid plan-review with permitted human acceptance after the initial planning stop; invoke goulart-plan again and verify it creates/continues test-plan and tasks, reports planning complete, STOPs, and identifies goulart-apply as the next step without executing it
- [ ] 11.7 After planning smoke checks populate the test change, run `openspec validate dogfood-test` and confirm no blocking errors, recording any warnings
- [ ] 11.8 Clean up temporary dogfood change directories after verification
