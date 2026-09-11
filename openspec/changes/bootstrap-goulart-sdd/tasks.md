## 1. Custom Schema Foundation

- [ ] 1.1 Fork `spec-driven` schema into `openspec/schemas/goulart-sdd/` using `openspec schema fork spec-driven goulart-sdd` and verify the directory structure contains `schema.yaml` and `templates/`
- [ ] 1.2 Review the forked `schema.yaml` to understand the base artifact graph and identify where goulart-sdd additions will be made, verify the file is valid YAML with `openspec schema validate goulart-sdd`
- [ ] 1.3 Add the `plan-review` artifact to `schema.yaml` with `requires: [proposal, specs, design]` and verify `openspec schema validate goulart-sdd` passes
- [ ] 1.4 Add the `test-plan` artifact to `schema.yaml` with `requires: [specs, plan-review]` and verify validation passes
- [ ] 1.5 Add the `code-review` artifact to `schema.yaml` with `requires: [tasks]` and verify validation passes
- [ ] 1.6 Add the `verify` artifact to `schema.yaml` with `requires: [tasks]` and verify validation passes
- [ ] 1.7 Update the `tasks` artifact `requires` to `[test-plan, design, plan-review]` (replacing `[specs, design]`) and verify the full dependency graph resolves correctly with `openspec schema validate goulart-sdd`
- [ ] 1.8 Set `goulart-sdd` as the default schema in `openspec/config.yaml` by updating the `schema:` field and verify `openspec schemas --json` lists it

## 2. Schema Templates

- [ ] 2.1 Create `openspec/schemas/goulart-sdd/templates/review.md` with sections for: Review Metadata, Findings (severity + category), Verdict, Required Changes, and verify the file exists
- [ ] 2.2 Create `openspec/schemas/goulart-sdd/templates/test-plan.md` with table format for: Requirement, Scenario, Test File, Test Name, Initial State, and verify the file exists
- [ ] 2.3 Create `openspec/schemas/goulart-sdd/templates/verify.md` with sections for: Task Completion, Test Integrity, Review Integrity, Scope Drift, Decision (PASS/PASS_WITH_WARNINGS/FAIL), and verify the file exists
- [ ] 2.4 Modify `openspec/schemas/goulart-sdd/templates/tasks.md` to include TDD ordering instructions (write failing test → implement → refactor) and verify the file exists
- [ ] 2.5 Run `openspec schema validate goulart-sdd` and confirm all templates resolve without errors

## 3. Schema Instructions

- [ ] 3.1 Write the `plan-review` instruction in `schema.yaml` — fresh-context requirement, adversarial attack surface, VERDICT format, severity rules, bounded rounds, and verify the instruction field is populated
- [ ] 3.2 Write the `test-plan` instruction in `schema.yaml` — scenario-to-test mapping, coverage ledger format, non-executable handling, and verify the instruction field is populated
- [ ] 3.3 Write the `code-review` instruction in `schema.yaml` — fresh-context requirement, implementation evaluation scope, VERDICT format, and verify the instruction field is populated
- [ ] 3.4 Write the `verify` instruction in `schema.yaml` — spec compliance check, test integrity check, review completeness check, DECISION format, mechanical vs semantic distinction, and verify the instruction field is populated
- [ ] 3.5 Update the `tasks` instruction in `schema.yaml` to enforce TDD ordering (write failing test → implement → refactor) per scenario and verify the instruction field is updated
- [ ] 3.6 Run `openspec schema validate goulart-sdd --verbose` and confirm all artifacts and instructions are valid

## 4. OpenCode Skills

- [ ] 4.1 Create `.opencode/skills/goulart-review/SKILL.md` with instructions for fresh-context adversarial review, finding format, verdict handling, and verify the file exists
- [ ] 4.2 Create `.opencode/skills/goulart-verify/SKILL.md` with instructions for post-implementation audit, spec compliance check, test integrity check, review staleness check, and verify the file exists
- [ ] 4.3 Create `.opencode/commands/goulart-review.md` command file that invokes the goulart-review skill and verify the file exists
- [ ] 4.4 Create `.opencode/commands/goulart-verify.md` command file that invokes the goulart-verify skill and verify the file exists

## 5. Project Configuration

- [ ] 5.1 Update `openspec/config.yaml` with project context describing Goulart SDD, its purpose, and constraints, and verify the file is valid YAML
- [ ] 5.2 Add per-artifact rules to `openspec/config.yaml` for `specs` (require SHALL/MUST, every scenario must be testable) and `tasks` (TDD ordering, small tasks), and verify the config file parses correctly

## 6. Methodology Documentation

- [ ] 6.1 Create `docs/methodology.md` explaining the Goulart SDD workflow, role model, gate categories, TDD approach, and review process, and verify the file exists and contains all major sections
- [ ] 6.2 Create `docs/getting-started.md` with installation steps, quick-start workflow, and first-change walkthrough, and verify the file exists

## 7. Dogfooding Verification

- [ ] 7.1 Create a test change using the `goulart-sdd` schema (`openspec new change "dogfood-test" --schema goulart-sdd`) and verify the change directory is created with the correct schema
- [ ] 7.2 Verify that the test change's `.openspec.yaml` references the `goulart-sdd` schema and that `openspec status` shows all expected artifacts (proposal, specs, design, plan-review, test-plan, tasks, code-review, verify)
- [ ] 7.3 Run `openspec validate` on the test change and confirm no errors, clean up the test change directory after verification
