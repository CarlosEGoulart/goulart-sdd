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
goulart-apply  →  select first eligible task → load context → implement →
                  RED → GREEN → REFACTOR (automated) or applicable evidence →
                  mark one task complete → report → STOP

all complete   →  STOP → instruct: run goulart-review code in fresh session
```

## 1. Authorization and ownership

This is the IMPLEMENTER context. The implementer executes exactly one task,
records evidence, and STOPs.

MAY:

- inspect repository files;
- modify implementation/source files;
- modify tests needed for the selected task;
- run task-relevant validation/tests;
- update ONLY the selected task checkbox after successful evidence.

MUST NOT:

- author or revise proposal.md, specs/, design.md;
- modify plan-review.md;
- perform plan review;
- modify human review decisions;
- modify test-plan.md merely to make implementation easier;
- weaken requirements/tests to get green;
- perform code-review;
- perform verify;
- archive;
- execute more than one implementation task per invocation;
- invoke goulart-plan, goulart-review, goulart-verify, goulart-archive;
- change schemas, templates, project configuration;
- modify upstream `opsx-*` commands / `openspec-*` skills.

If implementation reveals planning/specification drift:

STOP and report it.

Do not repair the specification from implementer context.

## 2. Resolve scope and select the change

Follow current integrated root/store conventions from goulart-plan and
goulart-review. Use actual CLI behavior.

1. Run `openspec context --json`. If the user names a registered store, or the
   work is in one, discover its id using `openspec store list --json` and use
   `openspec context --json --store "<id>"`. Ask if store selection is ambiguous.
   Once selected, keep `--store "<id>"` on every subsequent `context`, `list`,
   `status`, `instructions`, `show`, or `validate` command.
2. Use the returned `root.path`. On `no_openspec_root`, STOP and report that
   initialization is needed. On other context errors, STOP with the actual error.
3. If store/change selection is ambiguous: ASK.
4. If schema is not Goulart-compatible: STOP and report incompatibility.

Do NOT blindly hardcode `./openspec/changes/<name>/`. Use resolved values such as
`planningHome`, `changeRoot`, `artifactPaths`, `resolvedOutputPath`,
`existingOutputPaths`, `actionContext`.

## 3. Pre-implementation gate

Before selecting or modifying ANY implementation task, validate the actual
plan gate. Read the current plan-review artifact and its recorded state.

### 3a. Required planning artifacts

All of the following must exist:

- proposal.md
- specs (at least one spec file)
- design.md
- plan-review.md
- test-plan.md
- tasks.md

Artifact existence alone is NOT sufficient.

### 3b. Plan-review verdict and human disposition

Evaluate the current plan-review contract and actual artifact evidence.
Normal implementation may proceed only when the plan gate permits it.

**Permitting states:**

- APPROVE + actual human STATUS: ACCEPTED + current/non-stale review + degraded
  acknowledgement when applicable → may proceed.

- APPROVE_WITH_CHANGES + all applied NON_MATERIAL changes + recorded rationale +
  permitted human acceptance → may proceed if all other gates pass.

- REVISE + exact permitted human override with mandatory reason → may proceed
  under the override rules; prior review remains STALE.

**Blocking states:**

- plan-review.md missing → STOP. Report that plan-review is missing, downstream
  implementation is blocked, and the next action is `goulart-review plan` in a
  fresh session.

- APPROVE + human STATUS missing/pending → STOP.

- APPROVE_WITH_CHANGES + any unapplied Required Change → STOP. List every
  unapplied RC.

- APPROVE_WITH_CHANGES + applied MATERIAL change + no current re-review and
  no exact permitted human override → STOP because previous review is stale.

- REVISE without an exact permitted human override with mandatory reason → STOP.

- Degraded review without required human acknowledgement → STOP.

- Stale review without the exact contractually permitted resolution → STOP.

- Human STATUS missing/pending → STOP.

### 3c. Do NOT fabricate

Never fabricate:

- ACCEPTED;
- OVERRIDDEN;
- reasons;
- degraded acknowledgement;
- freshness.

If a gate is unresolved:

report the exact unresolved condition and STOP before implementation.

### 3d. Raw OpenSpec boundary

Raw OpenSpec artifacts may exist. Do not assume they satisfy Goulart gates
merely because all files exist. If the change appears to have bypassed Goulart
sequencing, evaluate actual evidence. If required Goulart guarantees are absent,
STOP and report them. Do not retroactively call raw execution Goulart-compliant.

## 4. Task selection

The portable baseline is EXACTLY ONE task per invocation.

Read the actual tasks artifact from disk (use resolved output path).

**If all tasks are complete:**

- do not modify source;
- do not create a fake task;
- do not run code-review yourself.

Report implementation complete.

Then instruct:

> Run `goulart-review code` in a FRESH/SEPARATE reviewer session.

STOP.

**If pending tasks remain:**

select the FIRST ELIGIBLE pending task in the task artifact's intended order.
Do not cherry-pick an easier later task merely to make progress.

Eligibility requires that the task's required planning/context dependencies are
available and that the task is not blocked by a known unresolved
specification/dependency condition.

If the first expected task is blocked and selecting a later task would violate
the documented ordering/dependencies:

STOP and report the blocker.

Do not execute multiple tasks in parallel. Do not loop to the next task after
completion.

## 5. Load focused task context

For the selected task, load at minimum:

- exact task description;
- relevant spec requirement(s);
- relevant spec scenario(s);
- relevant design decisions;
- relevant test-plan entry/entries).

The implementer may inspect ANY repository files needed to understand:

- existing architecture;
- dependencies;
- calling code;
- test conventions;
- configuration;
- integration points.

"Fresh/minimal context" means minimizing irrelevant conversational history. It
does NOT mean hiding the repository from the implementer.

## 6. Test-plan alignment

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

## 7. Automated tasks — RED

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

## 8. Automated tasks — GREEN

After valid RED evidence:

implement the minimum behavior needed to satisfy the selected task/scenario.

Then execute the relevant test.

Require GREEN.

Do not broaden implementation to unrelated tasks. Do not opportunistically
implement the next checkbox.

## 9. Automated tasks — REFACTOR

After GREEN:

perform only useful task-local cleanup/refactor.

Keep behavior unchanged.

Run the relevant test again.

Where reasonably applicable, run the smallest meaningful surrounding suite to
detect regressions.

The task must remain green.

## 10. Mechanical / semantic tasks

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

## 11. Bounded retry

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

## 12. Spec drift

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

## 13. Task completion

Mark ONLY the selected task `[x]` after its required evidence succeeds.

Never mark:

- multiple tasks;
- parent groups as a substitute;
- a blocked task;
- a task whose required validation is still failing.

Task checkbox state is NOT historical proof of RED/GREEN chronology. Do not
claim otherwise.

## 14. One-task hard stop

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

## 15. Final task handoff

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

## 16. Idempotent / safe re-invocation

If invoked again after a task has already been checked:

do not re-run that completed task merely because conversation history mentions it.

Re-read actual tasks state. Select the current first eligible unchecked task.

If all are complete: use the final review handoff.

If repository evidence suggests an unchecked task may already have been partly
implemented:

do not blindly mark it complete. Evaluate it as the selected task against
current spec/test-plan evidence.

## 17. Failure / stop reporting

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

## 18. Success reporting

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
