---
description: "Execute one Goulart SDD implementation task with TDD"
---

Use the `goulart-apply` skill as the authoritative workflow.

**Implementation boundary**: This command authorizes one-task TDD implementation only, even if the request says "implement everything". It never edits proposal, specs, design, test-plan, plan-review, code-review, or verify artifacts; never invokes plan, review, verify, or archive.

**Input**: Optionally specify a change name after `/goulart-apply` (e.g., `/goulart-apply add-auth`). If omitted, ask the user what they want to work on.
**Provided arguments**: $ARGUMENTS

**Steps**

1. **Load the goulart-apply skill**

   Load the `goulart-apply` skill and follow it exactly as the authoritative workflow for this invocation. The skill determines pre-implementation gates, task selection, TDD execution, bounded retry, spec drift handling, and completion reporting. Do not independently decide which task to implement.

2. **Pass user context to the skill**

   Provide any change name or store selection from the user arguments to the skill. If no arguments were provided, follow the skill's guidance for scope and change resolution.

3. **Report the skill's output**

   Present the skill's result to the user as the skill produces it.

**Guardrails**
- This command delegates all workflow behavior to the `goulart-apply` skill; do not reimplement or extend it here
- Never perform planning, review, verify, or archive in response to this command
- Preserve the skill's STOP conditions without exception
