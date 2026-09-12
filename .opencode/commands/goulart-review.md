---
description: "Independent plan or code review in the Goulart SDD workflow"
---

Use the `goulart-review` skill as the authoritative workflow.

**Review boundary**: This command authorizes review only, even if the request says "review and fix this". It never edits proposal, specs, design, or implementation files; never edits test files; never executes fixes; never invokes plan, apply, verify, or archive.

**Input**: Optionally specify a stage (`plan` or `code`) and/or a change name after `/goulart-review` (e.g., `/goulart-review plan add-auth`). If the stage is missing or ambiguous, ask the user whether they want plan review or code review.
**Provided arguments**: $ARGUMENTS

**Steps**

1. **Load the goulart-review skill**

   Load the `goulart-review` skill and follow it exactly as the authoritative workflow for this invocation. The skill determines the review stage, independence context, artifact state, findings, verdict, required changes, human disposition/triage, staleness, and override rules. Do not independently decide which stage applies.

2. **Pass user context to the skill**

   Provide any stage, change name, or store selection from the user arguments to the skill. If no arguments were provided, follow the skill's guidance for stage selection and change resolution.

3. **Report the skill's output**

   Present the skill's result to the user as the skill produces it.

**Guardrails**
- This command delegates all workflow behavior to the `goulart-review` skill; do not reimplement or extend it here
- Never perform plan, apply, verify, or archive in response to this command
- Preserve the skill's STOP conditions without exception
