---
description: "State-aware Goulart SDD planning with review and human gates"
---

Use the `goulart-plan` skill as the authoritative workflow.

**Planning boundary**: This command authorizes planning only, even if the request says "plan and implement this". It never edits project source code, executes task checkboxes, invokes apply, performs review, verification, or archive, or creates `goulart-continue`/`goulart-resume` commands.

**Input**: Optionally specify a change name or description after `/goulart-plan` (e.g., `/goulart-plan add-auth`). If omitted, ask the user what they want to work on.
**Provided arguments**: $ARGUMENTS

**Steps**

1. **Load the goulart-plan skill**

   Load the `goulart-plan` skill and follow it exactly as the authoritative workflow for this invocation. The skill determines whether the current state requires initial planning, a gate STOP, downstream planning, or completion. Do not independently decide which phase applies.

2. **Pass user context to the skill**

   Provide any change name, store selection, or goal description from the user arguments to the skill. If no arguments were provided, follow the skill's guidance for resolving scope and selecting or creating a change.

3. **Report the skill's output**

   Present the skill's result to the user as the skill produces it.

**Guardrails**
- This command delegates all workflow behavior to the `goulart-plan` skill; do not reimplement or extend it here
- Never perform review, apply, verify, or archive in response to this command
- Preserve the skill's STOP conditions without exception
