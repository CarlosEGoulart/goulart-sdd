---
description: "Goulart SDD archive eligibility gate"
---

Use the `goulart-archive` skill as the authoritative workflow.

**Archive boundary**: This command authorizes the Goulart archive stage only. It never performs planning, implementation, review, verification, fixes, or archive mechanics independently. All verify-result validation, freshness, human-decision, and upstream-delegation behavior remains in the skill.

**Input**: Optionally provide a change name or other archive context after `/goulart-archive` (e.g., `/goulart-archive add-auth`). If no arguments were provided, follow the skill's guidance for resolving scope.
**Provided arguments**: $ARGUMENTS

**Steps**

1. **Load the goulart-archive skill**

   Load the `goulart-archive` skill and follow it exactly as the authoritative workflow for this invocation. The skill determines whether the verify result is applicable, which decision path applies, and whether to delegate to the upstream OpenSpec archive. Do not independently decide which path applies.

2. **Pass user context to the skill**

   Provide any change name, store selection, or goal description from the user arguments to the skill. If no arguments were provided, follow the skill's guidance for resolving scope and identifying the change.

3. **Report the skill's output**

   Present the skill's result to the user as the skill produces it. Preserve all STOP conditions without exception.

**Guardrails**
- This command delegates all workflow behavior to the `goulart-archive` skill; do not reimplement or extend it here
- Never perform planning, review, implementation, verification, or fixes in response to this command
- Preserve the skill's STOP conditions without exception
