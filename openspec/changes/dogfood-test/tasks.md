# Implementation Tasks

## 1. Deterministic Smoke Behavior

- [ ] 1.1 Implement the deterministic `/goulart-dogfood-smoke` command for TP-001 using RED → GREEN → REFACTOR within this single task:
  - RED: create `tests/dogfood-smoke-command.sh` implementing the planned automated test (`dogfood_smoke_returns_deterministic_output`); the test must invoke `opencode run --command goulart-dogfood-smoke`, capture its output, and assert the result contains `dogfood-smoke: OK`; run the test and confirm it fails because the command file does not yet exist;
  - GREEN: create `.opencode/commands/goulart-dogfood-smoke.md` implementing the minimum behavior needed for the automated test to pass; re-run the test and confirm it passes;
  - REFACTOR: improve the implementation and test for clarity while keeping the test green;
  - completion verification: run `tests/dogfood-smoke-command.sh` and confirm it exits 0 with the expected deterministic output.

## 2. No Mutation Verification

- [ ] 2.1 Verify no repository mutation after implementation (TP-002): confirm a clean baseline with `test -z "$(git status --short)"`; invoke the command via `opencode run --command goulart-dogfood-smoke >/dev/null`; verify clean state with `test -z "$(git status --short)"`. If the working tree was not clean before the check, STOP and report rather than producing misleading evidence. Record the mechanical result as completion evidence.

## 3. Temporary Fixture Identification

- [ ] 3.1 Ensure `.opencode/commands/goulart-dogfood-smoke.md` contains a comment or header identifying it as a temporary dogfooding fixture scheduled for removal (TP-003). After adding the marker, re-run `tests/dogfood-smoke-command.sh` to confirm the TP-001 automated validation still passes and the final command-file state remains valid. Perform the planned semantic evaluation: verify the identification is present, clear, and unambiguously marks the fixture as temporary.

## 4. Cleanup Tracking

- [ ] 4.1 Record and verify the discrete cleanup obligation for the temporary fixture (Requirement: Temporary fixture lifecycle). Completion verification: confirm this tasks artifact explicitly records removal of `.opencode/commands/goulart-dogfood-smoke.md` as a discrete cleanup obligation and that cleanup has not been executed as part of the current planning/implementation step. Do not remove the fixture here.

## Spec-Drift Boundary

If implementation reveals that any requirement or scenario is wrong, incomplete, contradictory, materially ambiguous, or untestable:

- STOP;
- report the exact conflict;
- do not silently edit proposal/spec/design from implementer context;
- do not weaken tests or validations to make implementation pass;
- return to the planning/spec lifecycle before resuming implementation.
