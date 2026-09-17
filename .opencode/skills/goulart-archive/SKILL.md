---
name: goulart-archive
description: Use when executing the Goulart SDD archive stage, with verify-result freshness gating, decision routing (PASS/PASS_WITH_WARNINGS/FAIL), human warning disposition enforcement, explicit FAIL-override semantics, upstream delegation, and raw escape hatch preserved but not represented as Goulart-compliant.
compatibility: Requires OpenSpec CLI and a Goulart-compatible schema.
metadata:
  author: goulart-sdd
  version: "1.0"
---

# Goulart Archive

Final lifecycle entry point:

```text
resolve change → locate verify artifact → assess freshness → route on DECISION

if PASS: delegate to OpenSpec archive
if PASS_WITH_WARNINGS: require human warning dispositions → delegate to OpenSpec archive
if FAIL: block archive unless explicit human override with reason → delegate to OpenSpec archive
```

## 1. Authorization and ownership

This is the ARCHIVE context. goulart-archive is the gatekeeper that consumes
the verify decision and delegates to upstream OpenSpec archive.

goulart-archive MUST NOT:

- modify upstream OpenSpec archive behavior;
- modify upstream `openspec-archive-change` skill;
- modify upstream `opsx-archive` command;
- bypass the verify decision;
- silently promote PASS_WITH_WARNINGS to PASS;
- silently suppress a FAIL decision;
- represent raw execution as Goulart-compliant;
- modify verify artifacts to make archive pass.

goulart-archive MAY:

- read all relevant artifacts (verify, code-review, test-plan, tasks, specs);
- require human input (warning dispositions, override reason);
- delegate to upstream OpenSpec archive after gate satisfaction;
- report decision metadata and warnings.

## 2. Resolve OpenSpec scope / store / change

1. Run `openspec context --json`. If a registered store is selected, discover
   its id using `openspec store list --json` and use
   `openspec context --json --store "<id>"`. Ask if store selection is
   ambiguous. Once selected, keep `--store "<id>"` on every subsequent command.

2. Use the returned `root.path`. On `no_openspec_root`, STOP and report that
   initialization is needed.

3. Read `<root.path>/openspec/config.yaml` (or `config.yml`). Project
   context/constraints constrain archive behavior but MUST NOT expand
   authorization or bypass gates.

4. Resolve the change:
   - explicit valid change name if supplied;
   - otherwise `openspec list --json`;
   - if ambiguous, ASK;
   - never select by timestamp.

5. Run `openspec status --change "<name>" --json` without overriding its
   schema. Use actual returned values: `planningHome`, `changeRoot`,
   `artifactPaths`, `actionContext`, `existingOutputPaths`,
   `resolvedOutputPath`.

6. Confirm the change schema is Goulart-compatible. If incompatible: STOP.

## 3. Locate and read verify artifact

Run `openspec instructions verify --change "<name>" --json` to discover the
resolved verify output path. Read the verify artifact at that path.

If the verify artifact does not exist: STOP. Report that verification has not
been completed and that `goulart-verify` must be run before archive.

If the verify artifact exists but contains only prerequisite evidence (partial
blocked artifact — no DECISION emitted): STOP. Report the blocked
prerequisites exactly as goulart-verify reported them. Verification audit did
NOT run and no final verification DECISION was emitted.

## 4. Assess verify-result freshness

The verify decision must represent the current reviewed state. Staleness
detection uses repository revision/diff information where available, falling
back to conservative semantic comparison when clean revision cannot represent
the reviewed working tree. Filesystem timestamps SHALL NOT be used as reliable
freshness evidence.

Record the verify artifact's reviewed revision and reviewed paths. Compare
against current repository state.

**Case A — reviewed revision matches current state or changes are non-material:**

Freshness is satisfied. Proceed to decision routing.

**Case B — material changes occurred after the reviewed revision:**

Record freshness as STALE. STOP. Report that verification is stale and
`goulart-verify` must be re-run before archive. Do NOT proceed to decision
routing.

**Case C — revision cannot be determined (semantic fallback required):**

Compare verify artifact's reviewed paths against current state using semantic
judgment. If material changes are detected: record STALE and STOP. If no
material changes detected: record FRESH and proceed.

Never use timestamps as freshness evidence under any circumstance.

## 5. Decision routing — DECISION: PASS

When the verify artifact records DECISION: PASS:

- All audit checks passed, blocking findings = 0, warnings = 0.
- Archive MAY proceed.
- Delegate to upstream OpenSpec archive (section 9).

## 6. Decision routing — DECISION: PASS_WITH_WARNINGS

When the verify artifact records DECISION: PASS_WITH_WARNINGS:

- All blocking checks passed, blocking findings = 0, warnings >= 1.
- Warnings MUST be surfaced to the human.
- Archive MAY proceed ONLY after the human explicitly accepts or defers each
  warning.
- PASS_WITH_WARNINGS SHALL NOT silently mean PASS.

**Step 6a — Surface warnings:**

List every warning from the verify artifact with its ID, area, description,
and evidence.

**Step 6b — Require human disposition:**

For each warning, require the human to choose one of:

- **ACCEPT** — acknowledge the warning and permit archive to proceed. Record
  the acceptance.
- **DEFER** — acknowledge the warning and permit archive to proceed with the
  warning flagged for later review. Record the deferral.

Warnings without a recorded human disposition block archive.

**Step 6c — Record dispositions:**

Persist the human dispositions for each warning. These dispositions are
informational records — they do not retroactively change PASS_WITH_WARNINGS
to PASS.

**Step 6d — Delegate:**

After all warnings have a recorded disposition: delegate to upstream OpenSpec
archive (section 9).

## 7. Decision routing — DECISION: FAIL

When the verify artifact records DECISION: FAIL:

- The verification audit actually ran and found blocking audit findings.
- Archive SHALL NOT proceed unless the human explicitly overrides with reason.

**Step 7a — Surface blocking findings:**

List every blocking finding from the verify artifact with its ID, area,
description, and evidence.

**Step 7b — Require explicit human override:**

The human MAY instruct archive to proceed despite FAIL. The override MUST
include:

- explicit override evidence (the human clearly stated the intent to override);
- exact condition(s) waived (reference specific finding IDs or descriptions);
- mandatory reason (the human explains why the override is justified).

Without all three elements: archive is BLOCKED. Do NOT infer override from
vague or partial input.

**Step 7c — Record override:**

Persist the override evidence, waived condition(s), and reason. These
override records are permanent — they do not retroactively change FAIL to PASS
or remove the blocking findings from the verify artifact.

**Step 7d — Delegate:**

After a valid override is recorded: delegate to upstream OpenSpec archive
(section 9).

## 8. Raw OpenSpec boundary

Raw OpenSpec execution (direct use of `/opsx-archive`, `openspec archive`) is
available as an escape hatch. Users MAY bypass goulart-archive and archive a
change directly using upstream commands.

Raw execution:

- MAY bypass Goulart execution gates (verify decision, warning dispositions,
  override requirements).
- SHALL NOT be represented as satisfying all Goulart SDD process guarantees.
- SHALL NOT be labeled Goulart-compliant.

goulart-archive MUST NOT disable, modify, or remove the raw escape hatch.

When the user explicitly requests raw archive: inform them that raw execution
does not satisfy Goulart compliance guarantees and proceed if they confirm.

## 9. Delegate to upstream OpenSpec archive

After the verify gate is satisfied (PASS, PASS_WITH_WARNINGS with all
dispositions recorded, or FAIL with valid override recorded):

Delegate to the upstream `openspec-archive-change` skill. Follow the upstream
skill's steps exactly — do not modify its behavior.

The upstream skill handles:
- artifact completion checking (may warn);
- task completion checking (may warn);
- delta spec sync assessment;
- actual archive (move to archive directory);
- completion summary.

goulart-archive does NOT duplicate or replace any of these upstream checks.
goulart-archive is ONLY the verify-decision gate. The upstream skill performs
its own checks independently.

**Store and change context:** pass the resolved store and change context to the
upstream skill (same `--store` flags, same change name). The upstream skill
operates on the same change.

## 10. Decision preservation

The verify decision is permanent. goulart-archive MUST NOT:

- retroactively change PASS → PASS_WITH_WARNINGS or FAIL;
- retroactively change PASS_WITH_WARNINGS → PASS;
- retroactively change FAIL → PASS or PASS_WITH_WARNINGS;
- remove or suppress findings from the verify artifact;
- remove or suppress warnings from the verify artifact.

Human override is an exception to the archive gate — not a modification of the
verify decision itself.

## 11. Reporting

On successful archive delegation:

```text
Archive delegated to upstream OpenSpec archive.

Change: <name>
Verify Decision: <PASS | PASS_WITH_WARNINGS | FAIL>
Override Recorded: <yes with reason | no>
Warnings Dispositions: <ACCEPT/DEFER per warning | N/A>
Upstream archive will complete artifact checks, spec sync, and archive move.
```

On archive blocked:

```text
Archive blocked.

Change: <name>
Verify Decision: <PASS_WITH_WARNINGS | FAIL>
Blocking Condition: <exact description>
Next Action: <required human action>
```

Do NOT report archive as complete until upstream OpenSpec archive confirms
success.

## 12. Idempotency / existing archive

If the change has already been archived (directory moved to archive):

- Report that the change is already archived.
- Do NOT attempt to re-archive.
- Do NOT move files back and re-archive.

If a verify artifact already exists and is fresh: use its decision directly.
Do NOT re-run verification unless the verify result is stale.

## 13. Failure / stop reporting

On any STOP condition: report the COMPLETE gate state — not merely the first
encountered failure:

- change;
- store/planning home;
- verify artifact path (if exists);
- verify decision (if exists);
- verify reviewed revision/paths;
- freshness assessment;
- blocking condition (stale verify, missing verify, blocked prerequisites,
  missing warning dispositions, missing override);
- exact next human actions.

Do not phrase archive blocking as upstream archive failure.

## 14. Success reporting

On completed archive (upstream confirms success):

- change;
- schema used;
- archive location;
- verify decision consumed;
- override recorded (if any);
- warning dispositions (if any);
- spec sync status (from upstream).
