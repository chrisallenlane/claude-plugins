# /test-audit - Test Suite Quality Review

## Overview

The `/test-audit` skill reviews test quality interactively: it scans for problematic tests, presents findings for your selection, and implements your chosen fixes. It focuses on identifying tests that are worse than useless — tests that create false confidence, break on every refactor, or test nothing meaningful.

**Key benefits:**
- Catches agent-written test anti-patterns (tautological assertions, over-mocking)
- Interactive — you choose which findings to act on
- Handles both deletion and rewriting
- Parallelizes scanning for large test suites
- Verifies changes don't break the remaining suite

## When to Use

**Use `/test-audit` for:**
- After a burst of agent-written tests (agents write bad tests sometimes)
- When test maintenance feels disproportionate to value
- Before trusting a test suite you didn't write
- Periodically, to keep test quality from drifting
- When tests break on every refactor (sign of brittle tests)

**Don't use `/test-audit` for:**
- Checking if tests pass (just run them)
- Adding test coverage (use `/iterate` or ask directly)
- Projects with no tests yet (nothing to audit)

**Rule of thumb:** If you suspect your tests are giving you false confidence, use `/test-audit`.

## What It Finds

| Category | Description | Typical Action |
|----------|-------------|----------------|
| Tautological | Tests that can't fail (asserting constructor sets fields, mock returns configured value) | DELETE |
| Brittle | Coupled to implementation (exact error strings, over-specified mocks, internal state assertions) | REWRITE |
| Redundant | Duplicate coverage (copy-paste cases hitting same code path) | DELETE |
| False confidence | Tests that don't verify what they claim (missing assertions, asserting on mocks not behavior) | REWRITE |
| Missing coverage | Important gaps (happy paths without sad paths, untested error handling) | ADD |
| Test smells | Structural problems (excessive mocking, god tests, testing private internals) | SIMPLIFY |
| Other | Problems that don't fit the above categories | Varies |

## Workflow

```
┌─────────────────────────────────────────────────────┐
│                   TEST AUDIT                        │
├─────────────────────────────────────────────────────┤
│  1. Determine scope                                 │
│  2. Spawn qa-test-auditor agent(s) for analysis     │
│  3. Present findings as numbered list               │
│  4. User selects which to act on                    │
│  5. Implement selected changes (SME or direct)      │
│  6. Verify (run test suite)                         │
│  7. Summary + optional commit                       │
└─────────────────────────────────────────────────────┘
```

### 1. Determine Scope
Choose what to audit:
- **All tests** (default)
- **Specific directory** (`tests/`, `src/__tests__/`)
- **Specific files**
- **Recent changes** (tests modified on current branch)

For large projects, the skill suggests narrowing scope. You can always re-run on a different scope.

### 2. Scan
The `qa-test-auditor` agent analyzes test files and the production code they test. For large scopes (roughly >15 test files), the skill partitions by directory and spawns multiple auditors in parallel.

### 3. Present and Select
Findings are displayed as a numbered list grouped by category. You select which to act on (e.g., "1-5, 7" or "all").

### 4. Implement
- **DELETE**: Orchestrator removes tests directly
- **REWRITE/SIMPLIFY**: Language-specific SME rewrites to test behavior instead of implementation
- **ADD**: SME writes focused tests for identified gaps

### 5. Verify
Runs the test suite to confirm remaining tests still pass. Reports any failures for your decision.

### 6. Summary
Reports what changed (deleted, rewritten, added) and offers to commit.

## Example Session

```
> /test-audit

What tests should I audit?
> All tests

Spawning qa-test-auditor agent for analysis...

## Test Audit Findings

### Tautological (3 found)
1. [DELETE] user_test.go:TestNewUser — Asserts constructor sets fields
2. [DELETE] config_test.go:TestDefaultConfig — Asserts default struct values
3. [DELETE] model_test.go:TestUserFields — Checks struct has expected fields

### Brittle (2 found)
4. [REWRITE] api_test.go:TestCreateUserError — Exact error string match
5. [REWRITE] handler_test.go:TestNotFound — Asserts on full response body

### Missing Coverage (1 found)
6. [ADD] auth.go:ValidateToken — No tests for expired/malformed tokens

Select which items to address:
> 1-5

Deleting 3 tests (items 1-3)...
Spawning swe-sme-golang for rewrites (items 4-5)...
Running test suite... All tests pass.

## Test Audit Complete
- Tests deleted: 3
- Tests rewritten: 2
- Net test count change: -1

Commit these changes?
> Yes

Committed: "test: audit and clean up test suite"
```

## Tips

1. **Run after agent-written tests.** Agents are prolific test writers but sometimes produce tautological or brittle tests. Audit periodically.

2. **Prefer deletion.** A deleted bad test is better than a rewritten mediocre one. Don't hesitate to select DELETE recommendations.

3. **Scope to suspicious areas.** If certain test files break on every refactor, audit those specifically — they likely have brittle tests.

4. **Review rewrites.** When tests are rewritten to focus on behavior instead of implementation, verify the new tests are actually testing something meaningful.

5. **Complement with /refactor.** `/refactor` improves production code quality; `/test-audit` improves test code quality. Run either independently.
