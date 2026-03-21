# /refactor - Iterative Code Quality Improvement

## Overview

The `/refactor` skill autonomously improves code quality within the existing architecture. It iteratively scans for tactical improvements (DRY violations, dead code, naming issues, unnecessary complexity), implements them through specialist agents with QA verification, and loops until no further opportunities remain.

**Key benefits:**
- Autonomous operation - set it loose and let it work
- Gradient aggression - always makes the least aggressive change available first
- Fresh agent instances each pass (prevents context accumulation)
- Atomic commits per batch (easy to review, bisect, or revert)
- Built-in quality gates with QA verification
- Cascading improvements - rescans catch what previous changes unlocked

## When to Use

**Use `/refactor` for:**
- Cleaning up accumulated technical debt
- After a major feature is complete and you want to tidy up
- Routine code quality improvement
- DRY violations, dead code, naming stutter, unnecessary complexity
- When you want a quick, low-risk cleanup pass

**Don't use `/refactor` for:**
- Rethinking module boundaries or system architecture (use `/review-arch`)
- Module dissolution, creation, or reorganization (use `/review-arch`)
- Quick one-off fixes (just do them directly)
- Codebases without tests (refactoring needs verification)

**Rule of thumb:** Use `/refactor` when the code within modules needs cleaning up. Use `/review-arch` when the module structure itself needs rethinking.

## Workflow Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│ /refactor Workflow                                              │
└─────────────────────────────────────────────────────────────────┘

 ┌──────────────────────────────────────────────┐
 │  1. DETERMINE SCOPE                          │
 │  ────────────────────────────────────────    │
 │  • Default: Entire codebase                  │
 │  • Or: User-specified path/module            │
 │  • Boundary: git-tracked files only          │
 └──────────────────┬───────────────────────────┘
                    ▼
 ┌──────────────────────────────────────────────┐
 │  2. SELECT AGGRESSION CEILING                │
 │  ────────────────────────────────────────    │
 │  How far up the ladder to climb:             │
 │  • Maximum: All improvements                 │
 │  • High: Up to MODERATE changes              │
 │  • Low: SAFEST and SAFE only                 │
 └──────────────────┬───────────────────────────┘
                    ▼
 ┌──────────────────────────────────────────────┐
 │  3. GATHER QA INSTRUCTIONS                   │
 │  ────────────────────────────────────────    │
 │  Custom verification steps (optional)        │
 └──────────────────┬───────────────────────────┘
                    ▼
        ┌───────────────────────┐
        │  REFACTORING LOOP     │◄──────────────────┐
        └───────────┬───────────┘                   │
                    ▼                               │
 ┌──────────────────────────────────────────────┐   │
 │  4. SCAN FOR OPPORTUNITIES                   │   │
 │  ────────────────────────────────────────    │   │
 │  Agent: swe-code-reviewer (fresh instance)        │   │
 │                                              │   │
 │  Returns recommendations by risk level:      │   │
 │  SAFEST → SAFE → MODERATE → AGGRESSIVE       │   │
 │                                              │   │
 │  No opportunities? → EXIT ──────────────► DONE   │
 └──────────────────┬───────────────────────────┘   │
                    ▼                               │
 ┌──────────────────────────────────────────────┐   │
 │  5. SELECT & IMPLEMENT                       │   │
 │  ────────────────────────────────────────    │   │
 │  Pick least aggressive changes available     │   │
 │  Batch related changes together              │   │
 │  Agent: Language-specific SME or generalist  │   │
 └──────────────────┬───────────────────────────┘   │
                    ▼                               │
 ┌──────────────────────────────────────────────┐   │
 │  6. VERIFY CHANGES                           │   │
 │  ────────────────────────────────────────    │   │
 │  Agent: qa-engineer                          │   │
 │                                              │   │
 │  Passes? ──┬─ Yes → Commit ──────────────────┤   │
 │            └─ No  → Return to SME ──┐        │   │
 │                     (max 3 attempts)│        │   │
 │                                     ▼        │   │
 │                     ┌────────────────────┐   │   │
 │                     │ Still failing?     │   │   │
 │                     │ → Revert batch     │   │   │
 │                     │ → Log failure      │   │   │
 │                     └──────────┬─────────┘   │   │
 │                                │             │   │
 └────────────────────────────────┼─────────────┘   │
                                  ▼                 │
                     Rescan with fresh agent ───────┘

 ┌──────────────────────────────────────────────┐
 │  7. COMPLETION SUMMARY                       │
 │  ────────────────────────────────────────    │
 │  • Total commits, net lines changed          │
 │  • Batches completed vs aborted              │
 │  • Changes by category                       │
 │  • User-decision items (commented-out code,  │
 │    unused public APIs)                       │
 └──────────────────┬───────────────────────────┘
                    ▼
 ┌──────────────────────────────────────────────┐
 │  8. UPDATE DOCUMENTATION                     │
 │  ────────────────────────────────────────    │
 │  Run /review-doc to fix stale docs           │
 └──────────────────────────────────────────────┘
```

## Workflow Details

### 1. Determine Scope
By default, the workflow operates on the entire codebase. You can specify a narrower scope:

```
/refactor                     # Entire codebase
/refactor src/handlers/       # Just the handlers
/refactor *.go                # Just Go files
```

The scope is passed to all spawned agents.

**Important:** The workflow operates exclusively on files tracked by git. Untracked files and directories are never modified or deleted — their loss could be irreversible.

### 2. Select Aggression Ceiling
The workflow asks how far up the aggression ladder to climb:

- **Maximum**: All improvements, including aggressive changes (removing legacy code with unclear purpose, consolidating similar-but-not-identical behavior)
- **High**: Up to MODERATE changes (cross-module DRY, removing abstraction layers, splitting files) but skip aggressive changes
- **Low**: Only SAFEST and SAFE changes (formatters, linters, dead code, simple DRY, stutter reduction)
- **Let's discuss**: Talk through the situation to determine the right level

The workflow always starts with the least aggressive changes and works upward.

### 3. Gather QA Instructions
Before starting, the workflow asks if you have custom verification steps beyond the standard test suite. Examples:

- "After each change, start the app and take a screenshot to verify rendering"
- "Run `make demo` and check the output"
- "Verify the CLI `--help` output is still valid"

### 4. Scan and Implement (Loop)
Each iteration:

1. **Fresh `swe-code-reviewer` agent** scans the codebase and returns recommendations organized by risk level (SAFEST → AGGRESSIVE)
2. **Orchestrator selects** the least aggressive changes available (up to the ceiling)
3. **SME implements** the batch (language-specific specialist or orchestrator directly)
4. **QA verifies** - on failure, SME gets up to 3 repair attempts before the batch is reverted
5. **Atomic commit** on success
6. **Loop** - fresh scan catches cascading improvements

**Available specialists:**
- `swe-sme-golang` - Go projects
- `swe-sme-makefile` - Makefiles
- `swe-sme-docker` - Dockerfiles
- `swe-sme-graphql` - GraphQL schemas
- `swe-sme-ansible` - Ansible playbooks
- `swe-sme-zig` - Zig projects
- `swe-sme-html` - HTML/markup
- `swe-sme-css` - CSS/styling
- `swe-sme-javascript` - Vanilla JavaScript
- `swe-sme-typescript` - TypeScript

**For other languages** (Python, Rust, Lua, etc.): The orchestrator implements directly.

### 7. Completion Summary
```
## Refactoring Complete

### Statistics
- Commits made: 7
- Net lines changed: -312
- Batches completed: 7
- Batches aborted: 0

### Changes by Category
- Prune (dead code, unused indirection): 11 instances
- DRY consolidation: 5 instances
- Lint fixes: 2 instances
```

After the summary, the workflow presents **user-decision items** collected across all passes:

- **Commented-out code:** Locations and descriptions of commented-out code found during scanning. The user decides which (if any) to delete. Commented-out code may be debugging helpers, work-in-progress, or intermittently-used code that is temporarily disabled — it's not treated as dead code.
- **Apparently-unused public APIs:** Exported symbols that appear unused internally but may be consumed by external users of the package. Informational only.

### 8. Update Documentation
After the summary, the workflow runs `/review-doc` to bring documentation up to date.

## Examples

### Example 1: Full Codebase Cleanup
```
User: /refactor

Scope: entire codebase

How aggressive should refactoring be?
> High (up to MODERATE changes)

Any special QA instructions?
> Run `make test && make lint` after each change

[Pass 1] Dead code + lint fixes (SAFEST)
  Committed: "refactor: remove dead code and fix lint issues"

[Pass 2] DRY + prune (SAFE)
  Committed: "refactor: consolidate duplicate parsing logic"

[Pass 3] New dead code exposed by consolidation (SAFEST)
  Committed: "refactor: remove dead code exposed by consolidation"

[Passes 4-7...]

[Pass 8] No opportunities found.

## Refactoring Complete
- 7 commits, -312 net lines
- 7/7 batches completed
```

### Example 2: Quick Conservative Pass
```
User: /refactor src/api/

How aggressive?
> Low (SAFEST and SAFE only)

Any special QA instructions?
> (none)

[Pass 1] Remove 3 unused imports, 1 dead function (SAFEST)
  Committed: "refactor: remove dead code in api/"

[Pass 2] Fix naming stutter in route handlers (SAFE)
  Committed: "refactor: reduce stutter in api route handlers"

[Pass 3] No opportunities found.

## Refactoring Complete
- 2 commits, -28 net lines
```

### Example 3: Handling Failures
```
[Pass 4] Consolidate similar validation logic (MODERATE)
  QA verification: FAIL - TestValidateEmail broken
  Repair attempt 1/3... still failing
  Repair attempt 2/3... still failing
  Repair attempt 3/3... still failing
  Reverting batch, logging failure, continuing...

[Pass 5] No other opportunities found.

## Refactoring Complete
- 3 commits, -89 net lines
- 3 completed, 1 aborted

### Aborted Batches
- Consolidate validation logic: Could not resolve TestValidateEmail failure
```

## Tips for Effective Use

1. **Ensure tests exist first.** Refactoring without tests is dangerous. The workflow relies on QA verification to catch regressions.

2. **Start with a clean working tree.** The workflow makes commits. Uncommitted changes will complicate things.

3. **Review the commits afterward.** Each batch is an atomic commit. You can review, amend, squash, or revert as needed.

4. **Start with Low aggression.** You can always run again with a higher ceiling. Low-risk passes build confidence.

5. **Scope aggressively for large codebases.** Target specific modules: `/refactor src/core/` rather than everything.

6. **Run it periodically.** Like tidying a room, regular small sessions beat occasional massive cleanups.

7. **Follow up with `/review-arch` if needed.** If the tactical pass reveals that the module structure itself is the problem, escalate to `/review-arch`.

## Philosophy

**Clarity is the goal:**
- Less code almost always means clearer code
- Red diffs are the strongest signal
- But comprehensibility trumps line count

**Gradient aggression:**
- Always make the least aggressive change available
- More aggressive changes bubble up naturally as gentle ones are exhausted
- The user controls how far up the ladder to climb

**Work within existing architecture:**
- Improve code quality without questioning module boundaries
- No module dissolution, creation, or reorganization
- For architectural changes, use `/review-arch`

**Err on the side of trying:**
- Git makes failed experiments free
- The workflow reverts on failure automatically
- Missed opportunities are invisible

**Respect boundaries:**
- Only touch git-tracked files (untracked files may be irreversible to recover)
- Never delete public/exported APIs (external consumers are invisible to the scanner)
- Defer to the user on commented-out code (it may be intentionally disabled)

**Atomic and reversible:**
- Each batch is one commit
- Easy to review, bisect, or revert
