---
name: test-audit
description: Audit test suite quality. Identifies brittle, tautological, and otherwise harmful tests, then selectively fixes or removes them. Reports redundant tests as informational.
model: opus
---

# Test Audit — Interactive Test Quality Review

Reviews test suite quality, presents findings for user selection, and implements chosen fixes. Focuses on identifying tests that are worse than useless — tests that create false confidence, break on every refactor, or test nothing meaningful.

## Philosophy

**Tests exist to catch bugs and prevent regressions.** A test that literally can't fail or tests the wrong thing is harmful. But a simple test that guards against real changes has value — even if it looks trivial. Prefer fixing bad tests over deleting them. Only delete a test when it is genuinely self-fulfilling (structurally cannot fail) or completely orphaned from real code.

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
- Redundant tests (duplicate coverage — informational only, no action recommended)
- False confidence tests (don't verify what they claim)
- Missing coverage (important gaps only)
- Test smells (structural problems)

Return structured findings with recommended actions (DELETE, REWRITE, ADD, SIMPLIFY).
Redundant tests should be reported as informational only (no action recommended).
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

### Redundant (1 noted — informational, no action)
- [INFO] math_test.go:TestAddVariants — 5 cases all hitting same code path; consider adding edge-case tests

### Missing Coverage (1 found)
6. [ADD] auth.go:ValidateToken — No tests for expired/malformed tokens; only tests happy path

Select which items to address (e.g., "1-6" or "all"):
```

**Use `AskUserQuestion`** with multi-select to let the user choose. If there are more than ~10 findings, batch them by category and present in rounds.

### 4. Implement Selected Changes

**Detect appropriate SME and spawn based on project language:**
- Go: `swe-sme-golang`
- Dockerfile: `swe-sme-docker`
- Makefile: `swe-sme-makefile`
- GraphQL: `swe-sme-graphql`
- Ansible: `swe-sme-ansible`
- Zig: `swe-sme-zig`

**For languages without a dedicated SME:** implement directly as orchestrator.

#### Parallelization

**Group findings by file**, then spawn one SME agent per file **in parallel**. Findings affecting the same file must go to the same agent to avoid edit conflicts. Findings in different files are independent and should always run concurrently.

- If all findings are in one file, spawn a single agent.
- If findings span N files, spawn up to N agents in parallel, one per file.
- ADD findings (new test files): group by the target test file. If adding tests to an existing file that also has REWRITE/DELETE findings, bundle them together. If adding a new file, that's its own agent.

The SME has domain expertise to judge whether a deletion is appropriate or whether the test should be rewritten instead. The SME has final authority on deletions.

**Prompt each SME agent with:**
```
The test auditor identified the following issues in [file]. Implement the recommended changes.

DELETE findings (remove these tests — but if you believe a test has value, rewrite it instead of deleting):
[List of DELETE items for this file]

REWRITE/SIMPLIFY findings (fix these tests):
[List of REWRITE/SIMPLIFY items for this file]

ADD findings (write new tests):
[List of ADD items for this file]

Guidelines:
- Focus on testing observable behavior rather than implementation details.
- Follow the project's existing test conventions.
- Keep tests simple and readable.
- For DELETE items: if the test covers real behavior that could regress, rewrite it rather than deleting it. Only delete tests that are genuinely self-fulfilling or completely orphaned.
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

Co-Authored-By: Claude <noreply@anthropic.com>
EOF
)"
```

## Agent Coordination

**Analysis phase:** Sequential. Spawn `qa-test-auditor` agent(s) for scanning. For large scopes, partition and run auditors in parallel (see step 2).

**Implementation phase:** Parallel by file. Group findings by file, spawn one SME per file, run all SME agents concurrently. Wait for all to complete before proceeding to verification.

**Fresh instances:**
- Spawn fresh `qa-test-auditor` for each scan partition
- Spawn fresh SME for each file

**State to maintain (as orchestrator):**
- User's selected findings
- Implementation results (success/failure per agent/file)
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

### Redundant (1 found — informational, no action)
6. [INFO] math_test.go:TestAddVariants — 5 cases all hitting same code path

### Missing Coverage (1 found)
7. [ADD] auth.go:ValidateToken — No tests for expired/malformed tokens

Select which items to address:
> 1-5, 7

Findings span 5 files. Spawning 5 swe-sme-golang agents in parallel...
  - user_test.go: 1 finding (DELETE)
  - config_test.go: 1 finding (DELETE)
  - model_test.go: 1 finding (DELETE)
  - api_test.go: 1 finding (REWRITE)
  - handler_test.go: 1 finding (REWRITE)
  + auth_test.go: 1 finding (ADD)
All agents complete.
Running test suite...
All tests pass.

## Test Audit Complete

### Changes
- Tests deleted: 3
- Tests rewritten: 2
- Tests added: 3 (happy + 2 error cases for ValidateToken)
- Net test count change: +2

Commit these changes?
> Yes

Committed: "test: audit and clean up test suite"
```
