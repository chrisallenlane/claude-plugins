---
name: test-audit
description: Audit test suite quality. Identifies brittle, tautological, and useless tests, then selectively fixes or removes them.
model: opus
---

# Test Audit — Interactive Test Quality Review

Reviews test suite quality, presents findings for user selection, and implements chosen fixes. Focuses on identifying tests that are worse than useless — tests that create false confidence, break on every refactor, or test nothing meaningful.

## Philosophy

**Tests exist to catch bugs, not to exist.** A test that can't fail, tests the wrong thing, or breaks whenever you refactor is actively harmful. It costs maintenance time, slows the suite, and creates false confidence. Deleting a bad test is an improvement.

## Workflow Overview

```
┌─────────────────────────────────────────────────────┐
│                   TEST AUDIT                        │
├─────────────────────────────────────────────────────┤
│  1. Determine scope                                 │
│  2. Spawn qa-test-auditor agent (analysis)          │
│  3. Present findings to user (numbered list)        │
│  4. User selects which to act on                    │
│  5. Implement selected changes (SME or direct)      │
│  6. Verify (run test suite)                         │
│  7. Summary                                         │
└─────────────────────────────────────────────────────┘
```

## Workflow Details

### 1. Determine Scope

**Ask the user:** "What tests should I audit?"

Present these options:
- **All tests**: Review every test file in the project
- **Specific directory**: A path like `tests/`, `src/__tests__/`, `*_test.go`
- **Specific files**: Individual test files
- **Recent changes**: Tests added or modified in the current branch (via `git diff`)

**Default:** All tests.

**If the project is large** (many test files), suggest narrowing scope to keep findings manageable. The user can always run the audit again on a different scope.

### 2. Scan for Quality Issues

**First, assess the scope size.** Use Glob to count test files in scope.

#### Small scope (roughly ≤15 test files)

Spawn a single `qa-test-auditor` agent with the full scope.

#### Large scope (roughly >15 test files)

Partition the scope by directory or module (e.g., `tests/auth/`, `tests/api/`, `tests/models/`). Spawn multiple `qa-test-auditor` agents **in parallel**, each with a focused partition.

After all agents complete, merge their findings into a single list. Deduplicate if any findings overlap at partition boundaries.

**Note:** Parallel scanning trades away some cross-file redundancy detection (an agent reviewing `tests/auth/` can't spot redundancy with `tests/api/`). This is an acceptable tradeoff — most redundancy is within a module.

#### Prompt for each agent

```
Review the test suite for quality issues.
Scope: [partition or full scope]

Look for:
- Tautological tests (can't fail)
- Brittle tests (coupled to implementation)
- Redundant tests (duplicate coverage)
- False confidence tests (don't verify what they claim)
- Missing coverage (important gaps only)
- Test smells (structural problems)

Return structured findings with recommended actions (DELETE, REWRITE, ADD, SIMPLIFY).
```

**If no agent reports issues:** Present "No test quality issues found" to user and exit.

### 3. Present Findings

Display the agent's findings as a numbered list, grouped by category. Format clearly so the user can scan and select.

**Example presentation:**

```
## Test Audit Findings

### Tautological (3 found)
1. [DELETE] user_test.go:TestNewUser — Asserts constructor sets fields; tests Go assignment, not logic
2. [DELETE] config_test.go:TestDefaultConfig — Asserts default values match hardcoded expectations
3. [DELETE] model_test.go:TestUserStruct — Checks struct field existence

### Brittle (2 found)
4. [REWRITE] api_test.go:TestCreateUserError — Exact error string match; should test error type
5. [REWRITE] handler_test.go:TestNotFound — Asserts on full JSON response body; should check status + key fields

### Missing Coverage (1 found)
6. [ADD] auth.go:ValidateToken — No tests for expired/malformed tokens; only tests happy path

Select which items to address (e.g., "1-5, 6" or "all"):
```

**Use `AskUserQuestion`** with multi-select to let the user choose. If there are more than ~10 findings, batch them by category and present in rounds.

### 4. Implement Selected Changes

For each selected recommendation, group by action type and implement:

#### DELETE actions
Handle directly as orchestrator:
- Remove the test function/block
- If removing the last test in a file, delete the file
- If removing a test invalidates imports/setup, clean those up too

#### REWRITE / SIMPLIFY actions
**Detect appropriate SME and spawn based on project language:**
- Go: `swe-sme-golang`
- Dockerfile: `swe-sme-docker`
- Makefile: `swe-sme-makefile`
- GraphQL: `swe-sme-graphql`
- Ansible: `swe-sme-ansible`
- Zig: `swe-sme-zig`

**For languages without a dedicated SME:** implement directly as orchestrator.

**Prompt the SME with:**
```
Rewrite/simplify the following tests based on these findings:
[List of selected REWRITE/SIMPLIFY items with rationale]

Focus on testing observable behavior rather than implementation details.
Follow the project's existing test conventions.
Keep tests simple and readable.
```

#### ADD actions (missing coverage)
**Spawn appropriate SME agent** (same detection as above).

**Prompt the SME with:**
```
Add the following missing tests:
[List of selected ADD items with description of what to verify]

Write focused tests that verify behavior, not implementation.
Follow the project's existing test conventions.
Keep tests minimal — test one thing per test.
```

### 5. Verify

**Run the test suite** to confirm:
- Remaining tests still pass (deletions didn't break anything that depended on removed test infrastructure)
- Rewritten tests pass
- New tests pass

**If failures:**
- Report which tests failed and why
- For failures in rewritten tests: attempt one fix, then report to user
- For failures in unrelated tests: report to user (the deletion may have removed shared setup)
- Let user decide how to proceed

### 6. Summary

Present what was changed:

```
## Test Audit Complete

### Changes
- Tests deleted: N
- Tests rewritten: N
- Tests added: N
- Net test count change: +/-N

### Files Modified
- [file]: [what changed]

### Skipped (user declined)
- N findings not addressed
```

**Ask user if they want to commit.** If yes, create a commit:

```bash
git add [specific files]
git commit -m "$(cat <<'EOF'
test: audit and clean up test suite

[Brief description: deleted N tautological/brittle tests, rewrote N tests
to focus on behavior, added N missing coverage tests]

Co-Authored-By: Claude Opus 4.6 <noreply@anthropic.com>
EOF
)"
```

## Agent Coordination

**Sequential execution:**
- One agent at a time
- Wait for completion before spawning next

**Fresh instances:**
- Spawn fresh `qa-test-auditor` for each scan
- Spawn fresh SME for implementation

**State to maintain (as orchestrator):**
- User's selected findings
- Implementation results (success/failure per finding)
- Running totals for summary

## Abort Conditions

**Abort workflow:**
- User interrupts
- No test files found in scope
- Agent finds no issues (exit gracefully, not an error)

**Do NOT abort for:**
- Individual implementation failures (report and continue)
- Test suite failures after changes (report and let user decide)

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

### Redundant (1 found)
6. [DELETE] math_test.go:TestAddVariants — 5 cases all hitting same code path

### Missing Coverage (1 found)
7. [ADD] auth.go:ValidateToken — No tests for expired/malformed tokens

Select which items to address:
> 1-5, 7

Deleting 4 tests (items 1-3, 6)...
Spawning swe-sme-golang for rewrites (items 4-5) and new tests (item 7)...
Implementation complete.
Running test suite...
All tests pass.

## Test Audit Complete

### Changes
- Tests deleted: 4
- Tests rewritten: 2
- Tests added: 3 (happy + 2 error cases for ValidateToken)
- Net test count change: +1

Commit these changes?
> Yes

Committed: "test: audit and clean up test suite"
```
