# /test-mutation - Mutation Testing Workflow

## Overview

The `/test-mutation` skill verifies your tests actually catch bugs by systematically introducing mutations (deliberate small changes) into your source code and checking if the test suite detects them. A mutation that "survives" (tests still pass) reveals a genuine coverage gap.

**Key benefits:**
- Finds test weaknesses that line coverage metrics miss
- Distinguishes tests that run code from tests that verify behavior
- Multi-session with progress tracking (`.test-mutations.json`)
- Runs on autopilot — set the scope, then walk away
- Writes targeted tests via language SME agents

## When to Use

**Use `/test-mutation` for:**
- Verifying test quality for critical code (auth, payment, data processing)
- Finding blind spots after writing tests for a new feature
- Quality gate before shipping high-stakes changes
- Objective test quality measurement (mutation score is harder to game than line coverage)
- After refactoring, to ensure tests still catch bugs

**Don't use `/test-mutation` for:**
- Projects with no tests yet (write tests first)
- Quick prototypes or throwaway code
- Code that doesn't need high reliability (simple scripts, one-off tools)

**Rule of thumb:** If a bug in this code would wake you up at night, mutation test it.

## How It Works

Mutation testing introduces synthetic bugs one at a time:

| Mutation Type      | Example                                          |
|--------------------|--------------------------------------------------|
| Arithmetic         | `amount + fee` → `amount - fee`                  |
| Relational         | `count < max` → `count <= max`                   |
| Logical            | `valid && active` → `valid \|\| active`          |
| Constants          | `maxRetries = 3` → `maxRetries = 4`              |
| Statement deletion | Remove `audit.Log(event)`                        |
| Control flow       | `if err != nil` → `if err == nil`                |

For each mutation: apply the change, run the test suite, check if tests fail, then revert.

- **Killed**: Tests failed — they caught the mutation (good)
- **Survived**: Tests passed — they missed the mutation (gap found)

**Mutation score** = (mutations killed / total mutations) x 100%

## Workflow

```
┌──────────────────────────────────────────┐
│  SETUP (interactive)                     │
│  Initialize (load/create tracking file)  │
│  Detect test command (first run)         │
│  Select scope (default: all pending)     │
│                                          │
│  EXECUTION (autopilot)                   │
│  For each module in scope:               │
│    Apply mutations, run tests each time  │
│    Write tests targeting ALL survivors   │
│    Verify new tests kill the mutations   │
│    Commit changes                        │
│  Final summary                           │
└──────────────────────────────────────────┘
```

### First Run

The skill detects your test command (`make test`, `pytest`, `go test`, etc.), scans for source files, and creates `.test-mutations.json` to track progress.

### Scope Selection

You can specify individual files or test everything (the default). After scope is confirmed, the workflow runs unattended through all selected modules.

### Subsequent Runs

Loads progress from the tracking file, shows what's been tested and what's pending, and lets you pick the scope for this session.

### Autopilot Execution

For each module in scope, the workflow:
1. Spawns a mutator agent to apply mutations and identify survivors
2. Spawns an SME to write tests for all surviving mutations
3. Verifies the new tests actually kill the mutations
4. Commits changes and moves to the next module

If something goes wrong (SME can't write a test, new tests fail, a mutation resists), the workflow logs the issue and continues with the next module. A final summary reports everything that succeeded and everything that was skipped.

## Mutation Score Interpretation

| Score   | Meaning                                                 |
|---------|---------------------------------------------------------|
| 95-100% | Excellent — tests catch nearly all synthetic bugs       |
| 80-94%  | Good — some gaps, worth addressing for critical code    |
| 60-79%  | Adequate for non-critical code, weak for critical paths |
| < 60%   | Significant gaps — tests provide limited protection     |

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
> /test-mutation

Loading .test-mutations.json...
Project: 3/10 modules tested (overall score: 85%)

Scope? Enter file names/numbers, or press Enter to test all pending.
> [Enter]

Starting autopilot run for 7 modules...

[1/7] src/api/handler.go — score: 92% → 98%, 3 tests added. Continuing...
[2/7] src/middleware.go — score: 100%, no survivors. Continuing...
[3/7] src/models/user.go — score: 88% → 96%, 2 tests added. Continuing...
...
[7/7] src/utils/parser.go — score: 90% → 100%, 3 tests added.

## Mutation Testing Complete

| Module                 | Before | After | Tests Added | Unresolved |
|------------------------|--------|-------|-------------|------------|
| src/api/handler.go     | —      | 98%   | 3           | 1          |
| src/middleware.go      | —      | 100%  | 0           | 0          |
| ...                    |        |       |             |            |

Project: 10/10 modules complete, overall score: 94%
Commits: 7
Unresolved survivors: 3 (see summary above)
```

## Tips

1. **Start with critical modules.** Auth, payment, data validation — the code where bugs matter most. Pass specific files as the scope.

2. **Or just test everything.** Press Enter at the scope prompt and let it run. Come back when it's done.

3. **Pair with /review-test.** Run `/review-test` first to fill coverage gaps and clean up bad tests, then `/test-mutation` to find remaining weaknesses. Review builds, mutate strengthens.

4. **Don't chase 100%.** Some surviving mutations (e.g., constant tweaks, logging removal) may not be worth testing. The final summary lists unresolved survivors so you can decide later.

5. **Commit the tracking file.** Share mutation scores with your team. Track improvements over time.

6. **Re-test after refactoring.** Refactoring can weaken tests (they still pass but catch fewer mutations). Run `/test-mutation` on refactored modules to verify.

## Integration with Other Skills

`/test-mutation`, `/review-test`, `/implement`, and `/refactor` are complementary:

- **Use /review-test** to fill coverage gaps and audit test quality
- **Use /test-mutation** to find tests that run code without verifying behavior
- **Use /implement** to build features with quality gates
- **Use /refactor** for code cleanup, then `/test-mutation` to verify tests weren't weakened

Recommended sequence for test improvement: `/review-test` first (fill gaps, clean up), then `/test-mutation` (strengthen).
