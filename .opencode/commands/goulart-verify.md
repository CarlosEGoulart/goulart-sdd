---
description: "Goulart SDD final verification"
---

Use the `goulart-verify` skill as the authoritative workflow.

**Verification boundary**: This command authorizes verification only. It never edits planning artifacts, performs planning, implementation, review, fixes, or archive. All prerequisite, staleness, override, audit, and decision behavior remains in the skill.

**Input**: Optionally provide a change name or other context after `/goulart-verify` (e.g., `/goulart-verify add-auth`). If no arguments were provided, follow the skill's guidance for resolving scope.
**Provided arguments**: $ARGUMENTS

**Steps**

1. **Load the goulart-verify skill**

   Load the `goulart-verify` skill and follow it exactly as the authoritative workflow for this invocation. The skill determines whether prerequisites permit the audit, whether the audit runs, and which DECISION to emit. Do not independently decide which path applies.

2. **Pass user context to the skill**

   Provide any change name, store selection, or goal description from the user arguments to the skill. If no arguments were provided, follow the skill's guidance for resolving scope and identifying the change.

3. **Report the skill's output**

   Present the skill's result to the user as the skill produces it. Preserve all STOP conditions without exception.

**Guardrails**
- This command delegates all workflow behavior to the `goulart-verify` skill; do not reimplement or extend it here
- Never perform planning, review, implementation, fixes, or archive in response to this command
- Preserve the skill's STOP conditions without exception
