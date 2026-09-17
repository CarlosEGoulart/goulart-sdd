---
name: goulart-verify
description: Use when performing independent final whole-change verification in the Goulart SDD workflow, with prerequisite blocking distinguished from audit failure, two independent review-staleness lineages, full validation audit, and PASS/PASS_WITH_WARNINGS/FAIL decision production.
compatibility: Requires OpenSpec CLI and a Goulart-compatible schema.
metadata:
  author: goulart-sdd
  version: "1.0"
---

# Goulart Verify

Final whole-change verification entry point:

```text
resolve change → check prerequisites

if ANY prerequisite blocked:
    report block → STOP → no DECISION: FAIL

if all prerequisites pass:
    perform audit → produce verify artifact → emit DECISION → STOP
```

## 1. Authorization and ownership

This is the VERIFIER context. The verifier independently re-checks evidence
and produces the final verify artifact.

"Independent check" means:

- do not trust code-review assertions merely because code-review.md exists;
- do not trust task checkboxes without checking their required state;
- do not trust historical test claims as verification-time full-suite evidence;
- independently inspect repository state and authoritative artifacts.

Verifier MAY:

- read all relevant planning/review artifacts;
- inspect source/tests/configuration;
- inspect Git revision/diff history;
- run tests/validation commands;
- perform semantic evaluation;
- write the resolved verify artifact ONLY when verification prerequisites
  permit the audit and a verify result is being produced.

Verifier MUST NOT:

- fix source code;
- edit tests;
- edit implementation configuration;
- edit proposal/specs/design;
- edit plan-review;
- edit code-review;
- edit human triage;
- edit human overrides;
- edit test-plan;
- edit task completion;
- perform another review round;
- archive;
- weaken tests to obtain PASS.

If verification finds a problem: record/report it. Do NOT repair it from
verifier context.

## 2. Resolve OpenSpec scope / store / change

1. Run `openspec context --json`. If a registered store is selected, discover
   its id using `openspec store list --json` and use
   `openspec context --json --store "<id>"`. Ask if store selection is
   ambiguous. Once selected, keep `--store "<id>"` on every subsequent command.

2. Use the returned `root.path`. On `no_openspec_root`, STOP and report that
   initialization is needed.

3. Read `<root.path>/openspec/config.yaml` (or `config.yml`). Project
   context/constraints constrain verification but MUST NOT expand authorization
   or bypass gates.

4. Resolve the change:
   - explicit valid change name if supplied;
   - otherwise `openspec list --json`;
   - if ambiguous, ASK;
   - never select by timestamp.

5. Run `openspec status --change "<name>" --json` without overriding its
   schema. Use actual returned values: `planningHome`, `changeRoot`,
   `artifactPaths`, `actionContext`, `existingOutputPaths`,
   `resolvedOutputPath`.

6. Run `openspec instructions verify --change "<name>" --json`. Use the
   ACTUAL resolved verify output path.

7. Confirm the change schema is Goulart-compatible. If incompatible: STOP.

## 3. Prerequisite blocking is NOT audit failure

This distinction is mandatory.

The flow is:

```text
resolve change
→ inspect prerequisite evidence
→ if ANY mandatory prerequisite is blocked:
     REPORT BLOCK
     DO NOT run final verification audit
     DO NOT emit DECISION: FAIL
     DO NOT invent PASS/PASS_WITH_WARNINGS
     STOP
```

Only when prerequisites permit:

```text
perform audit
→ produce verify artifact
→ emit final DECISION
```

A prerequisite block is NOT DECISION: FAIL. FAIL means the verification AUDIT
actually ran and found blocking audit findings. Do not collapse these states.

## 4. Prerequisite — code-review exists

Code-review artifact MUST exist at its actual resolved path. Artifact
existence alone is insufficient.

If missing: STOP. Report code-review prerequisite blocked. Next action should
identify that code review must be completed through the proper review workflow.
Do NOT generate code-review yourself.

## 5. Prerequisite — code-review verdict permits

Consume the ACTUAL approved code-review contract. Do not invent a simplified
verdict mapping.

Validate together:

- verdict;
- findings;
- Requires Implementation Change;
- Blocking Condition;
- Required Changes;
- human finding triage;
- materiality;
- staleness;
- round state;
- explicit override evidence where applicable.

Important invariants:

- zero findings + APPROVE: no triage is required merely for approval.
- findings: mandatory triage applies under the code-review contract.
- Critical finding: REVISE.
- unresolved substantive Blocking Condition: normal progression blocked.
- APPROVE_WITH_CHANGES: any required implementation change must be addressed
  before verify.
- REVISE: normal progression blocked unless there is an ACTUAL permitted
  explicit human override covering that exact review condition.

Never invent ACCEPT, REJECT, DEFER, justification, override, or override
reason.

If code-review state does not permit verify: STOP as prerequisite blocking.
Do NOT emit FAIL.

## 6. Prerequisite — mandatory finding triage

When code-review findings exist, validate EACH finding's human triage.

Allowed triage values from the authoritative code-review contract: ACCEPT,
REJECT, DEFER.

For REJECT or DEFER: mandatory justification must exist.

Do not treat REJECT or DEFER as an implicit override.

If a finding remains blocking under its recorded state: verification is blocked
unless an exact permitted explicit human override covers that condition.

If any mandatory finding triage is missing/incomplete: STOP.

This prerequisite is NON-WAIVABLE by a general review override.

Zero findings: do not require artificial triage or a separate general human
approval.

## 7. Prerequisite — all tasks complete

Read the ACTUAL tasks artifact. All actual implementation task checkboxes must
be complete.

Do not assume code-review existence proves task completion.

If ANY task remains incomplete: STOP. Report the exact incomplete task(s).
Do NOT check them, implement them, or emit FAIL.

Task completion is mandatory and non-waivable by review override.

## 8. Prerequisite — test-plan final evaluability

Read every REQUIRED test-plan entry. A final evaluable state means:

- AUTOMATED: a completed check is recorded; a result is recorded.
- MECHANICAL: a completed check is recorded; a result is recorded.
- SEMANTIC: a documented evaluation exists.

Pending or unevaluated required entries: prerequisite BLOCK → STOP.

Important: EVALUABLE does NOT mean PASSING. A required entry may be evaluable
and record a failure. That does not block the audit from starting merely
because it is a failure. Instead: prerequisite = evaluable; audit examines the
result; a failing required validation becomes an audit finding and can lead to
FAIL.

Do NOT turn a test failure into an "unevaluable" prerequisite. Do NOT modify
test-plan.

## 9. Prerequisite — degraded review evidence

For ANY applicable review artifact recorded as Degraded, require BOTH:

- disclosed degraded independence limitation;
- actual human acknowledgement.

A degraded review MUST NOT be represented as independent.

Missing disclosure or acknowledgement: prerequisite BLOCK → STOP.

Do not change review mode from verifier context.

## 10. Prerequisite — review freshness / exceptions

Assess BOTH review lineages independently. Never create one global freshness
flag.

**Lineage A — PLAN REVIEW:** reviewed state = proposal, specs, design.

**Lineage B — CODE REVIEW:** reviewed state = source, tests,
implementation-relevant configuration.

Use repository revision/diff information where available. If a clean revision
cannot faithfully represent the reviewed working tree, use conservative
SEMANTIC_FALLBACK.

NEVER use filesystem timestamps as freshness evidence.

If freshness cannot be established conservatively: STOP and report the evidence
gap.

## 11. Plan-review staleness lineage

Compare current proposal/specs/design against the state recorded as reviewed
by plan-review.

Material changes after plan-review → plan-review = STALE.

This includes changes requested by the previous reviewer, architecture
changes, requirements changes, scope changes, acceptance-criteria changes,
implementation-assumption changes.

Normal progression requires re-review. A permitted plan-review human override
may waive the exact required re-review condition ONLY when its stage-specific
evidence is complete.

The exception MUST NOT change STALE to FRESH, imply modified inputs were
reviewed, or imply independent review occurred.

Non-material correction may avoid re-review only when its rationale is
recorded under the plan-review contract. Verifier must semantically assess
that rationale.

## 12. Code-review staleness lineage

Compare current source/tests/implementation-relevant configuration against the
state actually reviewed by code-review.

Material implementation changes after code-review → code-review = STALE.

Critically: a material implementation change made to FIX an ACCEPTED
code-review finding still makes the PRIOR code-review stale. Human acceptance
of the finding does NOT mean the modified implementation has been reviewed.

Normal progression requires a new code-review unless an exact permitted
code-review override covers the required condition. At the bounded round
limit: STOP/escalate. Do NOT invent Round 3.

Non-material post-review implementation corrections may avoid re-review only
with clear recorded justification. Verifier must assess that justification
semantically.

## 13. Review overrides

Consume plan and code override evidence according to THEIR OWN contracts. Do
NOT invent a shared fake override format if the stage contracts differ.

For any consumed override require at minimum:

- actual human-owned override evidence;
- exact condition waived;
- mandatory reason;
- applicability to the actual stale/blocking condition.

If the relevant contract requires affected inputs/state: validate them.

Multiple unresolved conditions must be explicitly covered. One override is not
a blanket waiver.

Overrides MUST NOT replace:

- missing code-review;
- incomplete tasks;
- incomplete mandatory finding triage;
- unevaluable required test-plan entries;
- required degraded-review acknowledgement.

An override does NOT make STALE → FRESH, establish independence, or prove
changed implementation/planning state was reviewed. Record the waived condition
separately from freshness.

## 14. Review round / escalation evidence

Where plan-review or code-review bounded round state is relevant, respect
ROUND 1 | 2. Round 2 requiring another review → STOP/escalate, no automatic
Round 3, no history reset.

Do not allow verifier to manufacture a new review round. If an actual human
exception exists, consume it only if the underlying stage contract permits it
and the exception covers the exact condition.

## 15. Only after prerequisites pass — start audit

After ALL mandatory prerequisites permit verification, perform the whole-
change audit. Before this point: do NOT manufacture a final DECISION.

## 16. Audit — spec compliance

Read ALL approved specification requirements and scenarios applicable to the
change. For every required scenario, compare actual
implementation/behavior/evidence against it. Record:

- spec path;
- requirement;
- scenario;
- result: SATISFIED | UNMET;
- evidence/finding.

Do not sample only a subset. Any unmet required scenario is an audit finding.
Determine blocking impact from the actual contract and evidence.

## 17. Audit — validation results

Evaluate every required test-plan entry.

AUTOMATED: execute/confirm the required validation as applicable; required
failures are blocking audit findings.

MECHANICAL: perform/confirm the mechanical check; required failures are
blocking audit findings.

SEMANTIC: perform the required documented semantic evaluation; record evidence.

Do not invent automated tests for semantic/mechanical entries. Do not silently
change validation type. Do not weaken a failing check.

## 18. Audit — full test suite

If the project/change has a meaningful applicable complete test-suite
command: RUN IT DURING VERIFICATION. Record command, result, evidence.

Historical CI output, old implementation-time test output, or code-review
assertions DO NOT replace verification-time execution.

If no meaningful complete-suite command exists: do NOT invent one. Record that
no meaningful complete-suite command exists and supporting rationale/evidence.

If a meaningful complete suite exists but cannot be executed: do not falsely
report PASS. Record the actual limitation and classify its impact.

## 19. Audit — behavioral coverage integrity

Check whether required tests/validations were removed, weakened, skipped, or
replaced in a way that reduces required behavioral coverage. Compare against
relevant spec/test-plan/repository history.

A coverage reduction without a corresponding justified specification change is
a BLOCKING audit finding. Do not equate a green suite with preserved coverage
automatically.

## 20. Audit — scope drift

Inspect implementation scope against approved specs/design/tasks. Record drift
as NONE or DETECTED. When detected, classify impact as NON_BLOCKING or
BLOCKING. Blocking scope drift contributes to DECISION: FAIL.

Do not modify scope from verifier context.

## 21. Findings and warnings

Separate BLOCKING findings from WARNING items. Use stable IDs for warnings
(W-001, W-002). Do not downgrade a blocking issue into a warning. Do not
upgrade harmless informational evidence into a warning.

## 22. Final decision — exact semantics

Emit EXACTLY ONE decision, only after the audit actually ran:

**DECISION: PASS** — ONLY when all required audit checks pass, blocking
findings = 0, warnings = 0. PASS MUST NOT contain unresolved warnings.

**DECISION: PASS_WITH_WARNINGS** — when all blocking checks pass, blocking
findings = 0, one or more genuine non-blocking warnings exist. Warnings must
be surfaced. Human acceptance/deferment of warnings belongs to later archive
eligibility. It does NOT retroactively change PASS_WITH_WARNINGS → PASS.

**DECISION: FAIL** — when the audit actually ran AND one or more blocking
audit findings exist. FAIL means verification discovered blocking problems
DURING the audit. FAIL is NOT the state for missing prerequisite evidence.
Archive must not normally proceed from FAIL.

## 23. Verify artifact content

Write the verify artifact using the actual resolved verify output path and the
approved verify template/contract. Record at minimum:

- Prerequisites Checked;
- Reviewed State / Revision / Paths;
- Task Completion;
- Spec Compliance;
- Test Integrity / Validation Results;
- Full Test Suite;
- Behavioral Coverage Integrity;
- Plan Review Staleness;
- Code Review Staleness;
- Review Overrides;
- Scope Drift;
- Findings / Warnings;
- DECISION;
- Decision Summary / Metadata.

Timestamp is informational ONLY. Never use timestamp as freshness evidence.

Ensure plan and code staleness remain distinct, override remains distinct from
freshness, and accepted-finding material code fixes are represented in
code-review lineage.

## 24. Verify output ownership

The verifier's durable methodology write is ONLY the resolved verify artifact.
Do not modify reviewed evidence to make verification pass.

If a prerequisite block occurs before verification: do not fabricate a
completed verify result merely to record the block. Report the block to the
user and STOP.

## 25. Raw OpenSpec boundary

Raw OpenSpec execution may have bypassed Goulart gates. Artifact existence
alone does not establish Goulart compliance. goulart-verify must check actual
evidence. If required Goulart guarantees are absent: block where required. Do
not retroactively label raw execution as Goulart-compliant. Do not modify or
disable upstream OpenSpec commands.

## 26. Idempotency / existing verify artifact

If a verify artifact already exists: do not assume it is current. Compare its
reviewed revision/paths to current state. If current repository state changed
materially: re-run verification. If an existing verify result accurately
represents the exact current reviewed state and the user has not requested a
new audit: report the current state. Never use an old PASS as permission after
relevant repository changes.

## 27. Failure / stop reporting

On prerequisite BLOCK report:

- change;
- store/planning home;
- exact blocked prerequisite(s);
- code-review verdict/triage state;
- incomplete tasks, if any;
- unevaluable test-plan entries, if any;
- plan-review freshness state;
- code-review freshness state;
- relevant override evidence;
- degraded-review evidence;
- exact next human action;
- explicitly say "verification audit did NOT run and no DECISION: FAIL was
  emitted."

On audit FAIL report:

- audit DID run;
- blocking finding IDs;
- failed spec/validation/test evidence;
- reviewed revision/paths;
- DECISION: FAIL;
- required next action.

## 28. Success reporting

On completed audit report:

- change;
- store/planning home;
- reviewed revision;
- reviewed paths;
- prerequisite results;
- spec scenario coverage;
- validation results by type;
- full-suite command/result or no-suite rationale;
- behavioral coverage assessment;
- plan-review freshness;
- code-review freshness;
- overrides consumed;
- scope drift;
- blocking findings;
- warnings;
- verify artifact path;
- exact DECISION;
- next lifecycle action.

For PASS: next action may be goulart-archive. For PASS_WITH_WARNINGS:
warnings must receive later human disposition. For FAIL: archive normally
blocked. Do NOT invoke archive automatically. STOP after verification result.
