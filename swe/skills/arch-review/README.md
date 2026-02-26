# /arch-review - Blueprint-Driven Architectural Improvement

## Overview

The `/arch-review` skill autonomously improves codebase architecture. It spawns an analysis agent that builds a domain model via noun analysis and produces a target architecture blueprint, then iteratively implements that blueprint through specialist agents with QA verification at each step. When the blueprint is fully implemented, it rescans for cascading improvements.

**Key benefits:**
- Blueprint-driven - implements a coherent architectural target, not a grab-bag of independent fixes
- Noun analysis identifies the natural decomposition boundaries in the domain
- Fresh agent instances each pass (prevents context accumulation)
- Atomic commits per item (easy to review, bisect, or revert)
- Built-in quality gates with QA verification
- Cascading improvements - rescans catch what reorganization unlocked

## When to Use

**Use `/arch-review` for:**
- Rethinking module boundaries and responsibilities
- When modules have unclear identities or overlap
- After a codebase has grown organically and needs structural cleanup
- When "helpers.go" or "utils.py" has become a dumping ground
- Preparing a codebase for a major new feature that needs clean abstractions

**Don't use `/arch-review` for:**
- Routine code cleanup (use `/refactor` instead)
- Quick DRY fixes or dead code removal (use `/refactor` instead)
- Codebases without tests (restructuring needs verification)
- Active development where changes are still in flux

**Rule of thumb:** Use `/arch-review` when the module structure itself needs rethinking. Use `/refactor` when the code within modules needs cleaning up.

## Workflow Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│ /arch-review Workflow                                            │
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
 │  Agent: swe-arch-review (fresh instance)     │
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
 │  Fresh swe-arch-review agent                 │
 │                                              │
 │  New blueprint? → Loop to step 5             │
 │  No changes?   ──────────────────────────────┐
 └──────────────────────────────────────────────┘
                                                │
                       ▼                        │
 ┌──────────────────────────────────────────────┐
 │  8. COMPLETION SUMMARY                       │
 │  ────────────────────────────────────────    │
 │  • Total commits made                        │
 │  • Net lines changed (target: negative)      │
 │  • Blueprint items completed vs skipped      │
 │  • Any skipped items with reasons            │
 └──────────────────┬───────────────────────────┘
                    ▼
 ┌──────────────────────────────────────────────┐
 │  9. UPDATE DOCUMENTATION                     │
 │  ────────────────────────────────────────    │
 │  Run /doc-review to fix stale docs           │
 │  (module renames, moved functions, etc.)     │
 └──────────────────────────────────────────────┘
```

## Workflow Details

### 1. Determine Scope
By default, the workflow operates on the entire codebase. You can specify a narrower scope:

```
/arch-review                     # Entire codebase
/arch-review src/parser/         # Just the parser module
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
A fresh `swe-arch-review` agent performs four sequential analysis steps:

| Step | What it does |
|------|-------------|
| 1. Prune dead code | Catalogs unused functions, dead imports, legacy assumptions |
| 2. Noun analysis | Builds domain model - identifies what nouns exist, what's missing, what's misnamed |
| 3. Identify repetition | Catalogs duplication patterns as inputs to the blueprint |
| 4. Produce blueprint | Synthesizes steps 1-3 into a target architecture |

The blueprint describes each module's target state: what it owns, what it absorbs from other modules, what gets renamed, and what implementation simplifications are possible.

**Why fresh instances?** Architectural changes create new opportunities. A fresh agent sees the codebase as it is *now*, not as it was before previous changes.

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
## Arch Review Complete

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

### 9. Update Documentation
After the summary, the workflow runs `/doc-review` to bring project documentation up to date. Architectural changes rename modules, move functions, and change project structure — documentation that references the old structure becomes stale. The doc-review agent audits all documentation files and fixes issues it finds, committing separately from the refactoring commits.

## Tips for Effective Use

1. **Ensure tests exist first.** Restructuring without tests is dangerous. The workflow relies on QA verification to catch regressions.

2. **Start with a clean working tree.** The workflow makes commits. Uncommitted changes will complicate things.

3. **Review the commits afterward.** Each item is an atomic commit. You can review, amend, squash, or revert as needed.

4. **Skipped items are information.** If an item fails 3 times, there may be a deeper issue. Review the skipped item details.

5. **Consider running `/refactor` first.** Cleaning up dead code and DRY violations with `/refactor` simplifies the architectural analysis.

6. **Scope aggressively if needed.** For large codebases, target specific modules: `/arch-review src/core/` rather than everything.

## Agent Coordination

**Fresh instances for context management:**
- A new `swe-arch-review` agent is spawned for each analysis pass
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

The `/arch-review` workflow embodies several key principles:

**Organization first:**
- Every module should own a clear domain noun
- Functions should live where a reader expects to find them
- The blueprint describes a target architecture, not a grab-bag of fixes

**Red diffs within modules:**
- Once code is in the right place, simplify it
- Less code is better when it doesn't sacrifice comprehensibility
- But don't let line count override architectural decisions

**Err on the side of trying:**
- When uncertain, attempt the restructuring anyway
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
