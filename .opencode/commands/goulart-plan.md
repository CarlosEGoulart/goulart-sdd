---
description: "State-aware Goulart SDD planning with review and human gates"
---

Start or continue state-aware Goulart SDD planning.

**Planning boundary**: This command authorizes planning only, even if the request says "plan and implement this". It creates or continues proposal, specs, and design artifacts, then stops for an independent plan review. It does not create test-plan, tasks, code-review, or verify artifacts in the same invocation. It never edits project source code, executes task checkboxes, or invokes apply.

**Store selection:** If the user names a store (a store is a standalone OpenSpec repo registered on this machine) or the work lives in one, run `openspec store list --json` to discover registered store ids, then pass `--store <id>` on the commands that read or write specs and changes (`new change`, `status`, `instructions`, `list`, `show`, `validate`, `archive`, `doctor`, `context`, `schemas`, `view`). Once selected, treat `--store <id>` as sticky for the rest of the workflow. Every unscoped example of those commands below is shorthand: before running it, append the flag. For example, run `openspec status --change "<name>" --json --store "<id>"`, not the unscoped form shown below. Other commands do not take the flag. Hints printed by commands already carry the flag; keep it on follow-ups. Without a store, commands act on the nearest local `openspec/` root.

**Input**: Optionally specify a change name or description after `/goulart-plan` (e.g., `/goulart-plan add-auth`). If omitted, ask the user what they want to work on.
**Provided arguments**: $ARGUMENTS

**Steps**

1. **Load and follow the goulart-plan skill**

   Load the `goulart-plan` skill and follow it as the authoritative workflow for this invocation. The skill controls:
   - Scope resolution and change selection
   - Structural and semantic state inspection
   - Initial planning (proposal, specs, design) and the mandatory review stop
   - Plan-review gate evaluation (verdict, human disposition, Required Changes, materiality, freshness, degraded mode, overrides, bounded rounds)
   - Downstream planning (test-plan, coverage check, tasks) when the complete gate permits
   - Planning-complete reporting and the `goulart-apply` handoff

   Do not bypass the skill's STOP conditions, review gates, or planning-only boundary. Do not add independent lifecycle logic, create continuation commands, or perform review, apply, verify, or archive in this invocation.

2. **Pass user context to the skill**

   Provide any change name, store selection, or goal description from the user arguments to the skill. If no arguments were provided, follow the skill's guidance for resolving scope and selecting or creating a change.

3. **Report the skill's output**

   Present the skill's result to the user, including:
   - Change name, schema, and planning home/store
   - Phase/state (initial planning, blocked gate, downstream planning, or complete)
   - Artifacts created/continued, retained, and any incomplete items
   - Review gate outcome (when applicable)
   - Exact unresolved gates (when blocked)
   - Next action (fresh-session review, human resolution, or `goulart-apply`)

**Guardrails**
- This command delegates all workflow behavior to the `goulart-plan` skill; do not reimplement or extend it here
- Never perform review, apply, verify, or archive in response to this command
- Never create `goulart-continue`, `goulart-resume`, or another continuation command
- Preserve the skill's STOP conditions without exception
