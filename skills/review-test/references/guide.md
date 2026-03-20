# /review-test - Comprehensive Test Suite Review

## Overview

The `/review-test` skill performs a three-phase test suite review: fill coverage gaps, identify missing fuzz tests, and audit test quality. Each phase runs its own analysis → present → select → implement → verify cycle.

**Key benefits:**
- Systematic: addresses coverage, fuzz, and quality in deliberate order
- Interactive: you see findings and choose what to address at each phase
- Parallel analysis: large scopes are partitioned across multiple agents
- Language-aware: dispatches to appropriate SME agents for implementation
- Measures improvement: before/after coverage comparison when tooling is available

## When to Use

**Use `/review-test` for:**
- Coverage metrics below target or onboarding to an under-tested codebase
- After a burst of agent-written tests that may need quality review
- Before a release, to strengthen and clean up the test suite
- Periodic comprehensive test health checks

**Don't use `/review-test` for:**
- Projects with no tests yet (write initial tests first)
- Quick one-off test additions (just write them directly)
- Mutation testing (use `/test-mutation` for that)

**Rule of thumb:** `/review-test` builds breadth (fill gaps, clean up). `/test-mutation` builds depth (verify tests actually catch bugs).

## Workflow Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│ /review-test Workflow                                            │
└─────────────────────────────────────────────────────────────────┘

 ┌──────────────────────────────────────────────┐
 │  1. DETERMINE SCOPE                          │
 │  ────────────────────────────────────────    │
 │  • Entire project (default)                  │
 │  • Specific directory or files               │
 │  • Recent changes (git diff)                 │
 └──────────────────┬───────────────────────────┘
                    ▼
 ┌──────────────────────────────────────────────┐
 │  PHASE 1: COVERAGE GAPS                      │
 │  ────────────────────────────────────────    │
 │  1a. Detect/obtain coverage data             │
 │      (existing report → generate → ask →     │
 │       manual analysis fallback)              │
 │  1b. Analyze gaps (qa-test-coverage-reviewer)       │
 │  1c. Present findings by priority tier       │
 │      (CRITICAL → HIGH → LOW)                 │
 │  1d. User selects which gaps to fill         │
 │  1e. SME implements selected tests           │
 │  1f. Verify + re-run coverage                │
 └──────────────────┬───────────────────────────┘
                    ▼
 ┌──────────────────────────────────────────────┐
 │  PHASE 2: FUZZ COVERAGE                      │
 │  ────────────────────────────────────────    │
 │  2a. Analyze fuzz gaps (qa-test-fuzz-reviewer)      │
 │  2b. Check infrastructure                    │
 │      No fuzz infra? → Skip to Phase 3        │
 │  2c. Present candidates by priority          │
 │  2d. User selects which to add               │
 │  2e. SME implements fuzz tests               │
 │  2f. Verify (compilation + seed corpus)      │
 └──────────────────┬───────────────────────────┘
                    ▼
 ┌──────────────────────────────────────────────┐
 │  PHASE 3: TEST QUALITY AUDIT                 │
 │  ────────────────────────────────────────    │
 │  3a. Scan for issues (qa-test-reviewer)        │
 │      • Tautological (can't fail)             │
 │      • Brittle (coupled to implementation)   │
 │      • Redundant (informational only)        │
 │      • False confidence                      │
 │      • Inconsistent assertions               │
 │  3b. Present findings by category            │
 │  3c. User selects which to address           │
 │  3d. SME implements changes                  │
 │  3e. Verify                                  │
 └──────────────────┬───────────────────────────┘
                    ▼
 ┌──────────────────────────────────────────────┐
 │  SUMMARY + OPTIONAL COMMIT                   │
 │  ────────────────────────────────────────    │
 │  • Per-phase results                         │
 │  • Net test count change                     │
 │  • Coverage improvement (if measurable)      │
 │  • Refactoring-for-testability suggestions   │
 └──────────────────────────────────────────────┘
```

## Phase Details

### Phase 1: Coverage Gaps

Detects or generates a coverage report using a four-step waterfall:

1. **Check for existing artifacts** (coverage.out, lcov.info, etc.)
2. **Detect coverage command** from build files (Makefile, package.json, go.mod, etc.)
3. **Ask the user** for the correct command
4. **Manual analysis fallback** — read source and test files to identify gaps by inspection

For large scopes (>15 source files), the analysis is partitioned across multiple `qa-test-coverage-reviewer` agents running in parallel.

Findings are grouped by priority (CRITICAL / HIGH / LOW). Refactoring-for-testability suggestions are collected separately and presented in the final summary.

### Phase 2: Fuzz Coverage

A single `qa-test-fuzz-reviewer` agent checks whether fuzz testing infrastructure exists and identifies functions that are good fuzz candidates.

If no fuzz infrastructure is detected, the phase is skipped with a recommendation for tooling. No attempt is made to set up fuzz tooling.

### Phase 3: Test Quality Audit

For large scopes (>15 test files), the audit is partitioned across multiple `qa-test-reviewer` agents running in parallel.

Issue categories and recommended actions:

| Category       | Action    | Description                                    |
|----------------|-----------|------------------------------------------------|
| Tautological   | DELETE    | Tests that can't fail                          |
| Brittle        | REWRITE   | Tests coupled to implementation details        |
| False confidence | REWRITE | Tests that don't verify what they claim        |
| Inconsistent   | SIMPLIFY  | Mixed assertion strategies                     |
| Missing        | ADD       | Important gaps not caught in Phase 1           |
| Redundant      | (info)    | Duplicate coverage — reported but not actioned |

SMEs receiving DELETE recommendations may choose to REWRITE instead if the test covers real behavior that could regress.

## Example Session

```
> /review-test

What should I review?
> Entire project

## Phase 1: Coverage Gap Analysis

Overall coverage: 68.3% lines (baseline)

### CRITICAL (2 found)
1. [ADD] auth.go:ValidateJWT (lines 45-72) — error paths untested
2. [ADD] payment.go:ChargeCard (lines 88-120) — retry logic untested

### HIGH (3 found)
3. [ADD] parser.go:ParseConfig (lines 30-55) — malformed input
4. [ADD] api.go:CreateUser (lines 15-40) — duplicate email conflict
5. [ADD] middleware.go:RateLimit (lines 22-45) — limit exceeded path

Select which gaps to fill:
> 1-5

Writing tests... Verifying...

Coverage: 68.3% → 78.1% (+9.8%)

## Phase 2: Fuzz Coverage

Fuzz infrastructure: native testing.F (Go 1.22)

### HIGH (2 found)
1. [ADD] parser.go:ParseConfig — arbitrary []byte input
2. [ADD] protocol.go:DecodeMessage — wire protocol messages

Select which fuzz tests to add:
> all

Writing fuzz tests... Verifying...

## Phase 3: Test Quality Audit

### Tautological (2 found)
1. [DELETE] model_test.go:TestUserStruct — checks struct fields exist
2. [DELETE] config_test.go:TestDefaultConfig — asserts hardcoded values

### Brittle (1 found)
3. [REWRITE] api_test.go:TestCreateUserError — exact error string match

Select which items to address:
> all

Implementing changes... Verifying...

## Test Review Complete

### Phase 1: Coverage Gaps
- Tests added: 5
- Coverage: 68.3% → 78.1% (+9.8%)

### Phase 2: Fuzz Coverage
- Fuzz tests added: 2

### Phase 3: Test Quality Audit
- Tests deleted: 2
- Tests rewritten: 1

### Net Change
- Total tests added: 7
- Total tests removed: 2
- Net: +5

Commit? > yes
```

## Agent Coordination

| Phase   | Analysis Agent              | Parallelized    | Implementation |
|---------|-----------------------------|-----------------|----------------|
| Phase 1 | `qa-test-coverage-reviewer` | Yes (>15 files) | Language SME   |
| Phase 2 | `qa-test-fuzz-reviewer`     | No (single)     | Language SME   |
| Phase 3 | `qa-test-reviewer`          | Yes (>15 files) | Language SME   |

Implementation is always parallelized by target test file — findings targeting the same file go to the same SME agent.

**Fresh instances:** Every agent spawn is a fresh instance. No state carried between invocations.

**State maintained by orchestrator:**
- Scope (shared across all phases)
- Coverage command and baseline metrics
- User selections for each phase
- Implementation results per phase
- Refactoring suggestions (held for final summary)
- Running totals for summary

## Integration with Other Skills

| Skill          | Relationship                                                                              |
|----------------|-------------------------------------------------------------------------------------------|
| `/test-mutation` | Complementary. `/review-test` builds breadth, `/test-mutation` builds depth.                |
| `/implement`     | `/implement` includes QA as part of feature development. `/review-test` is a standalone audit. |
| `/refactor`    | Run `/review-test` before refactoring to ensure tests are strong enough to catch regressions. |

Recommended sequence for test improvement: `/review-test` first (fill gaps, clean up), then `/test-mutation` (verify tests catch bugs).
