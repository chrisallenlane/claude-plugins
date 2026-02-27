---
name: test-cover
description: Fill test coverage gaps. Analyzes coverage reports (or runs coverage tooling), identifies untested code paths prioritized by risk, and implements selected test additions through specialist agents. Reports refactoring-for-testability suggestions.
model: opus
---

# Test Cover — Coverage Gap Filler

Analyzes test coverage, identifies untested code paths prioritized by risk, fills selected gaps through specialist agents, and surfaces refactoring suggestions for hard-to-test code.

## Philosophy

**Cover what matters most.** Not all uncovered code is equally important. Error handling, security checks, and business logic deserve tests before getters and boilerplate. Prioritize by consequence of a bug, not by line count.

## Workflow Overview

```
┌─────────────────────────────────────────────────────┐
│                  TEST COVER                          │
├─────────────────────────────────────────────────────┤
│  1. Determine scope                                  │
│  2. Detect/obtain coverage data                      │
│  3. Spawn qa-coverage-analyst agent(s) (analysis)    │
│  4. Present findings to user (numbered list)         │
│  5. User selects which gaps to fill                  │
│  6. Implement selected tests (SME or direct)         │
│  7. Verify (run test suite)                          │
│  8. Re-run coverage + show improvement               │
│  9. Present refactoring suggestions (informational)  │
│ 10. Summary + optional commit                        │
└─────────────────────────────────────────────────────┘
```

## Workflow Details

### 1. Determine Scope

**Ask the user:** "What should I analyze for coverage gaps?"

Present these options:
- **Entire project**: Analyze all source files (default)
- **Specific directory**: A path like `src/`, `pkg/`, `lib/`
- **Specific files**: Individual source files
- **Recent changes**: Files modified on the current branch (via `git diff`)

**Default:** Entire project.

**If the project is large** (many source files), suggest narrowing scope to keep findings manageable. The user can always re-run on a different scope.

### 2. Detect/Obtain Coverage Data

Follow this waterfall — stop at the first step that produces a usable report.

#### Step A: Check for existing coverage artifacts

Search for coverage files in common locations:

| Format | Files to search for |
|--------|-------------------|
| Go | `coverage.out`, `cover.out`, `c.out` |
| lcov | `lcov.info`, `coverage/lcov.info` |
| Istanbul/nyc | `coverage/coverage-summary.json`, `coverage/coverage-final.json`, `.nyc_output/` |
| coverage.py | `coverage.xml`, `coverage.json`, `htmlcov/` |
| JaCoCo | `target/site/jacoco/jacoco.xml`, `build/reports/jacoco/*/jacoco.xml` |
| Cobertura | `coverage.xml`, `cobertura.xml` |

If a report is found, verify it's reasonably recent (warn if the file is older than the most recent source change — it may be stale). Use the report and proceed to step 3.

#### Step B: Detect coverage command

If no report exists, detect how to generate one:

1. `Makefile` with a `cover` or `coverage` target → `make cover` (or `make coverage`)
2. `package.json` with a `coverage` script → `npm run coverage`
3. `go.mod` present → `go test -coverprofile=coverage.out ./...`
4. `pyproject.toml` / `setup.cfg` / `pytest.ini` with coverage config → `pytest --cov --cov-report=json`
5. `Cargo.toml` → `cargo tarpaulin --out json` (or `cargo llvm-cov --json`)
6. `build.gradle` / `build.gradle.kts` → `gradle jacocoTestReport`

If detected, **run the command** and parse the output. Store the command for re-use in step 8.

**Verify the command works** by checking its exit code and that it produces a report file. If it fails, report the error and ask the user for the correct command.

#### Step C: Ask the user

If no coverage tooling is detected: "What command generates a coverage report for this project?" Run whatever they provide.

#### Step D: Manual analysis fallback

If the user has no coverage tooling (or doesn't want to set one up), proceed with manual analysis. The agent will read source and test files to identify gaps by inspection.

**Note:** In manual analysis mode, quantitative coverage improvement (step 8) is unavailable.

**Store:** the coverage command (if any) and baseline coverage percentage for later comparison.

### 3. Spawn Coverage Analysis Agent(s)

**First, assess the scope size.** Use Glob to count source files in scope.

#### Small scope (roughly ≤15 source files)

Spawn a single `qa-coverage-analyst` agent with the full scope and coverage data.

#### Large scope (roughly >15 source files)

Partition the scope by directory or module. Spawn multiple `qa-coverage-analyst` agents **in parallel**, each with a focused partition and the relevant portion of the coverage data.

After all agents complete, merge their findings into a single list ordered by priority tier (CRITICAL → HIGH → LOW). Collect all REFACTOR-FOR-TESTABILITY suggestions separately.

#### Prompt for each agent

```
Analyze test coverage gaps.
Scope: [partition or full scope]
Mode: [coverage report / coverage command / manual analysis]
Coverage data: [file path or "manual analysis — no data"]

Identify:
- Untested code paths prioritized by risk (CRITICAL / HIGH / LOW)
- Code that is structurally hard to test (REFACTOR-FOR-TESTABILITY suggestions)

Return structured findings with ADD recommendations and refactoring suggestions.
```

**If no agent reports significant gaps:** Present "No significant coverage gaps found" to user and exit.

### 4. Present Findings

Display the agent's findings as a numbered list, grouped by priority tier. Refactoring suggestions are held back until step 9 — mention their count but don't display them yet.

**Example presentation:**

```
## Coverage Gap Analysis

Overall coverage: 68.3% lines (baseline)

### CRITICAL (2 found)
1. [ADD] auth.go:ValidateJWT (lines 45-72) — JWT validation error paths untested
   Risk: Invalid tokens could bypass authentication
2. [ADD] payment.go:ChargeCard (lines 88-120) — Retry and failure logic untested
   Risk: Silent charge failures or double charges

### HIGH (3 found)
3. [ADD] parser.go:ParseConfig (lines 30-55) — Malformed input handling untested
4. [ADD] api.go:CreateUser (lines 15-40) — Duplicate email conflict untested
5. [ADD] middleware.go:RateLimit (lines 22-45) — Limit exceeded path untested

### LOW (2 found)
6. [ADD] config.go:Defaults (lines 5-12) — Default value coverage
7. [ADD] router.go:RegisterRoutes (lines 8-25) — Route registration

### Refactoring Suggestions
2 testability improvements identified (shown after implementation)

Select which gaps to fill (e.g., "1-5" or "all"):
```

**Use `AskUserQuestion`** with multi-select to let the user choose. If there are more than ~10 findings, batch them by tier and present in rounds.

### 5. Implement Selected Tests

**Detect appropriate SME and spawn based on project language:**
- Go: `swe-sme-golang`
- Dockerfile: `swe-sme-docker`
- Makefile: `swe-sme-makefile`
- GraphQL: `swe-sme-graphql`
- Ansible: `swe-sme-ansible`
- Zig: `swe-sme-zig`

**For languages without a dedicated SME:** implement directly as orchestrator.

#### Parallelization

**Group findings by target test file**, then spawn one SME agent per file **in parallel**. Findings targeting the same test file must go to the same agent to avoid edit conflicts. Findings targeting different files are independent and should always run concurrently.

- If all findings target one test file, spawn a single agent.
- If findings target N test files, spawn up to N agents in parallel, one per file.
- If a test file doesn't exist yet and needs to be created, that's its own agent.

**Prompt each SME agent with:**
```
Write tests to fill the following coverage gaps in [source file].
Target test file: [test file]

Gaps to cover:
1. [function_name (lines N-M)]: [what is untested]
   Should verify: [specific test description from analyst]

2. ...

Guidelines:
- Write focused tests targeting the specific untested code paths.
- Follow the project's existing test conventions and framework.
- Test behavior, not implementation details.
- Cover the error/edge cases identified in the gap analysis.
- Each test should have a clear name indicating what it verifies.
- If existing test helpers or fixtures are available, use them.
```

### 6. Verify

**Run the test suite** to confirm:
- New tests pass
- Existing tests still pass

**If failures:**
- Report which tests failed and why
- For failures in new tests: attempt one fix, then report to user
- For failures in existing tests: report to user (new tests may have exposed a real bug — this is a good thing)
- Let user decide how to proceed

### 7. Re-run Coverage

**If a coverage command was established in step 2:**

1. Re-run the coverage command
2. Parse the new report
3. Display before/after comparison:

```
## Coverage Improvement

              Before    After     Change
Lines         68.3%     78.1%     +9.8%
Branches      61.2%     72.5%     +11.3%
```

Show whichever metrics are available from the coverage format. Many formats only report line coverage — that's fine.

**If manual analysis mode:** skip this step with: "No coverage tooling available — cannot measure improvement quantitatively."

### 8. Present Refactoring Suggestions

Display the REFACTOR-FOR-TESTABILITY findings collected from the analyst agent(s). These are purely informational — not selectable for implementation within this workflow.

```
## Refactoring for Testability (informational)

The following code patterns limit testability. Consider addressing these
with /refactor to enable better coverage in future passes.

1. payment.go:processTransaction — Business logic mixed with HTTP calls
   Suggestion: Extract payment calculation into a pure function; inject HTTP client
   Would enable testing: charge amount calculation, retry logic

2. auth.go:middleware — Global state dependency
   Suggestion: Accept auth config as parameter instead of reading global
   Would enable testing: token validation with different configurations

These suggestions are not implemented by this workflow. Use /refactor
or address them manually.
```

### 9. Summary

Present what was changed:

```
## Test Coverage Complete

### Changes
- Tests added: N
- Test files created: N
- Test files modified: N

### Coverage
- Before: XX.X%
- After: YY.Y%
- Improvement: +Z.Z%

### Refactoring Suggestions
- N testability improvements identified (see above)

### Files Modified
- [file]: [what changed]

### Skipped (user declined)
- N gaps not addressed
```

**Ask user if they want to commit.** If yes, create a commit:

```bash
git add [specific test files]
git commit -m "$(cat <<'EOF'
test: fill coverage gaps

Added N tests covering [brief description of areas].
Coverage: XX% → YY%

Co-Authored-By: Claude <noreply@anthropic.com>
EOF
)"
```

## Agent Coordination

**Coverage detection phase:** Sequential. Detect tooling, run coverage command if needed.

**Analysis phase:** Spawn `qa-coverage-analyst` agent(s). For large scopes, partition and run in parallel (see step 3).

**Implementation phase:** Parallel by file. Group findings by target test file, spawn one SME per file, run all concurrently. Wait for all to complete before verification.

**Fresh instances:**
- Spawn fresh `qa-coverage-analyst` for each partition
- Spawn fresh SME for each target file

**State to maintain (as orchestrator):**
- Coverage command and baseline metrics
- User's selected findings
- Implementation results (success/failure per agent/file)
- Refactoring suggestions (held for step 8)
- Running totals for summary

## Abort Conditions

**Abort workflow:**
- User interrupts
- No source files found in scope

**Do NOT abort for:**
- Coverage command failure (fall back to manual analysis)
- Individual SME failures (report and continue with remaining)
- Test suite failures after changes (report and let user decide)
- Coverage re-run failure (report, skip improvement metrics)

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

Overall coverage: 68.3% lines (baseline)

### CRITICAL (2 found)
1. [ADD] auth.go:ValidateJWT (lines 45-72) — JWT validation error paths untested
   Risk: Invalid tokens could bypass authentication
2. [ADD] payment.go:ChargeCard (lines 88-120) — Retry and failure logic untested
   Risk: Silent charge failures or double charges

### HIGH (3 found)
3. [ADD] parser.go:ParseConfig (lines 30-55) — Malformed input handling untested
4. [ADD] api.go:CreateUser (lines 15-40) — Duplicate email conflict untested
5. [ADD] middleware.go:RateLimit (lines 22-45) — Limit exceeded path untested

### LOW (2 found)
6. [ADD] config.go:Defaults (lines 5-12) — Default value coverage
7. [ADD] router.go:RegisterRoutes (lines 8-25) — Route registration

### Refactoring Suggestions
2 testability improvements identified (shown after implementation)

Select which gaps to fill:
> 1-5

Findings span 5 files. Spawning 5 swe-sme-golang agents in parallel...
  - auth_test.go: 1 gap (ValidateJWT error paths)
  - payment_test.go: 1 gap (ChargeCard retry/failure)
  - parser_test.go: 1 gap (ParseConfig malformed input)
  - api_test.go: 1 gap (CreateUser duplicate email)
  - middleware_test.go: 1 gap (RateLimit exceeded)
All agents complete.

Running test suite... All tests pass.

Re-running coverage...

## Coverage Improvement

              Before    After     Change
Lines         68.3%     79.4%     +11.1%

## Refactoring for Testability (informational)

1. payment.go:processTransaction — Business logic mixed with HTTP calls
   Suggestion: Extract payment calculation into a pure function
   Would enable testing: charge amount calculation, retry logic

2. auth.go:middleware — Global state dependency
   Suggestion: Accept auth config as parameter
   Would enable testing: token validation with different configurations

## Test Coverage Complete

### Changes
- Tests added: 14
- Test files modified: 5

### Coverage
- Before: 68.3%
- After: 79.4%
- Improvement: +11.1%

### Refactoring Suggestions
- 2 testability improvements identified

Commit these changes?
> Yes

Committed: "test: fill coverage gaps"
```
