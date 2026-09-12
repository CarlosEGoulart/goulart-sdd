---
description: "Goulart SDD plan or code review"
---

Use the `goulart-review` skill as the authoritative workflow.

**Review boundary**: This command authorizes review only. It never edits project source code, executes task checkboxes, invokes plan, apply, verify, or archive, performs implementation fixes, or modifies schemas, specs, design, or templates. All stage and verdict decisions remain in the skill.

**Input**: Optionally provide `plan` or `code`, a change name, and other context after `/goulart-review` (e.g., `/goulart-review plan add-auth`). If no arguments were provided, follow the skill's guidance for resolving scope.
**Provided arguments**: $ARGUMENTS

**Steps**

1. **Load the goulart-review skill**

   Load the `goulart-review` skill and follow it exactly as the authoritative workflow for this invocation. The skill determines whether to run plan-review or code-review and which artifact to produce. Do not independently decide the review stage.

2. **Pass user context to the skill**

   Provide any stage, change name, store selection, or goal description from the user arguments to the skill. If no arguments were provided, follow the skill's guidance for resolving scope and identifying the change.

3. **Report the skill's output**

   Present the skill's result to the user as the skill produces it. Preserve all STOP conditions without exception.

**Guardrails**
- This command delegates all workflow behavior to the `goulart-review` skill; do not reimplement or extend it here
- Never perform fixes, planning, implementation, verification, or archive in response to this command
- Preserve the skill's STOP conditions without exception
