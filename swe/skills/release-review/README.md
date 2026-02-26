# /release-review - Pre-Release Readiness Check

## Overview

The `/release-review` skill performs a comprehensive pre-flight check before cutting a release. It spawns a scanner agent for fast static analysis, then runs execution-based checks (tests, build, doc freshness), and presents all findings interactively for human review.

**Key benefits:**
- Fast feedback first — static analysis runs before expensive test/build checks
- Interactive — every finding is presented for human decision
- Structured severity levels (BLOCKER/WARNING/INFO) to prioritize attention
- Reuses existing workflows (doc-maintainer for freshness checks)
- Consolidated report with actionable recommendations

## When to Use

**Use `/release-review` for:**
- Preparing to tag and release a new version
- Final quality gate before shipping to users
- Validating that a codebase is ready for distribution
- Catching things that development workflows don't focus on

**Don't use `/release-review` for:**
- Routine development checks (use `/iterate` or `/refactor`)
- Test quality concerns (use `/test-audit` or `/test-mutate`)
- Documentation updates (use `/doc-review`)
- Security audits (the sec-reviewer agent handles that during `/iterate`)

**Key principle:** Releases deserve human review. This workflow surfaces issues — it doesn't silently fix them.

## What It Checks

| Check | What it looks for | Severity |
|-------|-------------------|----------|
| **Debug artifacts** | console.log, print(), breakpoint(), debugger statements | BLOCKER |
| **Work markers** | TODO, FIXME, HACK, XXX, REMOVEME | WARNING |
| **Hardcoded URLs** | localhost, 127.0.0.1 in production code | WARNING |
| **Version consistency** | Mismatched versions across manifests and source | BLOCKER |
| **Changelog coverage** | Changelog not updated since last tag | WARNING |
| **Git hygiene** | Merge conflict markers, tracked secrets, dirty working tree | BLOCKER |
| **Breaking changes** | Removed public API without changelog mention | BLOCKER |
| **License compliance** | New deps with incompatible or unknown licenses | BLOCKER/WARNING |
| **Test suite** | Test failures | BLOCKER |
| **Build verification** | Build failures | BLOCKER |
| **Doc freshness** | Documentation referencing removed/renamed code | WARNING |

## Workflow Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│ /release-review Workflow                                          │
└─────────────────────────────────────────────────────────────────┘

 ┌──────────────────────────────────────────────┐
 │  1. DETERMINE RELEASE CONTEXT                │
 │  ────────────────────────────────────────    │
 │  • Target version (optional)                 │
 │  • Detect last tag, test/build commands      │
 │  • Any checks to skip?                       │
 └──────────────────┬───────────────────────────┘
                    ▼
 ┌──────────────────────────────────────────────┐
 │  2. STATIC ANALYSIS (fast)                   │
 │  ────────────────────────────────────────    │
 │  Agent: qa-release-eng                       │
 │                                              │
 │  Scans for:                                  │
 │  • Debug/dev artifacts                       │
 │  • Version consistency                       │
 │  • Changelog coverage                        │
 │  • Git hygiene                               │
 │  • Breaking changes                          │
 │  • License compliance                        │
 └──────────────────┬───────────────────────────┘
                    ▼
 ┌──────────────────────────────────────────────┐
 │  3. PRESENT PHASE 1 FINDINGS                 │
 │  ────────────────────────────────────────    │
 │  Show BLOCKERs and WARNINGs from scan        │
 │                                              │
 │  Ask: Continue with execution checks?        │
 │  ├─ Continue → proceed to step 4             │
 │  ├─ Fix first → exit, user fixes, re-runs   │
 │  └─ Skip → go to step 7                     │
 └──────────────────┬───────────────────────────┘
                    ▼
 ┌──────────────────────────────────────────────┐
 │  4-6. EXECUTION CHECKS (slow)                │
 │  ────────────────────────────────────────    │
 │  4. Run test suite                           │
 │  5. Run build verification                   │
 │  6. Check doc freshness (doc-maintainer)     │
 └──────────────────┬───────────────────────────┘
                    ▼
 ┌──────────────────────────────────────────────┐
 │  7. FULL CONSOLIDATED REPORT                 │
 │  ────────────────────────────────────────    │
 │  All findings numbered by severity:          │
 │  BLOCKERS → WARNINGS → PASSED               │
 │                                              │
 │  User selects which items to address         │
 └──────────────────┬───────────────────────────┘
                    ▼
 ┌──────────────────────────────────────────────┐
 │  8. IMPLEMENT SELECTED FIXES                 │
 │  ────────────────────────────────────────    │
 │  Auto-fixable:                               │
 │  • Debug artifact removal                    │
 │  • TODO/FIXME marker removal                 │
 │  • Version number updates                    │
 │                                              │
 │  Report-only:                                │
 │  • Merge conflicts (manual resolution)       │
 │  • Test/build failures (investigate)         │
 │  • Changelog gaps (scaffold offered)         │
 │  • License issues (user decides)             │
 │  • Doc staleness (suggest /doc-review)       │
 │  • Breaking changes (document or revert)     │
 └──────────────────┬───────────────────────────┘
                    ▼
 ┌──────────────────────────────────────────────┐
 │  9. RE-VERIFY                                │
 │  ────────────────────────────────────────    │
 │  Quick re-check of affected static checks    │
 └──────────────────┬───────────────────────────┘
                    ▼
 ┌──────────────────────────────────────────────┐
 │  10. FINAL SUMMARY                           │
 │  ────────────────────────────────────────    │
 │  • Items resolved vs remaining               │
 │  • Changes made                              │
 │  • Recommendation:                           │
 │    HOLD / PROCEED WITH CAUTION / READY       │
 │  • Offer to commit changes                   │
 └──────────────────────────────────────────────┘
```

## Examples

### Example 1: Clean Release

```
User: /release-review

What version are you releasing?
> 3.1.0

Any checks to skip?
> Run everything

Spawning qa-release-eng agent...

## Release Readiness: Static Analysis

Last tag: v3.0.0 → Target: v3.1.0
12 commits, 8 files changed

### PASSED
- Debug artifacts: None found
- Version consistency: All manifests agree (3.1.0)
- Changelog: Updated with v3.1.0 section
- Git hygiene: Clean working tree, no issues
- Breaking changes: None detected
- License compliance: No new dependencies

Continue with execution checks?
> Continue

Running tests... All pass. ✓
Running build... Clean build. ✓
Checking docs... Up to date. ✓

## Release Readiness Report

No BLOCKERS or WARNINGS found.

### PASSED
- All checks pass

### Recommendation
READY TO RELEASE
```

### Example 2: Issues Found

```
User: /release-review

What version?
> 2.0.0

Spawning qa-release-eng agent...

## Release Readiness: Static Analysis

Last tag: v1.9.2 → Target: v2.0.0
89 commits, 42 files changed

### BLOCKERS (2)
1. [DEBUG] src/api/handler.go:15 — fmt.Println("debug: request body", body)
2. [VERSION] All manifests say 1.9.2 (target is 2.0.0)

### WARNINGS (3)
3. [CHANGELOG] Not updated since v1.9.2
4. [BREAKING] Removed exported function ParseConfig()
5. [DEBUG] TODO markers in 2 files

Continue with execution checks?
> Continue

Running tests... All pass. ✓
Running build... Clean build. ✓
Checking docs... README.md references removed ParseConfig().

## Full Report

### BLOCKERS (2)
1. [DEBUG] src/api/handler.go:15 — debug statement
2. [VERSION] Not bumped to 2.0.0

### WARNINGS (4)
3. [CHANGELOG] Not updated
4. [BREAKING] ParseConfig() removed, not in changelog
5. [DEBUG] TODO markers in 2 files
6. [DOCS] README references removed function

Select items to address:
> 1, 2, 5

[Fixes applied]

## Release Review Complete
- BLOCKERS resolved: 2/2
- WARNINGS addressed: 1/4
- Recommendation: PROCEED WITH CAUTION
```

### Example 3: Fix Blockers First

```
Spawning qa-release-eng agent...

### BLOCKERS (3)
1. [GIT] src/auth.go:42 — Merge conflict markers
2. [GIT] .env file tracked by git
3. [TESTS] (not yet run)

Continue with execution checks?
> Fix blockers first

Understood. Resolve the merge conflict in src/auth.go and
remove .env from tracking, then re-run /release-review.
```

## Tips for Effective Use

1. **Run on a clean branch.** The release review works best when your release branch is ready. Uncommitted work adds noise.

2. **Provide the target version.** This enables version consistency checking against a known target, not just checking that versions agree with each other.

3. **Fix blockers before execution checks.** If the static analysis finds obvious blockers, use the "Fix blockers first" option to save time.

4. **Run `/doc-review` for stale docs.** The release review only flags staleness — it doesn't fix documentation. Run `/doc-review` separately to update docs.

5. **Review the changelog manually.** The review can detect that the changelog wasn't updated, and can scaffold entries from git history, but meaningful release notes require human authorship.

6. **Run periodically, not just at release time.** Running occasionally during development catches issues early, when they're easier to fix.

7. **Don't ignore warnings.** PROCEED WITH CAUTION means exactly that — review the remaining warnings and make a conscious decision about each one.
