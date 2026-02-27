# /test-cover - Test Coverage Gap Filler

## Overview

The `/test-cover` skill analyzes test coverage, identifies untested code paths prioritized by risk, and fills selected gaps through specialist agents. It also surfaces refactoring suggestions for code that is structurally hard to test.

**Key benefits:**
- Uses coverage reports when available for precise gap identification
- Prioritizes gaps by risk (CRITICAL > HIGH > LOW) so you fix what matters first
- Falls back to manual analysis when no coverage tooling exists
- Shows before/after coverage improvement
- Identifies hard-to-test code and suggests refactoring approaches

## When to Use

**Use `/test-cover` for:**
- Coverage metrics are below your target
- After inheriting or onboarding to an under-tested codebase
- Before a release, to strengthen coverage in critical areas
- After adding features that may lack thorough testing
- When you want data-driven test prioritization

**Don't use `/test-cover` for:**
- Fixing bad existing tests (use `/test-audit`)
- Verifying that tests actually catch bugs (use `/test-mutate`)
- Projects with no source code to test
- When the problem is test quality, not quantity

**Rule of thumb:** If your test suite is clean but incomplete, use `/test-cover`. If your test suite is bloated with bad tests, use `/test-audit` first.

## Supported Coverage Formats

| Language/Tool | Report Files | Auto-detected |
|--------------|-------------|---------------|
| Go | `coverage.out`, `cover.out` | Yes (`go test -coverprofile`) |
| lcov | `lcov.info` | Yes |
| Istanbul/nyc | `coverage-summary.json`, `coverage-final.json` | Yes (`npm run coverage`) |
| coverage.py/pytest | `coverage.xml`, `coverage.json` | Yes (`pytest --cov`) |
| JaCoCo | `jacoco.xml` | Yes (`gradle jacocoTestReport`) |
| Cobertura | `coverage.xml` | Yes |
| Cargo tarpaulin | JSON output | Yes (`cargo tarpaulin`) |

If your project uses a different tool, the skill will ask you for the coverage command.

## Workflow

```
┌─────────────────────────────────────────────────────┐
│                  TEST COVER                          │
├─────────────────────────────────────────────────────┤
│  1. Determine scope                                  │
│  2. Detect/obtain coverage data                      │
│  3. Analyze coverage gaps (qa-coverage-analyst)      │
│  4. Present findings by priority tier                │
│  5. User selects which gaps to fill                  │
│  6. Implement tests (SME agents, parallel by file)   │
│  7. Verify (run test suite)                          │
│  8. Re-run coverage + show improvement               │
│  9. Present refactoring suggestions (informational)  │
│ 10. Summary + optional commit                        │
└─────────────────────────────────────────────────────┘
```

### 1. Determine Scope
Choose what to analyze:
- **Entire project** (default)
- **Specific directory** (`src/`, `pkg/`)
- **Specific files**
- **Recent changes** (files modified on current branch)

### 2. Obtain Coverage Data
The skill tries in order: existing coverage report → detect and run coverage command → ask you → manual analysis. Most projects with a test suite have coverage tooling — the skill usually finds it automatically.

### 3. Analyze
The `qa-coverage-analyst` agent reads coverage data (or inspects source/test files manually) and identifies untested code paths. For large scopes, multiple agents run in parallel on different partitions.

### 4-5. Present and Select
Findings are displayed by priority tier (CRITICAL first). You choose which gaps to fill.

### 6-7. Implement and Verify
Language-specific SME agents write tests in parallel (one per target test file). The test suite runs to confirm everything passes.

### 8. Coverage Improvement
If coverage tooling is available, the skill re-runs coverage and shows a before/after comparison.

### 9. Refactoring Suggestions
Code that is structurally hard to test is reported with specific refactoring approaches. These are informational — use `/refactor` to act on them.

## Example Session

```
> /test-cover

What should I analyze for coverage gaps?
> Entire project

Checking for coverage data...
Found go.mod — running: go test -coverprofile=coverage.out ./...
Baseline coverage: 68.3% of statements

Spawning qa-coverage-analyst for analysis...

## Coverage Gap Analysis

### CRITICAL (2 found)
1. [ADD] auth.go:ValidateJWT — JWT validation error paths untested
2. [ADD] payment.go:ChargeCard — Retry and failure logic untested

### HIGH (3 found)
3. [ADD] parser.go:ParseConfig — Malformed input handling untested
4. [ADD] api.go:CreateUser — Duplicate email conflict untested
5. [ADD] middleware.go:RateLimit — Limit exceeded path untested

Select which gaps to fill:
> 1-5

Spawning 5 swe-sme-golang agents in parallel...
All agents complete. Running test suite... All tests pass.

## Coverage Improvement

              Before    After     Change
Lines         68.3%     79.4%     +11.1%

## Refactoring for Testability

1. payment.go:processTransaction — Extract payment logic from HTTP calls
2. auth.go:middleware — Replace global state with parameter injection

Commit these changes?
> Yes

Committed: "test: fill coverage gaps"
```

## Tips

1. **Run `/test-audit` first.** Clean up bad tests before adding new ones. There's no point filling gaps if existing tests give false confidence.

2. **Start with CRITICAL gaps.** Authentication, payment, and data integrity code deserves tests before configuration defaults.

3. **Pair with `/test-mutate`.** After filling coverage gaps, use `/test-mutate` to verify the new tests actually catch bugs. Line coverage doesn't guarantee assertion quality.

4. **Act on refactoring suggestions.** The testability suggestions are there for a reason — hard-to-test code often hides bugs. Use `/refactor` to address them, then re-run `/test-cover`.

5. **Narrow scope for large projects.** Analyzing the entire project at once can produce an overwhelming number of findings. Focus on one module or directory at a time.

6. **Re-run periodically.** Coverage drifts as new features are added. Run `/test-cover` after major feature work to catch new gaps.

## Integration with Other Skills

| Skill | Relationship |
|-------|-------------|
| `/test-audit` | Run first — clean up bad tests before adding new ones |
| `/test-cover` | This skill — fills coverage gaps |
| `/test-mutate` | Run after — verifies new tests actually catch bugs |
| `/refactor` | Use for testability suggestions from this skill |
| `/iterate` | Includes QA as part of the development cycle |
