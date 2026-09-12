# Getting Started with Goulart SDD

## Prerequisites

- [OpenSpec CLI](https://github.com/openspec-dev/openspec) installed (`openspec` command available)
- A Goulart-compatible coding-agent harness (e.g., OpenCode)
- Git repository initialized

## Installation

1. Install OpenSpec:
   ```bash
   npm install -g openspec
   ```

2. Initialize OpenSpec in your repository:
   ```bash
   openspec init
   ```

3. The Goulart SDD schema and adapters are part of this repository. No additional installation is needed.

## First-Change Walkthrough

### Step 1: Initial Planning

```bash
/goulart-plan add-auth
```

The author (goulart-plan) creates:
- `proposal.md` — what we're building and why
- `specs/` — requirements with testable scenarios
- `design.md` — architecture decisions

Then **STOPs**. The planning artifacts are ready for review.

### Step 2: Plan Review (Fresh Session)

Open a **fresh session** (important for independence) and run:

```bash
/goulart-review plan add-auth
```

The reviewer evaluates the plan against compliance, quality, feasibility, scope, and risk. Produces a verdict:
- **APPROVE** → human records STATUS: ACCEPTED
- **APPROVE_WITH_CHANGES** → author applies Required Changes
- **REVISE** → author revises and re-reviews

### Step 3: Later Planning (After Review Gate)

Once the plan-review verdict is APPROVE and human has recorded STATUS: ACCEPTED:

```bash
/goulart-plan add-auth
```

The author now creates:
- `test-plan.md` — maps every spec scenario to validation entries
- `tasks.md` — implementation tasks in TDD order

Then **STOPs**. Reports planning complete and identifies `goulart-apply` as the next step.

### Step 4: Implementation (One Task at a Time)

```bash
/goulart-apply add-auth
```

The implementer:
1. Verifies pre-implementation gates (plan-review verdict, human acceptance, test-plan)
2. Selects the first pending task
3. Executes TDD: RED → GREEN → REFACTOR
4. Marks the task complete
5. **STOPs**

Invoke again for the next task:
```bash
/goulart-apply add-auth
```

When all tasks are complete, `goulart-apply` reports completion and instructs you to run `goulart-review code`.

### Step 5: Code Review (Fresh Session)

Open a **fresh session** and run:

```bash
/goulart-review code add-auth
```

The reviewer evaluates implementation against specs, design, test quality, and code quality. Produces a verdict with per-finding triage (ACCEPT/REJECT/DEFER).

### Step 6: Verification

```bash
/goulart-verify add-auth
```

The verifier checks:
- Code-review exists with permitting verdict
- All triage complete
- All tasks complete
- Test-plan entries evaluable
- Reviews fresh
- Spec compliance, test integrity, behavioral coverage

Produces a DECISION: PASS, PASS_WITH_WARNINGS, or FAIL.

### Step 7: Archive

```bash
/goulart-archive add-auth
```

If verify DECISION is PASS (or PASS_WITH_WARNINGS with human acceptance), the change is archived via OpenSpec.

## Goulart-Compliant vs. Raw Execution

**Goulart-compliant** (recommended):
- Follows the full lifecycle with gates, reviews, and verification
- Enforces independence, TDD, and staleness detection
- Provides structured artifacts for auditability

**Raw execution** (escape hatch):
- Use OpenSpec commands directly (`/opsx-propose`, `/opsx-apply`, etc.)
- Bypasses all Goulart gates and guarantees
- Useful when the methodology is too restrictive for a particular use case
- You lose: independent review, TDD enforcement, verification prerequisites, staleness detection

## Key Rules

1. **Fresh sessions for reviews**: Always open a new session for `goulart-review plan` and `goulart-review code`
2. **One task per invocation**: `goulart-apply` processes exactly one task, then STOPs
3. **Never fabricate human decisions**: STATUS, triage, acknowledgement, and override must be explicitly recorded by the human
4. **No Round 3**: If Round 2 is unresolved, escalate to human
5. **No timestamps as freshness evidence**: Use revision/diff comparison
