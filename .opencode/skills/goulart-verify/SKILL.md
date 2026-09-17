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
resolve change → assess ALL safely-evaluable prerequisites → persist prerequisite evidence

if ANY prerequisite blocked:
    STOP → no audit → no DECISION

if all prerequisites pass:
    perform audit → write full verify artifact → emit DECISION → STOP
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
- write the resolved verify artifact to persist prerequisite evidence before
  the audit when the authoritative verify instruction requires it;
- write the full resolved verify artifact after the audit completes.

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

This distinction is mandatory. The prerequisite phase MUST NOT short-circuit
at the first blocked condition. Independently assess every mandatory
prerequisite that can safely be evaluated, record each result, and only STOP
once the prerequisite table is as complete as safely possible.

The flow is:

```text
resolve change
→ independently assess every mandatory prerequisite
→ for each prerequisite:
     if upstream artifact is missing and the prerequisite depends on it:
       record Status: BLOCKED, Blocking Gap: cannot assess because <upstream> is missing
     else:
       assess against actual evidence, record SATISFIED or BLOCKED with evidence
→ persist the complete prerequisite evidence to verify artifact
→ if ANY prerequisite is BLOCKED:
     report ALL blocked prerequisites together
     DO NOT run final verification audit
     DO NOT emit DECISION: FAIL
     DO NOT invent PASS/PASS_WITH_WARNINGS
     STOP
```

Only when ALL prerequisites are SATISFIED:

```text
perform audit
→ write full verify artifact
→ emit final DECISION
```

A prerequisite block is NOT DECISION: FAIL. FAIL means the verification AUDIT
actually ran and found blocking audit findings. Do not collapse these states.

## 4. Prerequisite — code-review exists

Code-review artifact MUST exist at its actual resolved path. Artifact
existence alone is insufficient.

Record Status: SATISFIED if present with evidence, or BLOCKED with Blocking Gap
explaining the absence.

If missing: next action should identify that code review must be completed
through the proper review workflow. Do NOT generate code-review yourself.

## 5. Prerequisite — code-review verdict permits

Consume the ACTUAL approved code-review contract. Do not invent a simplified
verdict mapping.

If code-review artifact is missing: record Status: BLOCKED, Blocking Gap:
cannot assess because code-review artifact is missing. Do NOT invent a verdict
result. Move on to assess other prerequisites that do not depend on code-review.

If code-review artifact exists: validate together:

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

Record Status: SATISFIED if verdict permits, or BLOCKED with evidence and
Blocking Gap if it does not.

## 6. Prerequisite — mandatory finding triage

When code-review findings exist, validate EACH finding's human triage.

If code-review artifact is missing: record Status: BLOCKED, Blocking Gap:
cannot assess because code-review artifact is missing. Move on to assess other
independent prerequisites.

If code-review findings exist: validate EACH finding's human triage.

Allowed triage values from the authoritative code-review contract: ACCEPT,
REJECT, DEFER.

For REJECT or DEFER: mandatory justification must exist.

Do not treat REJECT or DEFER as an implicit override.

If a finding remains blocking under its recorded state: verification is blocked
unless an exact permitted explicit human override covers that condition.

Record Status: SATISFIED if all findings triaged (or no findings), or BLOCKED
with evidence and Blocking Gap if triage is incomplete.

This prerequisite is NON-WAIVABLE by a general review override.

Zero findings: do not require artificial triage or a separate general human
approval.

## 7. Prerequisite — all tasks complete

Read the ACTUAL tasks artifact. All actual implementation task checkboxes must
be complete.

Do not assume code-review existence proves task completion.

Record Status: SATISFIED if all tasks complete with evidence, or BLOCKED with
Blocking Gap listing exact incomplete task(s).

Do NOT check incomplete tasks, implement them, or emit FAIL.

Task completion is mandatory and non-waivable by review override.

## 8. Prerequisite — test-plan final evaluability

Read every REQUIRED test-plan entry. A final evaluable state means:

- AUTOMATED: a completed check is recorded; a result is recorded.
- MECHANICAL: a completed check is recorded; a result is recorded.
- SEMANTIC: a documented evaluation exists.

Record Status: SATISFIED if all required entries are evaluable, or BLOCKED
with Blocking Gap listing the unevaluated entries.

Important: EVALUABLE does NOT mean PASSING. A required entry may be evaluable
and record a failure. That does not block the audit from starting merely
because it is a failure. Instead: prerequisite = evaluable; audit examines the
result; a failing required validation becomes an audit finding and can lead to
FAIL.

Distinction between prerequisite block and audit finding for SEMANTIC entries:
a SEMANTIC entry with NO documented evaluation at all = prerequisite BLOCK (no
audit). A SEMANTIC entry with an evaluation but insufficient evidence =
evaluable prerequisite; the audit will assess evidence sufficiency and may
produce a blocking finding.

Do NOT turn a test failure into an "unevaluable" prerequisite. Do NOT modify
test-plan.

## 9. Prerequisite — degraded review evidence

For ANY applicable review artifact recorded as Degraded, require BOTH:

- disclosed degraded independence limitation;
- actual human acknowledgement.

A degraded review MUST NOT be represented as independent.

Record Status: SATISFIED if disclosure and acknowledgement both exist, or
BLOCKED with Blocking Gap if either is missing.

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

Record Status: SATISFIED if freshness assessed and either FRESH or covered by
permitted override, or BLOCKED with Blocking Gap explaining the evidence gap or
stale condition.

If freshness cannot be established conservatively: record BLOCKED with the
evidence gap. Move on to record any other prerequisite results already
established.

## 10a. Prerequisite — review override (if applicable)

The verify template includes a "Review override (if applicable)" prerequisite
row. Define deterministic status semantics:

**CASE A — no override required/invoked:**

Record Status: SATISFIED. Evidence: "no override applicable". Blocking Gap:
none.

**CASE B — valid applicable override covering the exact stale/blocking
condition:**

Record Status: SATISFIED. Evidence must include all stage-specific evidence:

- actual human-owned override;
- exact waived condition;
- mandatory reason;
- affected input/state where required by the relevant contract;
- applicability to the actual stale/blocking condition.

If review remains stale despite the override: Freshness Result stays STALE.
Override is an exception to the gate, never freshness.

**CASE C — override required/claimed but incomplete or invalid:**

Record Status: BLOCKED. Evidence: existing evidence. Blocking Gap: exact
missing or mismatched evidence.

Do NOT infer override from APPROVE, ACCEPT, REJECT, or DEFER alone.

Override cannot replace:

- code-review existence;
- mandatory triage;
- task completion;
- test-plan evaluability;
- degraded-review acknowledgement.

The override prerequisite row participates in the same aggregate prerequisite
assessment as all other rows.

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
limit: record the relevant prerequisite as BLOCKED with Round Limit State /
escalation evidence. Do NOT invent Round 3. Continue evaluating all other
independent prerequisites. Persist the complete prerequisite table, then STOP.

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
ROUND 1 | 2. Round 2 requiring another review: record the relevant prerequisite
as BLOCKED with escalation evidence. Do NOT create Round 3. Do NOT
immediately STOP the prerequisite phase — continue assessing all other
independent prerequisites. Persist the complete prerequisite table, then STOP.

Do not allow verifier to manufacture a new review round. If an actual human
exception exists, consume it only if the underlying stage contract permits it
and the exception covers the exact condition.

## 15. Only after ALL prerequisites pass — start audit

After the complete prerequisite assessment is finished and ALL mandatory
prerequisites are SATISFIED, persist the prerequisite evidence and perform the
whole-change audit. Before this point: do NOT manufacture a final DECISION.

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

SEMANTIC: independently assess every required SEMANTIC entry. Require:

- documented semantic evaluation exists;
- Semantic Evaluator / actual evaluator evidence is present where defined by
  the test-plan contract (see Semantic Entry Detail);
- evaluation evidence is present;
- the evidence is sufficient to assess the mapped requirement/scenario;
- no human/evaluator result is fabricated.

If semantic evidence demonstrates the required behavior is SATISFIED: record
the semantic validation as passing/satisfied with evidence.

If semantic evidence demonstrates required behavior is UNMET: record a
BLOCKING audit finding. That blocking finding contributes to DECISION: FAIL.

If an evaluation record exists but the supplied evidence is insufficient to
determine whether the mapped required behavior is satisfied: do NOT pretend it
passed. Record the evidence deficiency and treat required behavior as not
successfully verified; classify it as a BLOCKING audit finding.

Do not invent automated tests for semantic/mechanical entries. Do not silently
change validation type from SEMANTIC to AUTOMATED or MECHANICAL. Do not weaken
a failing check.

## 18. Audit — full test suite

Successful verification requires the applicable complete suite to pass at
verification time. Historical CI output, old implementation-time test output,
or code-review assertions DO NOT replace verification-time execution.

**Case A — meaningful applicable complete test-suite command EXISTS:**

Execute it during the audit. Record command, result, evidence.

- If it executes and passes: record command/result/evidence and continue.
- If it executes and fails: record a BLOCKING audit finding. This completed
  verification audit MUST emit DECISION: FAIL. The goulart-verify invocation
  itself MUST NOT convert that FAIL to PASS or PASS_WITH_WARNINGS through a
  human exception. Any later archive-stage human override is outside
  goulart-verify and does NOT retroactively change the recorded verify
  decision or remove the blocking finding from this audit.
- If it exists but CANNOT be executed: record the exact command that should
  have run, record why it could not be executed, record the missing
  verification evidence. Create a BLOCKING audit finding because required
  test integrity could not be established. This completed verification audit
  MUST emit DECISION: FAIL. Do NOT produce PASS or PASS_WITH_WARNINGS when an
  applicable required complete suite exists but its passing result was not
  established during this verification.

**Case B — NO meaningful applicable complete-suite command exists:**

Do NOT invent one. Record that no applicable complete-suite command exists.
Record rationale/evidence supporting that conclusion. Absence of a nonexistent
suite alone is not a failure and does not force FAIL.

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

**Prerequisites Checked:** for each mandatory prerequisite, record Status
(SATISFIED | BLOCKED), Evidence, and Blocking Gap.

**Reviewed State:** Reviewed Revision, Base Revision (when useful for diff
evidence), Timestamp (informational only — MUST NOT be freshness evidence),
Reviewed Paths / Artifacts.

**Task Completion:** Total Tasks, Completed Tasks, Incomplete Tasks, Evidence.

**Spec Compliance:** for each required scenario, record Spec Path, Requirement,
Scenario, Result (SATISFIED | UNMET), Evidence/Finding.

**Test Integrity — Validation Results:** for each required entry, record
Entry, Type (AUTOMATED | MECHANICAL | SEMANTIC), Validation, Result,
Evidence.

**Test Integrity — Full Test Suite:** Command, Result, Evidence.

**Test Integrity — Behavioral Coverage Integrity:** Coverage Reduction
Detected, and for each affected entry: Entry, Change, Coverage Impact, Spec
Amendment, Rationale.

**Review Staleness — Plan Review:** Reviewed Revision, Reviewed Paths,
Changes After Review, Affected Paths, Materiality, Materiality Rationale,
Assessment Method (REVISION_DIFF | SEMANTIC_FALLBACK), Assessment Evidence,
Freshness Result (FRESH | STALE), Re-review Required, Override Invoked,
Condition Waived, Override Reason.

**Review Staleness — Code Review:** Reviewed Revision, Reviewed Paths,
Changes After Review, Affected Paths, Materiality, Materiality Rationale,
Assessment Method, Assessment Evidence, Freshness Result, Re-review Required,
Accepted Finding Caused Material Change, Round Limit State, Override Invoked,
Condition Waived, Override Reason.

**Review Overrides:** for each relevant review: Review, Condition Waived,
Override Invoked, Reason, Freshness Result.

**Scope Drift:** Path/Area, Drift (NONE | DETECTED), Description, Impact
(NON_BLOCKING | BLOCKING).

**Findings / Warnings:** ID, Type (BLOCKING | WARNING), Area, Description,
Evidence.

**Decision:** DECISION (PASS | PASS_WITH_WARNINGS | FAIL).

**Decision Metadata (exact fields required after a completed audit):**

- Reviewed Revision;
- Decision Summary;
- Blocking Finding Count;
- Warning Count;
- Verification Evidence Paths.

The counts must match the actual findings/warnings recorded in the artifact.
Verification Evidence Paths must identify the actual evidence used by the
completed audit. Preserve Base Revision when useful for diff evidence.

For prerequisite BLOCK before the audit: do NOT fabricate final Decision
Metadata that implies a completed audit. Record prerequisite evidence only as
required by the partial blocked artifact.

Ensure plan and code staleness remain distinct, override remains distinct from
freshness, and accepted-finding material code fixes are represented in
code-review lineage.

## 24. Verify output ownership

The verifier's durable methodology write is ONLY the resolved verify artifact.
Do not modify reviewed evidence to make verification pass.

The verifier MAY write prerequisite evidence to the verify artifact before the
audit when the authoritative verify instruction requires it. A partially
populated blocked verify artifact is NOT a completed verify result — it is
persistent prerequisite evidence.

When prerequisite assessment is complete and ANY prerequisite is BLOCKED:
write/update the resolved verify artifact with ALL assessed prerequisite
evidence — both SATISFIED and BLOCKED rows — with Status, Evidence, and
Blocking Gap for each. Do NOT fabricate a final DECISION. Do NOT invent
PASS/PASS_WITH_WARNINGS. STOP.

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

On prerequisite BLOCK: prerequisite evidence has been persisted to the verify
artifact. Report the COMPLETE assessed gate state — not merely the first
encountered failure:

- change;
- store/planning home;
- every blocked prerequisite with its Blocking Gap;
- every satisfied prerequisite;
- dependencies that prevented assessment (e.g. "cannot assess because
  code-review artifact is missing");
- code-review verdict/triage state where available;
- task state;
- test-plan evaluability state;
- plan-review freshness state;
- code-review freshness state;
- degraded evidence;
- override evidence;
- exact next human actions;
- explicitly state: "verification audit did NOT run and no final verification
  DECISION was emitted."

Do not phrase prerequisite blocking as DECISION: FAIL.

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
