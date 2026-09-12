---
description: "Archive a completed Goulart SDD change after verify passes"
---

Use the `goulart-archive` skill as the authoritative workflow.

**Archive boundary**: This command authorizes archival only, even if the request says "archive and clean up". It never edits proposal, specs, design, test-plan, tasks, plan-review, code-review, verify, or implementation files; never invokes plan, review, apply, or verify.

**Input**: Optionally specify a change name after `/goulart-archive` (e.g., `/goulart-archive add-auth`). If omitted, ask the user what they want to archive.
**Provided arguments**: $ARGUMENTS

**Steps**

1. **Load the goulart-archive skill**

   Load the `goulart-archive` skill and follow it exactly as the authoritative workflow for this invocation. The skill determines whether the verify decision permits archival, whether human warning dispositions are recorded, and whether an override is required. Do not independently decide to skip the decision check.

2. **Pass user context to the skill**

   Provide any change name or store selection from the user arguments to the skill. If no arguments were provided, follow the skill's guidance for scope and change resolution.

3. **Report the skill's output**

   Present the skill's result to the user as the skill produces it.

**Guardrails**
- This command delegates all workflow behavior to the `goulart-archive` skill; do not reimplement or extend it here
- Never perform planning, review, apply, or verify in response to this command
- Preserve the skill's STOP conditions without exception
