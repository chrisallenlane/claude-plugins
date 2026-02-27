# /fix - Automated Bug-Fixing Workflow

## Overview

The `/fix` skill orchestrates a diagnosis-first bug-fixing workflow through specialist agents. It reproduces the bug with a failing test, performs root-cause analysis with git archaeology, implements a targeted fix, and verifies the fix through practical testing — all with the same review and documentation quality gates as `/iterate`.

**Key benefits:**
- Test-driven reproduction: a failing test defines "done"
- Root-cause analysis prevents surface-level patches
- Git archaeology uncovers when and why the bug was introduced
- Related failure modes are identified and tested, not just the reported bug
- Practical verification confirms the fix actually works
- Same review pipeline as `/iterate` (security, refactoring, performance, peer review)

## When to Use

**Use `/fix` for:**
- Bugs that need investigation before fixing
- Issues where you want a failing test before any code changes
- Bugs that might have related failure modes (same root cause, similar patterns)
- Fixes where you want comprehensive regression testing
- Problems where git history might reveal how the bug was introduced

**Don't use `/fix` for:**
- Obvious typos or trivial one-line fixes (just fix them directly)
- Feature requests disguised as bugs (use `/iterate`)
- Exploratory debugging where the symptom is unclear (investigate first, then `/fix`)

**Rule of thumb:** If the bug is worth a failing test and root-cause analysis, use `/fix`. If the fix is obvious, just do it.

## Workflow Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│ /fix Workflow                                                    │
└─────────────────────────────────────────────────────────────────┘

 ┌──────────────────────────────────────────────┐
 │  1. CLARIFY THE BUG                          │
 │  ────────────────────────────────────────    │
 │  • Symptoms, expected vs actual behavior     │
 │  • Reproduction steps                        │
 │  • When did it start? Recent changes?        │
 │  • Environment specifics                     │
 │  • Loop until bug is clearly understood      │
 └──────────────────┬───────────────────────────┘
                    ▼
 ┌──────────────────────────────────────────────┐
 │  2. WRITE FAILING TEST(S)                    │
 │  ────────────────────────────────────────    │
 │  Agent: Language-specific SME                │
 │                                              │
 │  • Write test encoding expected behavior     │
 │  • Verify test actually fails (proves bug)   │
 │  • This is the contract: when it passes,     │
 │    the bug is fixed                          │
 │                                              │
 │  Can't reproduce? ──┬─ Back to step 1       │
 │                     └─ (max 2 attempts)      │
 └──────────────────┬───────────────────────────┘
                    ▼
 ┌──────────────────────────────────────────────┐
 │  3. DIAGNOSIS                                │
 │  ────────────────────────────────────────    │
 │  Agent: swe-diagnostician (read-only)        │
 │                                              │
 │  • Trace execution paths                     │
 │  • Git archaeology (depth at discretion):    │
 │    - Shallow: git log on involved files      │
 │    - Medium: git blame, commit context       │
 │    - Deep: git log -S, pattern search        │
 │  • Identify root cause + evidence            │
 │  • Identify related failure modes            │
 │  • Recommend fix approach                    │
 └──────────────────┬───────────────────────────┘
                    ▼
 ┌──────────────────────────────────────────────┐
 │  4. IMPLEMENT THE FIX                        │
 │  ────────────────────────────────────────    │
 │  Agent: Same SME as step 2                   │
 │                                              │
 │  • Follow diagnostician's recommended fix    │
 │  • Write tests for related failure modes     │
 │  • Verify: originally-failing test passes    │
 │  • Run full test suite for regressions       │
 └──────────────────┬───────────────────────────┘
                    ▼
 ┌──────────────────────────────────────────────┐
 │  5. QA - VERIFY FIX                          │
 │  ────────────────────────────────────────    │
 │  Agent: qa-engineer (Mode 1)                 │
 │                                              │
 │  CRITICAL GATE:                              │
 │  1. Practical verification (reproduce the    │
 │     original scenario, confirm it's fixed)   │
 │  2. Run full test suite                      │
 │  3. Check related failure modes              │
 │                                              │
 │  Passes? ──┬─ Yes → Continue                 │
 │            └─ No  → Back to step 4 ──┐       │
 │                     (max 3 attempts) │       │
 └──────────────────┬───────────────────┘       │
                    ▼                           │
 ┌──────────────────────────────────────────────┤
 │  6. CODE REVIEW (conditional reviewers)      │
 │  ────────────────────────────────────────    │
 │  Only review working code!                   │
 │                                              │
 │  6a. Security (if sensitive code)            │
 │      Agent: sec-reviewer                     │
 │      Authority: Can demand changes           │
 │                                              │
 │  6b. Refactoring (if non-trivial)            │
 │      Agent: swe-refactor                     │
 │      Authority: Advisory suggestions         │
 │                                              │
 │  6c. Performance (if critical code)          │
 │      Agent: swe-perf-engineer                │
 │      Authority: Advisory suggestions         │
 └──────────────────┬───────────────────────────┘
                    ▼
 ┌──────────────────────────────────────────────┐
 │  7. IMPLEMENT REVIEW FEEDBACK                │
 │  ────────────────────────────────────────    │
 │  Agent: Same SME as steps 2/4                │
 │                                              │
 │  • Aggregate all review feedback             │
 │  • Security: Must address or get approval    │
 │  • Other: Advisory - use discretion          │
 └──────────────────┬───────────────────────────┘
                    ▼
 ┌──────────────────────────────────────────────┐
 │  8. SME PEER REVIEW (conditional)            │
 │  ────────────────────────────────────────    │
 │  Agent: Fresh instance of same SME           │
 │                                              │
 │  Non-trivial changes?                        │
 │  ├─ Yes → Fresh SME reviews, makes nit fixes │
 │  └─ No  → Skip to QA                         │
 └──────────────────┬───────────────────────────┘
                    ▼
 ┌──────────────────────────────────────────────┐
 │  9. QA - COVERAGE & QUALITY                  │
 │  ────────────────────────────────────────    │
 │  Agent: qa-engineer (Mode 2)                 │
 │                                              │
 │  • Test coverage analysis                    │
 │  • Fill coverage gaps                        │
 │  • Run linters/formatters                    │
 │  • Verify steps 7-8 didn't break tests       │
 └──────────────────┬───────────────────────────┘
                    ▼
 ┌──────────────────────────────────────────────┐
 │  10. DOCUMENTATION                           │
 │  ────────────────────────────────────────    │
 │  Agent: doc-maintainer                       │
 │                                              │
 │  • Update affected documentation             │
 │  • Verify code examples                      │
 │  • Check for broken links                    │
 └──────────────────┬───────────────────────────┘
                    ▼
 ┌──────────────────────────────────────────────┐
 │  11. FINAL VERIFICATION                      │
 │  ────────────────────────────────────────    │
 │  • Run full test suite                       │
 │  • Run all linters                           │
 │  • Present summary: root cause, fix, tests   │
 └──────────────────┬───────────────────────────┘
                    ▼
 ┌──────────────────────────────────────────────┐
 │  12. WORKFLOW COMPLETION (optional)          │
 │  ────────────────────────────────────────    │
 │  • Commit with root-cause in message         │
 │  • Update/close ticket (if applicable)       │
 │  • Rebase on main (if applicable)            │
 └──────────────────────────────────────────────┘
```

## How /fix Differs from /iterate

The `/fix` workflow diverges from `/iterate` in steps 2-4, then rejoins for the review pipeline:

| Step | `/iterate` | `/fix` |
|------|-----------|--------|
| 1 | Gather requirements | Clarify bug symptoms |
| 2 | Planning (conditional) | **Write failing test** (SME) |
| 3 | Implementation | **Diagnosis** (swe-diagnostician) |
| 4 | QA acceptance gate | **Implement fix** (guided by diagnosis) |
| 5+ | Reviews, docs, commit | Same as `/iterate` steps 5-11 |

Key differences:
- **Test first, then fix.** `/iterate` implements then tests. `/fix` writes a failing test before any fix attempt.
- **Diagnosis before implementation.** The diagnostician performs root-cause analysis so the SME implements a targeted fix, not a guess.
- **Git archaeology.** The diagnostician traces when and how the bug was introduced, providing context the SME wouldn't otherwise have.
- **Related failure modes.** The diagnostician identifies patterns, and the SME writes tests for all of them — not just the reported bug.

## Workflow Details

### 1. Clarify the Bug
The workflow starts by ensuring the bug is clearly understood:
- Symptoms (error messages, crashes, unexpected behavior)
- Expected vs actual behavior
- Reproduction steps
- When it started, any recent changes
- Environment specifics

Loops until there's enough information to write a reproducing test.

### 2. Write Failing Test(s)
A language-specific SME writes test(s) that encode the expected behavior:
- Tests fail against the current buggy code (proving the bug is real and reproducible)
- This is the "contract" — when these tests pass, the bug is fixed
- If the bug can't be reproduced after 2 attempts, escalates to user

**Specialists available:**
- `swe-sme-golang` - Go projects
- `swe-sme-graphql` - GraphQL schemas/resolvers
- `swe-sme-docker` - Dockerfiles and containers
- `swe-sme-makefile` - Makefiles and build systems

### 3. Diagnosis
The `swe-diagnostician` agent performs read-only root-cause analysis:
- **Traces execution paths** from the failing test through the code
- **Git archaeology** at its own discretion:
  - Shallow: `git log` on involved files for recent changes
  - Medium: `git blame` on suspicious code, commit context
  - Deep: `git log -S` for pattern search, similar bugs elsewhere
- **Identifies root cause** with supporting evidence
- **Identifies related failure modes** (same root cause affecting other paths)
- **Recommends fix approach** (what to change and why, not specific code)

The diagnostician does not modify any code. It produces a diagnosis report that guides the implementation agent.

### 4. Implement the Fix
The same SME from step 2 implements the fix:
- Follows the diagnostician's recommended approach
- Fixes the bug as narrowly and precisely as possible
- Writes additional tests for each related failure mode
- Verifies the originally-failing test now passes
- Runs the full test suite for regressions

### 5. QA - Verify Fix (CRITICAL GATE)
The `qa-engineer` performs practical verification:
- Actually reproduces the originally-reported scenario and confirms it's fixed
- Runs the full test suite
- Checks related failure modes identified by the diagnostician

If verification fails, returns to step 4 (max 3 attempts before escalating).

### 6-12. Reviews, Documentation, Completion
Identical to `/iterate` steps 5-11. See the [/iterate README](../iterate/README.md) for details.

## Examples

### Example 1: Off-by-One in Pagination
```
User: "The last item on each page is duplicated on the next page"

Workflow:
1. Clarify: Which endpoint? Happens with all page sizes? Started after recent changes?
2. Failing test: swe-sme-golang writes TestPaginationBoundary that asserts
   no duplicates — test fails against current code
3. Diagnosis: swe-diagnostician traces pagination logic, finds boundary
   comparison uses < instead of <=, git blame shows it was introduced
   in commit abc123 during a refactor. Also identifies: same pattern
   exists in the cursor-based pagination path.
4. Fix: swe-sme-golang fixes both comparison operators, adds tests for
   cursor pagination boundary too
5. QA: qa-engineer actually paginates through test data, confirms no dupes
6-12. Reviews, docs, commit

Summary:
- Root cause: < vs <= in boundary check, introduced in refactor abc123
- Fixed: 2 files, both offset and cursor pagination
- Tests added: 4 (original bug + 3 related failure modes)
```

### Example 2: Race Condition in Cache
```
User: "Intermittent 500 errors under load, started after adding caching"

Workflow:
1. Clarify: Which endpoint? Error logs? Frequency? Load characteristics?
2. Failing test: swe-sme-golang writes TestConcurrentCacheAccess using
   goroutines — test fails with data race detector
3. Diagnosis: swe-diagnostician finds unsynchronized map access in cache,
   git log shows cache was added without mutex protection. Related: the
   cache eviction path has the same issue.
4. Fix: Adds sync.RWMutex to cache, protects both read/write and eviction paths
5. QA: Runs concurrent test, race detector clean

Summary:
- Root cause: Unsynchronized map access in cache layer
- Fixed: 1 file (cache.go), added RWMutex protection
- Tests added: 3 (concurrent read/write, concurrent eviction, mixed load)
```

### Example 3: Can't Reproduce
```
User: "App crashes on startup"

Workflow:
1. Clarify: Error message? OS? Configuration file? Recent updates?
2. Failing test: swe-sme-golang tries to reproduce — test passes (no crash).
   Reports: tested with default config, various invalid configs, missing
   config file. Cannot reproduce.
   → Back to step 1.
   User provides: "Only happens with YAML config, not TOML"
2. (retry): swe-sme-golang writes TestYAMLConfigParsing — fails! Bug reproduced.
3-12. Continue with diagnosis and fix...
```

## Tips

1. **Provide reproduction details.** The more specific your reproduction steps, the faster step 2 goes. Vague reports lead to reproduction failures.

2. **Trust the diagnostician's depth.** It chooses shallow or deep git archaeology based on the bug's complexity. Don't worry about it spending too much time on simple bugs.

3. **The failing test is the contract.** Once the SME writes a failing test, that test defines "fixed." This is more reliable than manual verification alone.

4. **Related failure modes are valuable.** The diagnostician looks for patterns, not just the single reported bug. This often catches adjacent bugs that would have been reported next.

5. **Use `/iterate` for feature-shaped bugs.** If the "bug" is really a missing feature or a design change, `/iterate` is a better fit. `/fix` is for when existing behavior is wrong.

## Agent Coordination

**Sequential execution:** One agent at a time, each completes before the next starts.

**SME continuity:** The same SME type is used across steps 2 (failing test), 4 (fix), and 7 (review feedback). Step 8 (peer review) uses a fresh instance for independent perspective.

**Diagnosis pass-through:** The diagnostician's report from step 3 is passed to the SME in step 4. The SME follows the recommended approach and writes tests for each identified related failure mode.

## Iteration Limits

**Safety limits:**
- Reproduction loop (step 2): 2 attempts max
- Fix verification loop (step 5): 3 attempts max
- Overall workflow: 12 agent spawns max

If limits reached, workflow escalates to user with current state and specific blockers.
