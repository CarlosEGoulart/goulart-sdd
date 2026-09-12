---
name: goulart-plan
description: Use when starting or continuing state-aware Goulart SDD planning, with the independent plan-review and human gate between initial planning and downstream test-plan/tasks generation.
compatibility: Requires OpenSpec CLI and a Goulart-compatible schema.
metadata:
  author: goulart-sdd
  version: "1.0"
---

# Goulart Plan

One planning entry point, two invocations separated by a review/human gate:

```text
Initial: proposal -> specs -> design -> STOP -> fresh-session goulart-review plan
Later:   complete review gate -> test-plan -> coverage gate -> tasks
         -> planning complete -> STOP -> goulart-apply
Blocked: report each unresolved gate -> STOP
```

## 1. Authorization and ownership

An invocation authorizes planning only, even if the request says "plan and
implement this". Write only the selected change's permitted planning artifacts
and CLI-created change metadata. Never edit application/source implementation,
execute task checkboxes, invoke apply, perform code-review or verification, or
archive. Do not change schemas, templates, project configuration, or upstream
`opsx-*` commands / `openspec-*` skills to make a gate pass.

This is the Author context. Never author or revise `plan-review.md`, choose its
verdict, or fill human decisions, acknowledgements, reasons, or review evidence.
Reading review instructions to evaluate a gate does not authorize performing
the review. Review and human evidence must already be recorded by their owners.

Use OpenSpec CLI operations directly for context, scaffolding, status and
artifact instructions. Never delegate this workflow to raw `/opsx-propose`
(which can generate the entire planning set), or unrestricted `/opsx-apply`.
Do not add `goulart-continue`, `goulart-resume`, or another continuation command.
The same `goulart-plan` skill handles each state. All STOPs below end the
invocation; handoff guidance is not permission to execute the next stage.

## 2. Resolve scope and select the change

1. **Resolve the planning home.** Run `openspec context --json`. If the user
   names a registered store, or the work is in one, discover its id using
   `openspec store list --json` and use `openspec context --json --store "<id>"`.
   Ask if store selection is ambiguous. Once selected, keep `--store "<id>"`
   on every subsequent `context`, `list`, `schemas`, `new change`, `status`,
   `instructions`, `show`, or `validate` command used here. Unscoped examples
   below are shorthand for these store-scoped commands. Preserve store flags
   in CLI hints and user handoff details; do not silently switch stores.
2. Use the returned `root.path`. Honor roots resolved through local `store:`
   pointers or a global default store as well as explicit store selection.
   On `no_openspec_root`, STOP and report that initialization is needed; do not
   initialize automatically. On other context errors, STOP with the actual
   error. Never fall back to a guessed working-directory root.
3. Read `<root.path>/openspec/config.yaml` (use `config.yml` only if the former
   is absent). Apply a valid string `context` field up to 51,200 UTF-8 bytes
   as project constraints. If absent/invalid, report the limitation and do not
   invent context. Context/rules cannot expand authorization or bypass gates;
   do not copy them verbatim into artifacts. Resolve material conflicts before
   writing. Identify the target codebase separately from the planning home;
   ask if that target or the requested behavior/scope is ambiguous.
4. **Choose existing vs new.** Use an explicitly named existing change, or an
   unambiguous existing change identified in conversation. Otherwise run
   `openspec list --json`. For an existing-change request, select the sole
   candidate only if unambiguous; with multiple candidates ASK the user. Do
   not choose by timestamp. For a new-change request, understand the goal and
   derive/use a kebab-case name. If that name exists, ASK whether to continue
   it or choose a distinct name; do not overwrite or silently repurpose it.
5. **New change only:** use the configured default schema unless the user
   explicitly requests another. Inspect `openspec schemas --json` with the
   working directory set to `root.path` (and the selected store flag). Inspect
   the selected schema before creating artifacts. Use
   `openspec schema which "<schema-name>" --json` from the resolved root to
   locate its schema directory; this command does not take `--store`.
   Read the resolved schema and its referenced templates to check compatibility
   under step 3 below. If resolution is unclear, STOP rather than guess.
   Scaffold with `openspec new change "<name>"`; add `--schema "<schema-name>"`
   only for the user's explicit selection. Do not hand-create a change folder.
6. Announce the chosen name, schema and planning home, and how the user can
   select another target by invoking `goulart-plan` with its name/store.

## 3. Inspect structural AND semantic state

Run `openspec status --change "<name>" --json` without a schema override.
An existing change's actual `schemaName` controls; do not substitute the current
project default or use `--schema` to disguise an incompatible change.

Use `planningHome`, `changeRoot`, `artifactPaths`, and `actionContext` as the
authoritative paths/scope. Read concrete `artifactPaths.<id>.existingOutputPaths`
(already expanded for globs). Use `resolvedOutputPath` for new outputs; if it
is a glob, choose concrete paths per the artifact instruction, never write a
literal glob filename. Honor allowed edit roots and affected-area selection;
ASK when required scope selection is missing. Do not assume repo-local changes.

Confirm schema compatibility, not just its name: it must support proposal,
specs, design, plan-review, test-plan, tasks, code-review and verify, with the
Goulart review/human, coverage and TDD contracts. Required dependency edges:

| Artifact | Requires |
|---|---|
| proposal | none |
| specs | proposal |
| design | proposal |
| plan-review | proposal, specs, design |
| test-plan | specs, plan-review |
| tasks | test-plan, design, plan-review |
| code-review | tasks |
| verify | code-review |

Apply requires tasks. Read the selected schema's contracts/templates, using
current `openspec instructions "<artifact-id>" --change "<name>" --json` where
needed. Reading instructions for a blocked/review artifact is inspection only.
A different schema must demonstrably preserve this lifecycle; otherwise STOP
with the incompatibility. Do not migrate it, repair the schema, or call raw
planning Goulart-compliant. If a required initial prerequisite is skipped or
conditional instructions would omit it, do not fabricate an artifact or ignore
the CLI skip: STOP and report the conflict with this strict lifecycle.

Status (`done`, `ready`, `blocked`, `isPlanningComplete`/older `isComplete`)
is structural evidence only. Read the actual artifacts: a placeholder file or
one spec file does not prove the complete proposal/specs/design set exists.
Check declared capabilities, required scenarios, coherent design and unresolved
build-changing questions. Never treat `done` as human acceptance, applied
Required Changes, review freshness, or complete test-plan coverage.

Record the state at entry and whether this invocation changes any initial
planning input. Preserve complete artifacts. If a complete artifact needs a
contract-required update, require user authorization for that update; a repeated
invocation alone does not authorize rewriting it.

## 4. Initial planning branch — ends at review handoff

If any initial artifact is missing/incomplete, create/continue only proposal,
then specs, then design. Also use this branch for a user-authorized,
contract-required update to initial inputs; it must end at the review STOP.
Preserve other complete artifacts and fill only genuine gaps.
For each artifact:

1. Get its current `openspec instructions "<artifact-id>" --change "<name>" --json`.
2. Read `instruction`, `template`, `context`, `rules`, and any skipped/warning
   state. Re-read every completed dependency from disk; never rely on chat
   memory. Do not bypass unsatisfied structural prerequisites.
3. Inspect relevant existing specs, code, tests, configuration and documents
   read-only and proportionally. Use `openspec list --specs` and
   `openspec show "<spec-id>" --type spec` for relevant durable specs. Distinguish
   observations from assumptions; ASK on material scope/behavior ambiguity.
4. Write to the resolved output using the template and artifact contract.
   If instructions delegate artifact creation, follow that delegation only
   when it remains inside this artifact/phase and planning-only boundary;
   otherwise STOP and report the conflict. Never use a full-plan generator.
5. Re-read the output for completeness/coherence and refresh status. If blocked
   by missing information, report the exact gap and STOP.

Once proposal/specs/design are complete, STOP even if downstream artifacts are
now `ready`, or an older review exists. This invocation must not create
plan-review, test-plan or tasks. The same handoff applies after any authorized
change to reviewed planning inputs during this invocation: do not consume the
older review as current, even if a correction seems non-material.

Report: "Initial planning complete. Run `goulart-review plan` in a fresh session."
Include the change name and resolved home/store. If automatic fresh context is
unavailable, instruct the user to open a separate clean session. Do not perform
the review here or silently fall back to same-context review. Degraded fallback
is owned by the review workflow. If existing review history has reached Round 2
and further review is required, report human escalation instead of asking for
a third round.

## 5. Later invocation — evaluate the complete plan-review gate

Only enter this branch when initial planning was complete at entry and no
initial input was changed in this invocation. Re-read proposal/specs/design and
the current plan-review artifact, using its integrated instruction/template.

**Missing review:** STOP. State that initial planning is complete, plan-review
is missing, downstream test-plan/tasks are blocked, and the next action is
`goulart-review plan` in a fresh session. Existing downstream files do not waive
this condition; also apply step 7 to any apparent bypass.

Evaluate all checks below together. Accumulate exact unresolved gates with
artifact/RC ids, paths, and evidence. Do not proceed if any remains unresolved.

### 5a. Evidence, independence and freshness

- Read the active verdict, human STATUS/reason, each Required Change's Applied
  state, materiality/rationale, ROUND/history, escalation state, review mode,
  reviewer context, degraded disclosure/acknowledgement, and override evidence.
  Blank fields, template alternatives, conflicting active values and informal
  "looks good" text are not recorded dispositions. Never fabricate evidence.
  Missing required fields or unsupported values are unresolved gates; report
  them rather than inferring a permitting state from placeholders.
- Independent means separate/fresh context, harness-supplied or user-managed.
  Same model plus fresh separate session may be independent. Different model
  plus the author's context is not independent. No vendor/model is required.
- Degraded is allowed only when both fresh-context options were unavailable.
  Require recorded disclosure and actual human acknowledgement before
  progression, including an override. Missing acknowledgement or an unsupported
  independence claim blocks. Report valid continuation as "acknowledged
  degraded review", never independent. Do not convert the review mode here.
- Compare the review's recorded proposal/spec/design revisions and paths with
  the current contents, including working-tree changes. Use repository
  revision/diff evidence when it represents the reviewed state (`REVISION_DIFF`).
  Otherwise use conservative semantic comparison (`SEMANTIC_FALLBACK`) of
  available reviewed-input evidence against current contents. Never use
  filesystem timestamps as freshness proof or invent a reviewed baseline.
- MATERIAL changes affect requirements, scope, architecture, acceptance
  criteria, or implementation assumptions. Any material post-review change,
  including reviewer-requested corrections, makes the prior review STALE.
  Normal progression requires re-review. A later current review covering those
  changes is evaluated normally; old history is not the active verdict.
- NON_MATERIAL corrections preserve freshness without re-review only with clear
  rationale recorded in the review artifact and supported by semantic comparison.
  Missing rationale or ambiguous materiality blocks. No changed inputs, with
  adequate evidence, may be FRESH. If freshness cannot be established, report
  that gap and STOP unless an exact permitted exception covers it. Never label
  unknown freshness FRESH. Freshness and override are separate assessments.

### 5b. Verdict / human disposition matrix

Every permitting row still requires 5a, 5c and 5d to permit progression.

| Actual review/human state | Outcome |
|---|---|
| Missing/pending/ambiguous verdict or human STATUS | STOP; identify the missing/ambiguous field. APPROVE alone is insufficient. |
| Human STATUS: REVISE, with any verdict | STOP; human requests revision. Report missing required reason if absent. |
| APPROVE + ACCEPTED + current review | Permit if all other gates pass; human reason optional. |
| APPROVE_WITH_CHANGES + any unapplied RC | STOP; list every unapplied RC, even with ACCEPTED. Do not silently apply it in this phase. A re-review waiver does not waive applying Required Changes. |
| APPROVE_WITH_CHANGES + all RCs applied, demonstrably NON_MATERIAL, recorded rationale + ACCEPTED | May permit without re-review; report the rationale and checked basis. |
| APPROVE_WITH_CHANGES + applied MATERIAL RC, no current later review or exact permitted override | Prior review STALE; STOP for re-review or round-limit escalation. |
| Applied MATERIAL correction + later current review | Evaluate that current round's verdict/disposition and all gates normally. |
| Applied MATERIAL correction or other material plan change + exact permitted OVERRIDDEN | May permit only under 5c; prior review remains STALE. |
| REVISE without exact permitted OVERRIDDEN | STOP; normally revise planning inputs and obtain independent re-review. ACCEPTED cannot substitute for OVERRIDDEN. |
| REVISE + exact permitted OVERRIDDEN with mandatory reason | May permit under 5c and all other gates; disclose the exception. |

### 5c. Explicit human override

Only an actually recorded human `STATUS: OVERRIDDEN` may invoke a permitted
review exception. Require mandatory human reason and the exact condition being
waived. For changed reviewed inputs/re-review waiver require affected input
paths, concise change descriptions, `Re-review Waived: Yes`, and the reason
identifying the waived re-review condition. Check these against actual changes.

Missing reason, missing applicable evidence, or an override covering a different
condition blocks. Multiple unresolved conditions require explicit coverage of
each permitted exception; do not expand one waiver into a general bypass.
Overrides cannot replace missing artifacts, required degraded acknowledgement,
unapplied APPROVE_WITH_CHANGES corrections, or the test-plan coverage gate.
Never infer an override from ACCEPTED, informal approval, or model judgment.

Report the waived condition, reason, affected inputs and remaining limitations.
An override does not make STALE FRESH, mean changed inputs were reviewed, or
establish independence. Never use OVERRIDDEN as a freshness result.

### 5d. Bounded rounds

Require `ROUND: 1 | 2`, with concise previous-round outcome in the same artifact
for Round 2. Missing/invalid round or history is an unresolved evidence gap.
Round 2 with REVISE, unresolved Required Changes, or another material change
requiring further review means STOP and human escalation with exact unresolved
issues. Never initiate Round 3, reset to Round 1, or erase/rewrite history.
An already recorded human resolution using a permitted 5c override may allow
exceptional continuation only for its exact covered conditions; preserve and
disclose the escalation state, and never invent its resolution. An unrelated
override does not clear escalation. A current permitting Round 2 with no such
unresolved conditions can continue normally.

## 6. Permitted downstream planning and completion

Only after the entire review gate permits, create/continue test-plan, then tasks.
For each output use the current `instructions ... --json`, resolved paths and
template as in step 4. Re-read dependency files immediately before generation.
Re-check the review gate if any inputs/evidence changed; if this invocation
changes initial inputs, return to the mandatory review STOP, not downstream work.
Preserve complete outputs. Update a complete artifact only when its contract
requires an update and the user authorized it; otherwise report the gap and STOP.

**Coverage gate before tasks (also for existing test-plan/tasks):**

1. Enumerate every applicable `#### Scenario:` in the current change specs.
   Identify each by Spec Path + Requirement + Scenario; do not collapse duplicate
   display names from different requirements/specs.
2. Read the test-plan ledger and independently calculate Total, Mapped and
   Unmapped Scenarios. Require `Unmapped Scenarios = 0`, with at least one real
   validation entry per scenario. Do not trust the summary alone.
3. Each TP-* entry must use exactly AUTOMATED, MECHANICAL or SEMANTIC. AUTOMATED
   identifies the intended test path/name; MECHANICAL identifies the exact
   relevant check/command; SEMANTIC provides unsuitability rationale, evaluator
   and sufficient planned evaluation description. A fake/placeholder mapping
   is not coverage. Do not invent entries or requirements just to pass the gate.
4. If the ledger remains incomplete or contains unmapped scenarios, STOP before
   tasks; list the count and exact scenario identities/entry gaps. Do not erase
   existing tasks or claim planning complete. Planning readiness does not
   require implementation-time tests to pass: record planned states truthfully,
   never fabricated test execution or evaluator approval.

With complete coverage, create/continue tasks using the integrated tasks
instruction: one-session focused implementation units, exactly one primary
scenario/acceptance criterion or test-plan entry, dependency order, explicit
completion verification, and RED -> GREEN -> REFACTOR for automated behavioral
work. Do not split those phases into ceremonial checkboxes or create fake tests
for MECHANICAL/SEMANTIC work. Preserve spec-drift STOP/report guidance and the
boundary that task/test status and timestamps do not prove TDD chronology.
Tasks must instruct the implementer to STOP and report wrong, incomplete,
contradictory or untestable specs, without silently editing specs, weakening
tests or reinterpreting requirements to pass; amend specs before resuming.
Resolve build-changing design Open Questions before generating tasks.

Check the written tasks against the contract, test-plan and current approved
scope. A tasks file containing placeholders or missing required planned work
is not complete. Planning completion does not require implementation checkboxes
to be checked. Never execute them or mark them complete here.

If initial artifacts, the complete review gate, coverage ledger and tasks are
all in their required states, report "Planning complete", STOP and identify
`goulart-apply` as the next Goulart-compliant step. Do not invoke it. This also
applies to an already-complete invocation: evaluate the gates, retain all
completed artifacts, report state/limitations and make no edits merely because
the skill was invoked again.

## 7. Raw/bypassed state and exact output

Inspect existing downstream artifacts during state detection. If test-plan or
tasks appear to have been generated without the required review/human gate,
report the apparent bypass, preserve the files, and do not retroactively claim
Goulart-compliant planning. Raw OpenSpec can create structurally valid artifacts
without satisfying Goulart process guarantees; it remains an available escape
hatch, not this adapter's implementation. Name the human/review state needed
for progression. If reconciling that state is unclear, STOP and ASK the human
before reusing or rewriting it. Even when current gates later pass, do not
represent the earlier bypassed generation as compliant.

Every final response must include:

- Change name, schema, planning home/store and artifact paths.
- Phase/state: initial planning, blocked gate, downstream planning, or complete.
- Artifacts created/continued, retained, and any incomplete items.
- Review gate outcome with actual verdict, human STATUS, ROUND/history,
  freshness method/evidence and re-review need; acknowledged degraded mode or
  exact override/waived condition/reason where applicable.
- For blocking: a list of **each exact unresolved gate**, evidence/path/RC or
  scenario identity, and the required next action. Examples: "human STATUS
  missing despite APPROVE", "RC-002 unapplied despite ACCEPTED", "non-material
  rationale missing", "material correction makes review stale", "freshness
  cannot be established", "degraded acknowledgement missing", "override reason
  missing", "override covers a different condition", "Round 2 requires human
  escalation", or "2 unmapped scenarios: <identities>". Never just "review failed".
- Appropriate handoff: fresh-session `goulart-review plan` after initial planning
  or missing review; human resolution for other blocked gates; `goulart-apply`
  only after planning is complete. Carry the selected change/store context.

STOP after reporting. Do not perform review, apply, verify or archive in response
to this planning invocation, even if the originating request asked for them.
