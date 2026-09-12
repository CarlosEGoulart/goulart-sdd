---
description: "Goulart SDD single-task implementation"
---

Use the `goulart-apply` skill as the authoritative workflow.

**Implementation boundary**: This command authorizes exactly one implementation task per invocation, as defined by the skill. It never edits planning artifacts, performs review, verification, or archive, or automatically continues to another task or lifecycle stage. All state and gate decisions remain in the skill.

**Input**: Optionally provide a change name or other context after `/goulart-apply` (e.g., `/goulart-apply add-auth`). If no arguments were provided, follow the skill's guidance for resolving scope.
**Provided arguments**: $ARGUMENTS

**Steps**

1. **Load the goulart-apply skill**

   Load the `goulart-apply` skill and follow it exactly as the authoritative workflow for this invocation. The skill determines whether implementation is complete and should hand off to code review, or whether a pending task should be selected and executed. Do not independently decide which path applies.

2. **Pass user context to the skill**

   Provide any change name, store selection, or goal description from the user arguments to the skill. If no arguments were provided, follow the skill's guidance for resolving scope and identifying the change.

3. **Report the skill's output**

   Present the skill's result to the user as the skill produces it. Preserve all STOP conditions without exception.

**Guardrails**
- This command delegates all workflow behavior to the `goulart-apply` skill; do not reimplement or extend it here
- Never perform planning, review, verification, or archive in response to this command
- Preserve the skill's STOP conditions without exception
