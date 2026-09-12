---
description: "Verify Goulart SDD implementation completeness and emit the verify decision"
---

Use the `goulart-verify` skill as the authoritative workflow.

**Verification boundary**: This command authorizes verification only, even if the request says "verify and fix this". It never edits proposal, specs, design, test-plan, tasks, plan-review, code-review, or implementation files; never invokes plan, review, apply, or archive.

**Input**: Optionally specify a change name after `/goulart-verify` (e.g., `/goulart-verify add-auth`). If omitted, ask the user what they want to verify.
**Provided arguments**: $ARGUMENTS

**Steps**

1. **Load the goulart-verify skill**

   Load the `goulart-verify` skill and follow it exactly as the authoritative workflow for this invocation. The skill determines prerequisite checks, staleness assessment, spec compliance, test integrity, and the verify decision. Do not independently decide which checks to skip.

2. **Pass user context to the skill**

   Provide any change name or store selection from the user arguments to the skill. If no arguments were provided, follow the skill's guidance for scope and change resolution.

3. **Report the skill's output**

   Present the skill's result to the user as the skill produces it.

**Guardrails**
- This command delegates all workflow behavior to the `goulart-verify` skill; do not reimplement or extend it here
- Never perform planning, review, apply, or archive in response to this command
- Preserve the skill's STOP conditions without exception
