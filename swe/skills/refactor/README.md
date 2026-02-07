# /refactor - Iterative Codebase Improvement

## Overview

The `/refactor` skill autonomously improves code quality through an iterative loop: scan for opportunities, implement changes, verify with tests, commit, repeat. It continues until no further improvements are found at any risk level.

**Key benefits:**
- Autonomous operation - set it loose and let it work
- Always prefers least aggressive changes first (more aggressive improvements bubble up)
- Fresh agent instances each pass (prevents context accumulation)
- Atomic commits per batch (easy to review, bisect, or revert)
- Built-in quality gates with QA verification
- Cascading improvements - early refactors unlock new opportunities

## When to Use

**Use `/refactor` for:**
- Cleaning up a codebase that has accumulated technical debt
- After a major feature is complete and you want to tidy up
- Preparing a codebase for a new team member or handoff
- When you have time allocated specifically for refactoring
- Codebases where you want systematic, thorough improvement

**Don't use `/refactor` for:**
- Quick one-off cleanups (just do them directly)
- Codebases without tests (refactoring needs verification)
- When you need specific refactorings (just ask for them)
- Active development where changes are still in flux

**Rule of thumb:** Use `/refactor` when you want comprehensive, autonomous improvement rather than targeted fixes.

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
 └──────────────────┬───────────────────────────┘
                    ▼
 ┌──────────────────────────────────────────────┐
 │  2. SELECT AGGRESSION CEILING                │
 │  ────────────────────────────────────────    │
 │  How far up the ladder to climb:             │
 │  • Maximum: All levels including AGGRESSIVE  │
 │  • High: Up to MODERATE (default)            │
 │  • Low: Only SAFEST and SAFE                 │
 └──────────────────┬───────────────────────────┘
                    ▼
 ┌──────────────────────────────────────────────┐
 │  3. GATHER FOCUS AREAS                       │
 │  ────────────────────────────────────────    │
 │  Optional priorities for the scanner:        │
 │  • "Focus on DRY in the handlers"            │
 │  • "Prioritize dead code removal"            │
 │  (Passed to swe-refactor agent each scan)    │
 └──────────────────┬───────────────────────────┘
                    ▼
 ┌──────────────────────────────────────────────┐
 │  4. GATHER QA INSTRUCTIONS                   │
 │  ────────────────────────────────────────    │
 │  Ask user for custom verification steps:     │
 │  • Visual checks, screenshots                │
 │  • Manual test commands                      │
 │  • Specific scenarios to validate            │
 │  (Optional - standard tests run regardless)  │
 └──────────────────┬───────────────────────────┘
                    ▼
        ┌───────────────────────┐
        │  REFACTORING LOOP     │◄─────────────────────────┐
        └───────────┬───────────┘                          │
                    ▼                                      │
 ┌──────────────────────────────────────────────┐          │
 │  5. SCAN FOR OPPORTUNITIES                   │          │
 │  ────────────────────────────────────────    │          │
 │  Agent: swe-refactor (fresh instance)        │          │
 │                                              │          │
 │  Full scan across all risk levels:           │          │
 │  • SAFEST: formatters, linters, dead code    │          │
 │  • SAFE: DRY, early exit, inlining, naming   │          │
 │  • MODERATE: extract methods, SRP, types     │          │
 │  • AGGRESSIVE: split classes, reorganize     │          │
 │                                              │          │
 │  Returns structured recommendations          │          │
 └──────────────────┬───────────────────────────┘          │
                    ▼                                      │
 ┌──────────────────────────────────────────────┐          │
 │  6. SELECT LEAST AGGRESSIVE                  │          │
 │  ────────────────────────────────────────    │          │
 │  Orchestrator filters recommendations:       │          │
 │                                              │          │
 │  Opportunities exist?                        │          │
 │  ├─ Yes → Select gentlest available          │          │
 │  │        Group into atomic batch            │          │
 │  └─ No  → EXIT: Workflow complete ───────────┼──► DONE  │
 └──────────────────┬───────────────────────────┘          │
                    ▼                                      │
 ┌──────────────────────────────────────────────┐          │
 │  7. IMPLEMENT BATCH                          │          │
 │  ────────────────────────────────────────    │          │
 │  Agent: Language-specific SME or generalist  │          │
 │                                              │          │
 │  Detects primary file type:                  │          │
 │  • Go       → swe-sme-golang                 │          │
 │  • Makefile → swe-sme-makefile               │          │
 │  • Docker   → swe-sme-docker                 │          │
 │  • GraphQL  → swe-sme-graphql                │          │
 │  • Other    → Orchestrator implements        │          │
 │                                              │          │
 │  Implements refactorings from batch          │          │
 └──────────────────┬───────────────────────────┘          │
                    ▼                                      │
 ┌──────────────────────────────────────────────┐          │
 │  8. VERIFY CHANGES                           │          │
 │  ────────────────────────────────────────    │          │
 │  Agent: qa-engineer                          │          │
 │                                              │          │
 │  • Run test suite                            │          │
 │  • Run linters/formatters                    │          │
 │  • Check for regressions                     │          │
 │                                              │          │
 │  Passes? ──┬─ Yes → Continue to commit       │          │
 │            └─ No  → Return to SME ──┐        │          │
 │                     (max 3 attempts)│        │          │
 │                                     ▼        │          │
 │                     ┌────────────────────┐   │          │
 │                     │ Still failing?     │   │          │
 │                     │ → Revert batch     │   │          │
 │                     │ → Log failure      │   │          │
 │                     │ → Continue loop ───┼───┼──────────┤
 │                     └────────────────────┘   │          │
 └──────────────────┬───────────────────────────┘          │
                    ▼                                      │
 ┌──────────────────────────────────────────────┐          │
 │  9. COMMIT BATCH                             │          │
 │  ────────────────────────────────────────    │          │
 │  • Stage specific files (not git add -A)     │          │
 │  • Verify staged changes                     │          │
 │  • Commit with refactor: prefix              │          │
 │  • Include Co-Authored-By                    │          │
 │                                              │          │
 │  Atomic commit for this batch                │          │
 └──────────────────┬───────────────────────────┘          │
                    │                                      │
                    └──────────────────────────────────────┘
                              Loop continues

                                 ▼
 ┌──────────────────────────────────────────────┐
 │  10. COMPLETION SUMMARY                      │
 │  ────────────────────────────────────────    │
 │  When no opportunities remain:               │
 │                                              │
 │  • Total commits made                        │
 │  • Net lines changed (target: negative)      │
 │  • Batches completed vs aborted              │
 │  • Changes by category                       │
 │  • Any failed batches with reasons           │
 └──────────────────────────────────────────────┘
```

## Workflow Details

### 1. Determine Scope
By default, the workflow operates on the entire codebase. You can specify a narrower scope:

```
/refactor                     # Entire codebase
/refactor src/parser/         # Just the parser module
/refactor *.go                # Just Go files
```

The scope is passed to all spawned agents.

### 2. Select Aggression Ceiling
Choose how far up the aggression ladder to climb:

- **Maximum**: Attempt everything, including aggressive restructuring
- **High**: Go up to MODERATE changes, skip major restructuring
- **Low**: Only SAFEST and SAFE changes

Default is High (MODERATE ceiling). The workflow still proceeds from least aggressive upward - this just sets where to stop.

### 3. Gather Focus Areas
Optionally specify what the refactoring should prioritize:

- "Aggressively use namespaces to decompose large files and reduce stutter"
- "Focus on eliminating duplication in the API handlers"
- "Prioritize dead code removal - we just removed a major feature"

These focus areas are passed to the scanner, which prioritizes matching opportunities while still scanning for all patterns.

### 4. Gather QA Instructions
Before starting, the workflow asks if you have custom verification steps beyond the standard test suite. Examples:

- "After each change, start the app and take a screenshot to verify rendering"
- "Run `make demo` and check the output"
- "Verify the CLI `--help` output is still valid"

These instructions are passed to the QA agent on every verification cycle. If you have no special requirements, standard verification (tests + linters) runs.

### 5. Scan for Opportunities
A fresh `swe-refactor` agent scans the codebase across all 25 refactoring patterns, organized by risk level:

| Level | Examples |
|-------|----------|
| SAFEST | Run formatters, remove dead code, fix lint issues |
| SAFE | Eliminate duplication (DRY), early returns, inline single-use vars, rename for clarity |
| MODERATE | Extract methods, improve error handling, centralize config, consistent interfaces |
| AGGRESSIVE | Split god objects, reorganize modules, revisit legacy assumptions, establish backbone |

**Why full scan every time?** Refactoring creates new opportunities. Consolidating duplicates might reveal higher-order patterns. Dead code removal might expose unused dependencies. Fresh scans catch what previous changes unlocked.

### 6. Select Least Aggressive
The orchestrator filters the scan results:

- From all recommendations, select the **least aggressive** ones available (respecting the ceiling from step 2)
- Group related changes into an atomic batch (same module, same pattern type)
- More aggressive changes naturally "bubble up" as gentler options are exhausted
- Stop when reaching the user's ceiling (skip recommendations above that level)

**Example progression:**
```
Pass 1: Dead code removal (SAFEST) - 8 blocks removed
Pass 2: Dead code removal (SAFEST) - nothing left
Pass 3: DRY consolidation (SAFE) - 3 duplications eliminated
Pass 4: Dead code removal (SAFEST) - 1 new block exposed by DRY!
Pass 5: Extract method (MODERATE) - 2 long functions split
...
Pass N: No opportunities at any level → Done
```

### 7. Implement Batch
A language-specific SME (or the orchestrator for unsupported languages) implements the refactorings:

**Available specialists:**
- `swe-sme-golang` - Go projects
- `swe-sme-makefile` - Makefiles
- `swe-sme-docker` - Dockerfiles
- `swe-sme-graphql` - GraphQL schemas
- `swe-sme-ansible` - Ansible playbooks
- `swe-sme-zig` - Zig projects

**For other languages** (Python, JavaScript, Rust, etc.): The orchestrator implements directly, following language idioms.

**For mixed-language batches**: Split into per-language batches, or implement directly if changes are mechanical.

### 8. Verify Changes
The `qa-engineer` agent verifies the refactoring didn't break anything:
- Runs the full test suite
- Runs linters and formatters
- Checks for regressions

**On failure:**
1. Returns to SME with failure details
2. SME attempts repair
3. QA re-verifies
4. Max 3 attempts per batch

**After 3 failures:** Revert the batch, log the failure, continue with next batch. Failed batches appear in the final summary.

### 9. Commit Batch
Each successful batch gets an atomic commit:

```bash
git add src/parser/dedup.go src/parser/dedup_test.go
git diff --staged  # verify only intended changes
git commit -m "refactor: consolidate duplicate parsing logic

Extracted shared validation into parseAndValidate() function.
Removed 47 lines of duplicated code across 3 files.

Co-Authored-By: Claude Opus 4.5 <noreply@anthropic.com>"
```

**Commit guidelines:**
- Stage only files from the current batch
- Use `refactor:` prefix
- Describe what changed and why
- Keep batches atomic (one logical change)

### 10. Completion Summary
When no opportunities remain at any risk level:

```
## Refactoring Complete

### Statistics
- Commits made: 7
- Net lines changed: -312
- Batches completed: 7
- Batches aborted: 0

### Changes by Category
- Dead code removal: 9 instances
- Lint fixes: 2 instances
- DRY consolidation: 5 instances
- Early exit refactoring: 2 instances
- Extract method: 3 instances

### Aborted Batches
(none)
```

## Examples

### Example 1: Full Codebase Cleanup
```
User: /refactor

Scope: entire codebase

How aggressive should refactoring be?
> Maximum (all levels)

Any specific focus areas?
> (none)

Any special QA instructions?
> (none)

Starting iterative refactoring...

[Pass 1]
Scanning... Found: 8 dead code blocks (SAFEST), 3 DRY violations (SAFE)
Selecting: dead code removal (least aggressive)
Implementing via swe-sme-golang...
QA verification: PASS
Committed: "refactor: remove dead code and unused imports"

[Pass 2]
Scanning... Found: 3 DRY violations (SAFE), 1 extract opportunity (MODERATE)
Selecting: DRY consolidation (least aggressive available)
Implementing via swe-sme-golang...
QA verification: FAIL - TestParseConfig broken
Repair attempt 1/3...
QA verification: PASS
Committed: "refactor: consolidate duplicate parsing logic"

[Pass 3]
Scanning... Found: 1 new dead code (exposed by DRY), 1 extract opportunity
Selecting: dead code removal (SAFEST available again!)
...

[Pass 8]
Scanning... No opportunities found at any level.

## Refactoring Complete
- 7 commits, -312 net lines
- 0 failed batches
```

### Example 2: Scoped Refactoring
```
User: /refactor src/api/

Starting iterative refactoring of src/api/...

[Pass 1]
Scanning src/api/... Found: 2 lint issues, 1 naming drift
Selecting: lint fixes + rename (both SAFE)
...

[Pass 4]
Scanning src/api/... No opportunities in scope.

## Refactoring Complete
- 3 commits, -45 net lines
```

### Example 3: Handling Failures
```
[Pass 5]
Scanning... Found: 1 god object to split (AGGRESSIVE)
Selecting: split UserManager class
Implementing via swe-sme-golang...
QA verification: FAIL - 12 tests broken
Repair attempt 1/3... still failing
Repair attempt 2/3... still failing
Repair attempt 3/3... still failing
Reverting batch, logging failure, continuing...

[Pass 6]
Scanning... Found: 1 namespace improvement (AGGRESSIVE)
...

## Refactoring Complete
- 6 commits, -198 net lines
- 1 batch aborted

### Aborted Batches
- Split UserManager class: Could not resolve test failures after 3 attempts.
  Tests affected: TestUserCreate, TestUserUpdate, ...
```

## Tips for Effective Use

1. **Ensure tests exist first.** Refactoring without tests is dangerous. The workflow relies on QA verification to catch regressions.

2. **Start with a clean working tree.** The workflow makes commits. Uncommitted changes will complicate things.

3. **Review the commits afterward.** Each batch is an atomic commit. You can review, amend, squash, or revert as needed.

4. **Trust the aggression ordering.** The "least aggressive first" approach is intentional. Gentle changes often unlock or obviate aggressive ones.

5. **Failed batches are information.** If a batch fails 3 times, there may be a deeper issue. Review the aborted batch details.

6. **Scope aggressively if needed.** For large codebases, target specific modules: `/refactor src/core/` rather than everything.

7. **Run it periodically.** Like tidying a room, regular small sessions beat occasional massive cleanups.

## Agent Coordination

**Fresh instances for context management:**
- A new `swe-refactor` agent is spawned for each scan pass
- This prevents context accumulation in the scanner
- The orchestrator maintains only summary state

**Sequential execution:**
- One agent at a time
- No parallel agent execution
- Each agent completes before the next spawns

**State maintained by orchestrator:**
- Completed batches (brief log)
- Aborted batches (with reasons)
- Failure count per active batch
- Running totals for summary

## Abort Conditions

**Abort current batch:**
- 3 consecutive QA failures → revert, log, continue

**Abort entire workflow:**
- User interrupts
- Git repository in unclean state
- Critical system error

**Agent failures:**
- Spawn failure → retry once, then abort workflow
- Malformed output → log, skip batch, continue
- Timeout → treat as failure, apply retry logic

## Philosophy

The `/refactor` workflow embodies several key principles:

**Least aggressive first:**
- Gentle changes are safer and often unlock bigger wins
- Aggressive refactorings bubble up naturally as simpler options are exhausted
- Running formatters before restructuring modules

**Red diffs over green diffs:**
- The goal is to make the codebase *smaller*
- Best refactorings delete more than they add
- DRY is the most powerful tool

**Err on the side of trying:**
- When uncertain, attempt the refactoring anyway
- Git makes failed experiments free
- Missed opportunities are invisible; failed attempts teach you something

**Fresh eyes each pass:**
- New agent instances prevent accumulated context bias
- Each scan sees the codebase as it is *now*, not as it was
- Cascading improvements are caught by fresh scans

**Atomic and reversible:**
- Each batch is one commit
- Easy to review, bisect, or revert
- Failed batches don't pollute the history

**Autonomous but bounded:**
- Runs until done, but respects failure limits
- Escalates to user when stuck
- Reports everything for human review

---

Ready to clean up? Just type `/refactor` and let it work.
