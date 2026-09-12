---
name: goulart-apply
description: Use when executing exactly one implementation task per invocation in the Goulart SDD workflow, with pre-implementation gate enforcement, RED-GREEN-REFACTOR where automated testing is appropriate, mechanical/semantic evidence otherwise, bounded retry, spec-drift STOP, and fresh-session code-review handoff when all tasks complete.
compatibility: Requires OpenSpec CLI and a Goulart-compatible schema.
metadata:
  author: goulart-sdd
  version: "1.0"
---

# Goulart Apply

Single-task implementation entry point:

```text
resolve change → read tasks state

if all complete:
    report completion → STOP → instruct: run goulart-review code in fresh session

if pending tasks exist:
    validate pre-implementation gate → if blocked → STOP
    select first eligible task → load context → implement →
    RED → GREEN → REFACTOR (automated) or applicable evidence →
    mark one task complete → report → STOP
```

## 1. Authorization and ownership

This is the IMPLEMENTER context. The implementer executes exactly one task,
records evidence, and STOPs.

MAY:

- inspect repository files;
- modify implementation/source files;
- modify tests needed for the selected task;
- run task-relevant validation/tests;
- update ONLY the selected task checkbox after successful evidence;
- change implementation-relevant configuration when required by the selected
  task/spec/design (application config, dependency manifests, build config,
  runtime config, task-scoped CI/test config). These are implementation state
  and may later affect code-review staleness. Do not modify configuration
  unrelated to the selected task.

MUST NOT:

- author or revise proposal.md, specs/, design.md;
- modify plan-review.md;
- perform plan review;
- modify human review decisions;
- modify test-plan.md — test-plan is planning-owned and READ-ONLY during
  goulart-apply. If the selected task reveals the test-plan is wrong, missing,
  or unevaluable, STOP and return to planning;
- weaken requirements/tests to get green;
- perform code-review;
- perform verify;
- archive;
- execute more than one implementation task per invocation;
- invoke goulart-plan, goulart-review, goulart-verify, goulart-archive;
- change Goulart schema or template contracts as a way to bypass gates;
- modify upstream `opsx-*` commands / `openspec-*` skills.

If implementation reveals planning/specification drift:

STOP and report it.

Do not repair the specification from implementer context.

## 2. Resolve scope and select the change

1. Run `openspec context --json`. If the user names a registered store, or the
   work is in one, discover its id using `openspec store list --json` and use
   `openspec context --json --store "<id>"`. Ask if store selection is ambiguous.
   Once selected, keep `--store "<id>"` on every subsequent `context`, `list`,
   `status`, `instructions`, `show`, or `validate` command.

2. Use the returned `root.path`. On `no_openspec_root`, STOP and report that
   initialization is needed. On other context errors, STOP with the actual error.

3. Read `<root.path>/openspec/config.yaml` (use `config.yml` only if the former
   is absent). Apply valid project `context` constraints without allowing them
   to expand authorization or bypass methodology gates.

4. Resolve the existing change:
   - explicit named change if valid;
   - otherwise `openspec list --json`;
   - if ambiguous, ASK;
   - never choose by timestamp.

5. Run `openspec status --change "<name>" --json` without overriding its schema.
   Use returned: `planningHome`, `changeRoot`, `artifactPaths`, `actionContext`.
   Use `existingOutputPaths` and `resolvedOutputPath` rather than guessed
   repository paths.

6. Run `openspec instructions tasks --change "<name>" --json`. Use the actual
   tasks instructions and resolved task output path. When evaluating related
   artifacts, use their actual resolved paths from `artifactPaths`.

7. Confirm actual schema compatibility with the Goulart lifecycle. If
   incompatible: STOP. Do not silently substitute the project's current default
   schema.

## 3. Read tasks state

Read the actual tasks artifact from disk (use resolved output path).

**If all tasks are complete:**

- do NOT require the pre-implementation gate merely to perform a no-op;
- do NOT modify source;
- do NOT modify tests;
- do NOT modify task state;
- report that implementation is complete;
- instruct the user to run:

  > Run `goulart-review code` in a fresh/separate reviewer session.

- STOP.

Only when at least one implementation task is still pending should the skill
enforce the full pre-implementation gate before selecting/executing a task.

## 4. Pre-implementation gate

Before selecting or modifying ANY implementation task, validate the actual
plan gate. Read the current plan-review artifact and its recorded state.

### 4a. Required planning artifacts

All of the following must exist:

- proposal.md
- specs (at least one spec file)
- design.md
- plan-review.md
- test-plan.md
- tasks.md

Artifact existence alone is NOT sufficient.

### 4b. Plan-review verdict and human disposition

Evaluate the current plan-review contract and actual artifact evidence.
Normal implementation may proceed only when the plan gate permits it.

**Permitting states:**

- APPROVE + actual human STATUS: ACCEPTED + current/non-stale review + degraded
  acknowledgement when applicable → may proceed if all other gates pass.

- APPROVE_WITH_CHANGES + all applied NON_MATERIAL changes + recorded materiality
  rationale + STATUS: ACCEPTED → may proceed if all other gates pass.

- REVISE + exact human STATUS: OVERRIDDEN + mandatory reason identifying the
  waived condition → may proceed only under the exact override; freshness is
  assessed independently (section 4d).

**Blocking states:**

- plan-review.md missing → STOP. Report that plan-review is missing, downstream
  implementation is blocked, and the next action is `goulart-review plan` in a
  fresh session.

- Human STATUS missing/pending/ambiguous → STOP. Report incomplete human
  disposition evidence.

- Human STATUS: REVISE without Human Reason → STOP. Report incomplete human
  disposition evidence.

- Human STATUS: REVISE with mandatory Human Reason → STOP. Human requested
  revision. Do not implement.

- APPROVE + human STATUS missing/pending → STOP.

- APPROVE_WITH_CHANGES + any unapplied Required Change → STOP. List every
  unapplied RC.

- APPROVE_WITH_CHANGES + applied MATERIAL change + no current re-review and
  no exact permitted human override → STOP because previous review is stale.

- REVISE without an exact permitted human STATUS: OVERRIDDEN with mandatory
  reason → STOP.

- Degraded review without required human acknowledgement → STOP.

- Stale review without the exact contractually permitted resolution → STOP.

### 4c. Do NOT fabricate

Never fabricate:

- ACCEPTED;
- OVERRIDDEN;
- reasons;
- degraded acknowledgement;
- freshness.

If a gate is unresolved:

report the exact unresolved condition and STOP before implementation.

### 4d. Freshness evidence

Determine CURRENT vs STALE using the plan-review artifact's recorded reviewed
revision and input paths.

Compare current proposal/spec/design state against that reviewed state using
revision/diff evidence where available.

- Material change to reviewed planning inputs after review → STALE.
- Non-material corrections may preserve continuation only under the integrated
  plan-review contract and recorded rationale.
- If freshness cannot be established: do NOT assume FRESH. STOP and report the
  evidence gap.
- Never use filesystem timestamps as freshness proof.

An override does NOT make a stale review fresh. An override does NOT mean
changed planning inputs were reviewed. An override itself does NOT automatically
make a current review stale. Freshness is determined by whether inputs
materially changed, not by whether an override exists.

### 4e. Raw OpenSpec boundary

Raw OpenSpec artifacts may exist. Do not assume they satisfy Goulart gates
merely because all files exist. If the change appears to have bypassed Goulart
sequencing, evaluate actual evidence. If required Goulart guarantees are absent,
STOP and report them. Do not retroactively call raw execution Goulart-compliant.

## 5. Task selection

The portable baseline is EXACTLY ONE task per invocation.

Select the FIRST ELIGIBLE pending task in the task artifact's intended order.
Do not cherry-pick an easier later task merely to make progress.

Eligibility requires that the task's required planning/context dependencies are
available and that the task is not blocked by a known unresolved
specification/dependency condition.

If the first expected task is blocked and selecting a later task would violate
the documented ordering/dependencies:

STOP and report the blocker.

Do not execute multiple tasks in parallel. Do not loop to the next task after
completion.

## 6. Load focused task context

For the selected task, load at minimum:

- exact task description;
- relevant spec requirement(s);
- relevant spec scenario(s);
- relevant design decisions;
- relevant test-plan entry/entries.

The implementer may inspect ANY repository files needed to understand:

- existing architecture;
- dependencies;
- calling code;
- test conventions;
- configuration;
- integration points.

"Fresh/minimal context" means minimizing irrelevant conversational history. It
does NOT mean hiding the repository from the implementer.

## 7. Test-plan alignment

Use the actual test-plan classification for the selected behavior.

Possible validation types:

- AUTOMATED
- MECHANICAL
- SEMANTIC

Do NOT invent a fake automated test for mechanical or semantic-only work. Do NOT
silently change the validation type.

If the test-plan entry required for the selected task is missing, contradictory,
or unevaluable:

STOP.

Report the planning/test-plan problem. Do not modify test-plan from implementer
context merely to proceed.

## 8. Automated tasks — RED

Where automated testing is appropriate:

RED must happen FIRST.

Before implementation:

- identify the relevant automated test location/convention;
- write or update the smallest meaningful failing test;
- execute the relevant test;
- confirm it FAILS for the expected missing/incorrect behavior.

A failure caused only by:

- syntax error;
- wrong path;
- broken fixture;
- missing dependency unrelated to the requirement;
- malformed test setup

is NOT acceptable RED evidence.

Correct test setup first if appropriate, while still avoiding implementation.

If you cannot establish a meaningful RED state:

STOP and report why.

Do NOT implement first and retroactively claim RED.

## 9. Automated tasks — GREEN

After valid RED evidence:

implement the minimum behavior needed to satisfy the selected task/scenario.

Then execute the relevant test.

Require GREEN.

Do not broaden implementation to unrelated tasks. Do not opportunistically
implement the next checkbox.

## 10. Automated tasks — REFACTOR

After GREEN:

perform only useful task-local cleanup/refactor.

Keep behavior unchanged.

Run the relevant test again.

Where reasonably applicable, run the smallest meaningful surrounding suite to
detect regressions.

The task must remain green.

## 11. Mechanical / semantic tasks

When automated TDD is not appropriate:

do NOT invent meaningless tests.

Use the applicable evidence defined by the task/test-plan.

**MECHANICAL:**

- file exists;
- command succeeds;
- schema validates;
- configuration parses;
- expected path/content exists.

**SEMANTIC:**

- explicit structured inspection against contract;
- documented manual evaluation;
- consistency check against spec/design.

Still follow:

implement → validate → only then mark task complete.

Do not claim RED/GREEN chronology for non-automated work.

## 12. Bounded retry

If the SAME test fails twice consecutively for the SAME root cause:

STOP the task.

Do NOT attempt a third implementation approach in the same invocation.

Report:

- selected task;
- failing test;
- root cause;
- attempts made;
- current repository state;
- exact human action needed.

Do not mark the task complete.

## 13. Spec drift

If implementation reveals a requirement or scenario is:

- wrong;
- incomplete;
- contradictory;
- untestable;
- incompatible with required design behavior;

STOP.

Do NOT:

- weaken the test;
- reinterpret the requirement silently;
- edit the spec;
- edit design;
- mark the task complete.

Report the exact conflict. Planning/specification must be amended through the
appropriate planning/review workflow before implementation resumes.

## 14. Task completion

Mark ONLY the selected task `[x]` after its required evidence succeeds.

Never mark:

- multiple tasks;
- parent groups as a substitute;
- a blocked task;
- a task whose required validation is still failing.

Task checkbox state is NOT historical proof of RED/GREEN chronology. Do not
claim otherwise.

## 15. One-task hard stop

After one task completes:

report result.

Then STOP.

Even if:

- the next task is trivial;
- there is lots of context budget;
- the model offers to continue;
- tests are already open;
- another task touches the same file.

Never process Task N+1 in the same goulart-apply invocation.

The user must invoke goulart-apply again for the next task.

## 16. Final task handoff

If completing the selected task means ALL tasks are now complete:

report that implementation is complete.

STOP.

Instruct the user to run:

> Run `goulart-review code` in a fresh/separate reviewer session.

Do NOT:

- perform code-review yourself;
- automatically invoke goulart-review;
- verify;
- archive.

## 17. Idempotent / safe re-invocation

If invoked again after a task has already been checked:

do not re-run that completed task merely because conversation history mentions it.

Re-read actual tasks state. Select the current first eligible unchecked task.

If all are complete: use the final review handoff.

If repository evidence suggests an unchecked task may already have been partly
implemented:

do not blindly mark it complete. Evaluate it as the selected task against
current spec/test-plan evidence.

## 18. Failure / stop reporting

On any blocking gate or implementation STOP, report:

- change name;
- schema/store/planning home;
- selected task (if one was selected);
- gate state;
- relevant plan-review state;
- relevant test-plan state;
- tests/validation attempted;
- blocking condition;
- files modified, if any;
- exact next human action.

Do not hide partial changes. Do not mark a failed task complete.

## 19. Success reporting

After a successful single task report:

- change name;
- schema/store/planning home;
- selected task ID and text;
- relevant spec/scenario;
- validation type;
- RED evidence (if automated);
- GREEN evidence (if automated);
- REFACTOR validation (if automated);
- mechanical/semantic evidence (where applicable);
- tests/commands executed;
- files changed;
- task checkbox updated;
- remaining pending task count;
- next action.

If more tasks remain:

> Next action: invoke goulart-apply again in a fresh/minimal implementer
> context.

If no tasks remain:

> Next action: run goulart-review code in a fresh reviewer session.

Then STOP.
