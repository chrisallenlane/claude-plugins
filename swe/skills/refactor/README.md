# /refactor - Blueprint-Driven Codebase Improvement

## Overview

The `/refactor` skill autonomously improves code quality. It spawns an analysis agent that builds a domain model of the codebase and produces a target architecture blueprint, then iteratively implements that blueprint through specialist agents with QA verification at each step. When the blueprint is fully implemented, it rescans for cascading improvements.

**Key benefits:**
- Autonomous operation - set it loose and let it work
- Blueprint-driven - implements a coherent architectural target, not a grab-bag of independent fixes
- Fresh agent instances each pass (prevents context accumulation)
- Atomic commits per item (easy to review, bisect, or revert)
- Built-in quality gates with QA verification
- Cascading improvements - rescans catch what reorganization unlocked

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
 │  2. SELECT AGGRESSION LEVEL                  │
 │  ────────────────────────────────────────    │
 │  How much disruption is acceptable:          │
 │  • Maximum: Full architectural restructuring │
 │  • High: Moderate structural changes         │
 │  • Low: Safe changes only                    │
 └──────────────────┬───────────────────────────┘
                    ▼
 ┌──────────────────────────────────────────────┐
 │  3. GATHER QA INSTRUCTIONS                   │
 │  ────────────────────────────────────────    │
 │  Ask user for custom verification steps:     │
 │  • Visual checks, screenshots                │
 │  • Manual test commands                      │
 │  • Specific scenarios to validate            │
 │  (Optional - standard tests run regardless)  │
 └──────────────────┬───────────────────────────┘
                    ▼
 ┌──────────────────────────────────────────────┐
 │  4. ANALYZE CODEBASE                         │
 │  ────────────────────────────────────────    │
 │  Agent: swe-refactor (fresh instance)        │
 │                                              │
 │  Four sequential analysis steps:             │
 │  • Step 1: Catalog dead code                 │
 │  • Step 2: Noun analysis (domain model)      │
 │  • Step 3: Identify repetition               │
 │  • Step 4: Produce target blueprint          │
 │                                              │
 │  Returns dead code list + blueprint          │
 │                                              │
 │  No opportunities? → EXIT ──────────────► DONE
 └──────────────────┬───────────────────────────┘
                    ▼
 ┌──────────────────────────────────────────────┐
 │  5. IMPLEMENT DEAD CODE REMOVAL              │
 │  ────────────────────────────────────────    │
 │  Batch all dead code removals together       │
 │  Agent: SME or orchestrator                  │
 │  Verify with QA, commit                      │
 └──────────────────┬───────────────────────────┘
                    ▼
        ┌───────────────────────┐
        │  BLUEPRINT LOOP       │◄───────────────────┐
        └───────────┬───────────┘                    │
                    ▼                                │
 ┌──────────────────────────────────────────────┐    │
 │  6. IMPLEMENT BLUEPRINT ITEM                 │    │
 │  ────────────────────────────────────────    │    │
 │  Ordered by safety:                          │    │
 │  1. Linter/formatter fixes                   │    │
 │  2. Renames and stutter fixes                │    │
 │  3. Function moves within modules            │    │
 │  4. Module absorptions                       │    │
 │  5. Module dissolutions                      │    │
 │  6. New module creation                      │    │
 │                                              │    │
 │  Agent: Language-specific SME or generalist  │    │
 └──────────────────┬───────────────────────────┘    │
                    ▼                                │
 ┌──────────────────────────────────────────────┐    │
 │  VERIFY CHANGES                              │    │
 │  ────────────────────────────────────────    │    │
 │  Agent: qa-engineer                          │    │
 │                                              │    │
 │  Passes? ──┬─ Yes → Commit, next item ───────┤    │
 │            └─ No  → Return to SME ──┐        │    │
 │                     (max 3 attempts)│        │    │
 │                                     ▼        │    │
 │                     ┌────────────────────┐   │    │
 │                     │ Still failing?     │   │    │
 │                     │ → Revert item      │   │    │
 │                     │ → Log failure      │   │    │
 │                     │ → Next item ───────┼───┤    │
 │                     └────────────────────┘   │    │
 └──────────────────────────────────────────────┘    │
                    ▼                                │
           All items done?                           │
           ├─ No  → Back to step 6 ─────────────────┘
           └─ Yes ▼
 ┌──────────────────────────────────────────────┐
 │  7. RESCAN FOR CASCADING IMPROVEMENTS        │
 │  ────────────────────────────────────────    │
 │  Fresh swe-refactor agent                    │
 │                                              │
 │  New blueprint? → Loop to step 5             │
 │  No changes?   → EXIT ─────────────────► DONE
 └──────────────────────────────────────────────┘

                       ▼
 ┌──────────────────────────────────────────────┐
 │  8. COMPLETION SUMMARY                       │
 │  ────────────────────────────────────────    │
 │  • Total commits made                        │
 │  • Net lines changed (target: negative)      │
 │  • Blueprint items completed vs skipped      │
 │  • Any skipped items with reasons            │
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

### 2. Select Aggression Level
The workflow asks how much disruption is acceptable:

- **Maximum**: Full architectural restructuring — dissolve modules, create new namespaces, reorganize the module hierarchy
- **High**: Moderate structural changes — move functions between modules, rename modules, absorb small modules into larger ones, but don't reorganize the top-level structure
- **Low**: Safe changes only — rename for clarity, fix stutter, move misplaced functions, remove dead code, but don't create or dissolve modules
- **Let's discuss**: Talk through the situation to determine the right level

This level is passed to the analysis agent so it can calibrate the scope of its blueprint.

### 3. Gather QA Instructions
Before starting, the workflow asks if you have custom verification steps beyond the standard test suite. Examples:

- "After each change, start the app and take a screenshot to verify rendering"
- "Run `make demo` and check the output"
- "Verify the CLI `--help` output is still valid"

These instructions are passed to the QA agent on every verification cycle. If you have no special requirements, standard verification (tests + linters) runs.

### 4. Analyze Codebase
A fresh `swe-refactor` agent performs four sequential analysis steps:

| Step | What it does |
|------|-------------|
| 1. Prune dead code | Catalogs unused functions, dead imports, legacy assumptions |
| 2. Noun analysis | Builds domain model - identifies what nouns exist, what's missing, what's misnamed |
| 3. Identify repetition | Catalogs duplication patterns as inputs to the blueprint |
| 4. Produce blueprint | Synthesizes steps 1-3 into a target architecture |

The blueprint describes each module's target state: what it owns, what it absorbs from other modules, what gets renamed, and what implementation simplifications are possible.

**Why fresh instances?** Refactoring creates new opportunities. A fresh agent sees the codebase as it is *now*, not as it was before previous changes.

### 5. Implement Dead Code Removal
Dead code removal happens first because it's uncontroversial and simplifies everything that follows. All dead code identified in the analysis is batched together, implemented, verified by QA, and committed.

### 6. Implement Blueprint
The orchestrator works through blueprint items in safety order:

1. **Linter/formatter fixes** - mechanical, lowest risk
2. **Renames and stutter fixes** - low risk, no structural change
3. **Function moves within existing modules** - moderate risk
4. **Module absorptions** (A absorbs functions from B)
5. **Module dissolutions** (all of C's functions distributed elsewhere)
6. **New module creation** - highest structural change

Each item goes through: SME implementation -> QA verification -> atomic commit.

**Available specialists:**
- `swe-sme-golang` - Go projects
- `swe-sme-makefile` - Makefiles
- `swe-sme-docker` - Dockerfiles
- `swe-sme-graphql` - GraphQL schemas
- `swe-sme-ansible` - Ansible playbooks
- `swe-sme-zig` - Zig projects

**For other languages** (Python, JavaScript, Rust, Lua, etc.): The orchestrator implements directly, following language idioms.

After each item, the `qa-engineer` agent verifies the change didn't break anything (test suite, linters, formatters). On failure, the SME gets up to 3 repair attempts. After 3 failures: revert the item, log the failure, continue with the next item.

### 7. Rescan for Cascading Improvements
After the blueprint is fully implemented, a fresh analysis agent rescans the codebase. Reorganization often reveals new opportunities: internal duplication in modules that absorbed functions from multiple sources, dead code that was only reachable through dissolved modules, etc.

If the rescan produces a new blueprint, the workflow loops back. If not, it's done.

### 8. Completion Summary
```
## Refactoring Complete

### Statistics
- Commits made: 7
- Net lines changed: -198
- Blueprint items completed: 5/5
- Rescans performed: 2

### Blueprint Status
- snippet.lua: completed (renamed from parser.lua, absorbed frontmatter.lua)
- keymaps.lua: completed (extracted from init.lua)
- strings.lua: completed (dissolved, functions distributed)
- loader.lua: completed (absorbed strip() from strings.lua)
- init.lua: completed (simplified after extractions)

### Skipped Items
(none)
```

## Examples

### Example 1: Full Codebase Cleanup
```
User: /refactor

Scope: entire codebase

How aggressive should the refactoring be?
> Maximum

Any special QA instructions?
> Run `make test && make lint` after each change

Starting analysis...

Spawning swe-refactor agent...

Analysis complete:
  Dead code: 4 instances across 3 files
  Blueprint: 5 modules affected, 2 dissolutions
  Behavior-altering: none

Implementing dead code removal...
  QA verification: PASS
  Committed: "refactor: remove dead code (4 instances)"

Implementing blueprint item 1/5: rename parser.go → request.go
  QA verification: PASS
  Committed: "refactor: rename parser to request (domain noun)"

Implementing blueprint item 2/5: request.go absorbs validate()
  QA verification: FAIL - TestServerValidate broken
  Repair attempt 1/3...
  QA verification: PASS
  Committed: "refactor: move validation into request module"

[Items 3-5...]

Blueprint complete. Rescanning...
Found: 2 new dead code blocks. No further architectural changes.

Implementing dead code removal...
  Committed: "refactor: remove dead code exposed by reorganization"

Rescanning... No refactoring needed.

## Refactoring Complete
- 7 commits, -198 net lines
- 5/5 blueprint items completed
```

### Example 2: Scoped Refactoring
```
User: /refactor src/api/

How aggressive should the refactoring be?
> High

Any special QA instructions?
> (none)

Analyzing src/api/...

Analysis complete:
  Dead code: 1 instance
  Blueprint: 2 modules affected, 0 dissolutions

Implementing dead code removal...
  Committed: "refactor: remove unused handler in api/"

Implementing blueprint item 1/2: fix naming stutter in routes
  Committed: "refactor: reduce stutter in api route handlers"

Implementing blueprint item 2/2: move validation into request module
  Committed: "refactor: centralize request validation"

Rescanning... No further improvements in scope.

## Refactoring Complete
- 3 commits, -45 net lines
```

### Example 3: Handling Failures
```
[Implementing blueprint item 3/5: dissolve UserManager]
  QA verification: FAIL - 12 tests broken
  Repair attempt 1/3... still failing
  Repair attempt 2/3... still failing
  Repair attempt 3/3... still failing
  Reverting item, logging failure, continuing...

[Implementing blueprint item 4/5: extract AuthService]
  QA verification: PASS
  Committed: "refactor: extract auth logic into AuthService"

...

## Refactoring Complete
- 6 commits, -198 net lines
- 4/5 blueprint items completed

### Skipped Items
- Dissolve UserManager: Could not resolve test failures after 3 attempts.
  Tests affected: TestUserCreate, TestUserUpdate, ...
```

## Tips for Effective Use

1. **Ensure tests exist first.** Refactoring without tests is dangerous. The workflow relies on QA verification to catch regressions.

2. **Start with a clean working tree.** The workflow makes commits. Uncommitted changes will complicate things.

3. **Review the commits afterward.** Each item is an atomic commit. You can review, amend, squash, or revert as needed.

4. **Skipped items are information.** If an item fails 3 times, there may be a deeper issue. Review the skipped item details.

5. **Scope aggressively if needed.** For large codebases, target specific modules: `/refactor src/core/` rather than everything.

6. **Run it periodically.** Like tidying a room, regular small sessions beat occasional massive cleanups.

## Agent Coordination

**Fresh instances for context management:**
- A new `swe-refactor` agent is spawned for each analysis pass
- This prevents context accumulation in the analyzer
- The orchestrator maintains only summary state

**Sequential execution:**
- One agent at a time
- No parallel agent execution
- Each agent completes before the next spawns

**State maintained by orchestrator:**
- Current blueprint and progress through it
- Completed items (brief log)
- Skipped items (with reasons)
- Failure count per active item
- Running totals for summary

## Abort Conditions

**Abort current item:**
- 3 consecutive QA failures -> revert, log, continue

**Abort entire workflow:**
- User interrupts
- Git repository in unclean state
- Critical system error

**Agent failures:**
- Spawn failure -> retry once, then abort workflow
- Malformed output -> log, skip item, continue
- Timeout -> treat as failure, apply retry logic

## Philosophy

The `/refactor` workflow embodies several key principles:

**Organization first:**
- Every module should own a clear domain noun
- Functions should live where a reader expects to find them
- The blueprint describes a target architecture, not a grab-bag of fixes

**Red diffs within modules:**
- Once code is in the right place, simplify it
- Less code is better when it doesn't sacrifice comprehensibility
- But don't let line count override architectural decisions

**Err on the side of trying:**
- When uncertain, attempt the refactoring anyway
- Git makes failed experiments free
- Missed opportunities are invisible; failed attempts teach you something

**Fresh eyes each pass:**
- New agent instances prevent accumulated context bias
- Each analysis sees the codebase as it is *now*, not as it was
- Cascading improvements are caught by fresh rescans

**Atomic and reversible:**
- Each item is one commit
- Easy to review, bisect, or revert
- Skipped items don't pollute the history

**Autonomous but bounded:**
- Runs until done, but respects failure limits
- Escalates to user when stuck
- Reports everything for human review

---

Ready to clean up? Just type `/refactor` and let it work.
