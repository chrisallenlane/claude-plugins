# /test-mutate - Mutation Testing Workflow

## Overview

The `/test-mutate` skill verifies your tests actually catch bugs by systematically introducing mutations (deliberate small changes) into your source code and checking if the test suite detects them. A mutation that "survives" (tests still pass) reveals a genuine coverage gap.

**Key benefits:**
- Finds test weaknesses that line coverage metrics miss
- Distinguishes tests that run code from tests that verify behavior
- Multi-session with progress tracking (`.test-mutations.json`)
- Interactive — you choose which survivors to address
- Writes targeted tests via language SME agents

## When to Use

**Use `/test-mutate` for:**
- Verifying test quality for critical code (auth, payment, data processing)
- Finding blind spots after writing tests for a new feature
- Quality gate before shipping high-stakes changes
- Objective test quality measurement (mutation score is harder to game than line coverage)
- After refactoring, to ensure tests still catch bugs

**Don't use `/test-mutate` for:**
- Projects with no tests yet (write tests first)
- Quick prototypes or throwaway code
- Code that doesn't need high reliability (simple scripts, one-off tools)

**Rule of thumb:** If a bug in this code would wake you up at night, mutation test it.

## How It Works

Mutation testing introduces synthetic bugs one at a time:

| Mutation Type | Example |
|---------------|---------|
| Arithmetic | `amount + fee` → `amount - fee` |
| Relational | `count < max` → `count <= max` |
| Logical | `valid && active` → `valid \|\| active` |
| Constants | `maxRetries = 3` → `maxRetries = 4` |
| Statement deletion | Remove `audit.Log(event)` |
| Control flow | `if err != nil` → `if err == nil` |

For each mutation: apply the change, run the test suite, check if tests fail, then revert.

- **Killed**: Tests failed — they caught the mutation (good)
- **Survived**: Tests passed — they missed the mutation (gap found)

**Mutation score** = (mutations killed / total mutations) x 100%

## Workflow

```
┌──────────────────────────────────────────┐
│  Initialize (load/create tracking file) │
│              |                           │
│  Pick a module to mutate                │
│              |                           │
│  Apply mutations, run tests each time    │
│              |                           │
│  Show surviving mutations                │
│              |                           │
│  User selects survivors to address       │
│              |                           │
│  Write tests targeting survivors         │
│              |                           │
│  Verify new tests kill the mutations     │
│              |                           │
│  Update progress, commit                 │
└──────────────────────────────────────────┘
```

### First Run

The skill detects your test command (`make test`, `pytest`, `go test`, etc.), scans for source files, and creates `.test-mutations.json` to track progress.

### Subsequent Runs

Loads progress from the tracking file, shows what's been tested and what's pending, and lets you pick the next module.

### After Mutations

Surviving mutations are presented as a numbered list. You select which to address, and the skill spawns a language-appropriate SME to write targeted tests. After writing, it re-applies the mutations to confirm the new tests catch them.

## Mutation Score Interpretation

| Score | Meaning |
|-------|---------|
| 95-100% | Excellent — tests catch nearly all synthetic bugs |
| 80-94% | Good — some gaps, worth addressing for critical code |
| 60-79% | Adequate for non-critical code, weak for critical paths |
| < 60% | Significant gaps — tests provide limited protection |

100% isn't always necessary or practical. Focus effort on critical code paths.

## Progress Tracking

The skill maintains `.test-mutations.json` in your project root:

```json
{
  "version": "1.0",
  "test_command": "pytest",
  "modules": {
    "src/auth.py": {
      "status": "completed",
      "mutation_score": 95.5
    },
    "src/payment.py": {
      "status": "in_progress",
      "mutation_score": 72.0
    }
  },
  "global_statistics": {
    "overall_score": 87.0,
    "completed_modules": 1,
    "total_modules": 8
  }
}
```

This file persists across sessions. You can commit it to share mutation scores with your team, or add it to `.gitignore` to keep it local.

## Example Session

```
> /test-mutate

Loading .test-mutations.json...
Project: 3/10 modules tested (overall score: 85%)

Next pending: src/payment.go
Proceed? > yes

Spawning qa-test-mutator...
Applying 40 mutations...

Results:
- Killed: 34 (85%)
- Survived: 6

Surviving Mutations:
1. [Line 42] amount + fee → amount - fee
2. [Line 78] price > 0 → price >= 0
3. [Line 103] valid && active → valid || active
4. [Line 118] err != nil → err == nil
5. [Line 92] maxRetries = 3 → maxRetries = 4
6. [Line 55] removed audit.Log(transaction)

Address which survivors?
> 1-4

Writing tests... Verifying... All mutations now killed.
Mutation score: 85% → 95%

Continue with next module? > no

Session summary:
- Tested: src/payment.go
- Tests added: 4
- Project score: 85% → 87%

Commit? > yes
```

## Tips

1. **Start with critical modules.** Auth, payment, data validation — the code where bugs matter most.

2. **Incremental approach.** Test one module per session. Mutation testing is inherently slow (one test run per mutation).

3. **Pair with /test-audit.** Run `/test-audit` first to remove bad tests, then `/test-mutate` to find missing ones. Audit cleans, mutate strengthens.

4. **Don't chase 100%.** Address high-value survivors (business logic, error handling), accept diminishing returns on the rest.

5. **Commit the tracking file.** Share mutation scores with your team. Track improvements over time.

6. **Re-test after refactoring.** Refactoring can weaken tests (they still pass but catch fewer mutations). Run `/test-mutate` on refactored modules to verify.

## Integration with Other Skills

`/test-mutate`, `/test-audit`, `/iterate`, and `/refactor` are complementary:

- **Use /test-audit** to remove bad tests (quality)
- **Use /test-mutate** to find missing tests (coverage)
- **Use /iterate** to build features with quality gates
- **Use /refactor** for code cleanup, then `/test-mutate` to verify tests weren't weakened

Recommended sequence for test improvement: `/test-audit` first (clean up), then `/test-mutate` (strengthen).
