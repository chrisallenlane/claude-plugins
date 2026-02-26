---
name: arch-review
description: Autonomous architectural review workflow. Analyzes codebase organization via noun analysis, produces a target blueprint, implements changes through SMEs, verifies with QA, commits atomically. Optionally rescans for cascading improvements.
model: opus
---

# Arch Review - Blueprint-Driven Architectural Improvement

Autonomous workflow that analyzes codebase architecture, produces a target blueprint via noun analysis, and iteratively implements it.

## Philosophy

**Clarity through organization is the goal.** Every module should have a clear identity - a domain noun it owns. Functions should live where a reader expects to find them. DRY and Prune serve this organizational goal, not the other way around.

**Err on the side of trying.** When uncertain whether a refactoring is worthwhile, attempt it anyway. Git makes failed experiments free - the workflow will revert changes that don't pass QA. Missed opportunities are invisible; failed attempts teach you something. Be bold, knowing that version control provides the safety net.

**Red diffs are good within modules.** Once a function is in the right place, simplify its implementation. Less code is better when it doesn't sacrifice comprehensibility. But don't let line-count savings override architectural decisions.

## Workflow Overview

```
┌─────────────────────────────────────────────────────┐
│           ARCH REVIEW WORKFLOW                       │
├─────────────────────────────────────────────────────┤
│  1. Determine scope                                  │
│  2. Select aggression level                          │
│  3. Gather QA instructions                           │
│  4. Spawn swe-arch-review agent (full analysis)      │
│     → returns dead code list + target blueprint      │
│  5. Implement dead code removal                      │
│  6. Implement blueprint items iteratively            │
│     ├─ For each item: SME → QA → commit              │
│     └─ On persistent failure: skip item              │
│  7. Rescan for cascading improvements (optional)     │
│     └─ If new blueprint → loop to step 5             │
│  8. Completion summary                               │
│  9. Update documentation (/doc-review)               │
└─────────────────────────────────────────────────────┘
```

## Workflow Details

### 1. Determine Scope

**Default:** Entire codebase.

**If user specifies scope:** Respect that scope (directory, files, module). Pass scope constraint to all spawned agents.

### 2. Select Aggression Level

**Ask the user:** "How aggressive should the architectural review be?"

Present these options:
- **Maximum**: Full architectural restructuring — dissolve modules, create new namespaces, reorganize the module hierarchy
- **High**: Moderate structural changes — move functions between modules, rename modules, absorb small modules into larger ones, but don't reorganize the top-level structure
- **Low**: Safe changes only — rename for clarity, fix stutter, move misplaced functions, remove dead code, but don't create or dissolve modules
- **Let's discuss**: Talk through the situation to determine the right level

Pass this level to the `swe-arch-review` agent so it can calibrate the scope of its blueprint.

### 3. Gather QA Instructions

**Ask the user:** "Are there any special verification steps for the QA agent? For example: visual checks, manual testing commands, specific scenarios to validate."

**If provided:** Pass these instructions to the QA agent on every verification cycle, in addition to standard test suite execution.

**Examples of custom QA instructions:**
- "After each change, start the app, take a screenshot, and verify it renders correctly"
- "Run `make demo` and check that output matches expected behavior"
- "Hit the `/health` endpoint and verify 200 response"
- "Verify the CLI still produces valid output for `./tool --help`"

**If none provided:** QA agent runs standard verification (test suite, linters, formatters).

### 4. Analyze Codebase

**Spawn fresh `swe-arch-review` agent:**

The agent performs four sequential analysis steps:
1. Catalogs dead code for removal
2. Builds a domain model via noun analysis
3. Catalogs repetition patterns
4. Produces a target architecture blueprint

**Prompt the agent with:**
```
Perform a full analysis of this codebase.
Scope: [entire codebase | user-specified scope]
Aggression level: [Maximum | High | Low]
Produce a comprehensive target architecture blueprint showing where
everything should live. Cover every module that should change.
```

The agent returns:
- A dead code list (to implement first)
- A noun analysis table (domain model)
- A repetition catalog (DRY candidates, resolved in the blueprint)
- A target architecture blueprint (the primary output)
- Any linter/formatter issues
- Any behavior-altering changes requiring approval

**If the agent reports "No refactoring needed":** Workflow complete.

**Why fresh instances:** Refactoring creates new opportunities. A fresh agent sees the codebase as it is *now*, not as it was before previous changes. No accumulated context or assumptions.

### 5. Implement Dead Code Removal

If the analysis identified dead code, implement removal first. This is uncontroversial and simplifies everything that follows.

- Batch all dead code removals together
- Spawn appropriate SME (or implement directly for mechanical deletions)
- Verify with QA
- Commit atomically

If no dead code was found, skip to step 6.

### 6. Implement Blueprint

Work through the target architecture blueprint iteratively. Each blueprint item describes a module's target state - what it owns, what it absorbs, what gets renamed or simplified.

#### 6a. Order Blueprint Items

Sequence items for safety:
1. Linter/formatter fixes
2. Renames and stutter fixes (lowest risk)
3. Function moves within existing modules
4. Module absorptions (A absorbs functions from B)
5. Module dissolutions (all of C's functions distributed elsewhere)
6. New module creation

Within each category, prefer items that don't depend on other items.

#### 6b. For Each Blueprint Item

**Check for behavior-altering changes.** If the blueprint item was flagged as behavior-altering by the analysis agent, present it to the user for approval before proceeding. Skip if not approved.

**Detect appropriate SME and spawn based on primary file type:**
- Go: `swe-sme-golang`
- Dockerfile: `swe-sme-docker`
- Makefile: `swe-sme-makefile`
- GraphQL: `swe-sme-graphql`
- Ansible: `swe-sme-ansible`
- Zig: `swe-sme-zig`

**For languages without a dedicated SME** (Python, JavaScript, Rust, Lua, etc.): implement directly as orchestrator, following language idioms and project conventions.

**For mixed-language items**: split into per-language batches, or implement directly if changes are mechanical.

**Prompt the SME with:**
```
Implement the following architectural change:

Target state for [module]:
[paste the blueprint item]

This is part of a larger reorganization. Move the listed functions/code
into this module, update all references, and simplify implementations
where possible.

Follow existing project conventions. Maintain all existing behavior.
Report when complete.
```

**SME implements and reports back.**

#### 6c. Verify Changes

**Spawn `qa-engineer` agent:**
- Run test suite
- Run linters/formatters
- Execute any custom QA instructions gathered in step 3
- Verify no regressions introduced
- Report pass/fail with specifics

**On PASS:** Proceed to commit.

**On FAIL:**
1. Return failure details to SME for repair
2. SME attempts fix
3. Re-verify with QA
4. Track failure count for this item

**After 3 consecutive failures for an item:**
- Revert all changes for that item (`git checkout -- .`)
- Log the failure (what was attempted, why it failed)
- Continue with next blueprint item
- Include in final summary as "skipped item"

#### 6d. Commit Changes

**Create atomic commit for successful item:**

```bash
git add [specific files modified]
git diff --staged  # verify only intended changes
git commit -m "$(cat <<'EOF'
refactor: [brief description of changes]

[Details of what was refactored and why]

Co-Authored-By: Claude <noreply@anthropic.com>
EOF
)"
```

**Commit guidelines:**
- Stage only files modified by the current item (not `git add -A`)
- Verify staged changes before committing
- Use `refactor:` prefix in commit message
- Keep items atomic (one logical change per commit)

#### 6e. Next Item

Proceed to the next blueprint item. Continue until all items are implemented or skipped.

### 7. Rescan for Cascading Improvements

After the entire blueprint is implemented, rescan the codebase for cascading improvements.

**Why rescan:** Reorganizing code often reveals new opportunities. A module that absorbed functions from three sources may now have internal duplication. Dead code that was reachable through dissolved modules may now be unreachable.

**Spawn a fresh `swe-arch-review` agent** (new instance, fresh context). If it produces a new blueprint with meaningful changes, return to step 5. If it reports "No refactoring needed," the workflow is complete.

### 8. Completion Summary

When workflow completes, present summary:

```
## Arch Review Complete

### Statistics
- Commits made: N
- Net lines changed: -XXX (target: negative in source)
- Blueprint items completed: N
- Blueprint items skipped: N
- Rescans performed: N

### Blueprint Status
- [module]: completed / skipped (reason)

### Skipped Items (if any)
- [Item description]: [reason for failure]
```

### 9. Update Documentation

After the summary, run the `/doc-review` workflow to bring project documentation up to date. Architectural changes often rename modules, move functions, and change the project structure — documentation that references the old structure becomes stale.

Invoke the skill directly:
```
/doc-review
```

This spawns a doc-maintainer agent that audits all project documentation and fixes issues it finds. Any changes are committed separately from the refactoring commits.

## Agent Coordination

**Fresh instances for context management:**
- Spawn NEW `swe-arch-review` agent for each analysis pass
- This prevents context accumulation in the analyzer
- Orchestrator (you) maintains only summary state

**Sequential execution:**
- One agent at a time
- Wait for completion before spawning next
- No parallel agent execution

**State to maintain (as orchestrator):**
- Current blueprint and progress through it
- Completed items (brief log)
- Skipped items (with reasons)
- Failure count per active item
- Running totals for summary

## Abort Conditions

**Abort current item:**
- 3 consecutive QA failures
- Revert changes, log failure, continue with next item

**Abort entire workflow:**
- User interrupts
- Git repository in unclean state that can't be resolved
- Critical system error

**Agent failures:**
- Spawn failure: retry once, then abort workflow with error
- Malformed output: log issue, skip item, continue
- Timeout: treat as failure, apply retry logic

**Do NOT abort for:**
- Individual item failures (skip and continue)
- Warnings from linters (fix or note, don't abort)

## Integration with Other Skills

**Relationship to `/refactor`:**
- `/refactor` is a tactical workflow for code quality improvements within existing architecture (DRY, dead code, naming, complexity)
- `/arch-review` is a strategic workflow that questions and restructures the architecture itself (noun analysis, module boundaries, blueprints)
- Use `/refactor` for routine cleanup; use `/arch-review` when the module structure itself needs rethinking

**Relationship to `/iterate`:**
- `/iterate` is a feature development workflow that optionally invokes `swe-refactor` (tactical) for code review after implementation
- `/arch-review` is a dedicated architectural improvement workflow

**Relationship to `/scope`:**
- `/scope` explores and creates tickets
- `/arch-review` implements architectural improvements autonomously
- Could use `/scope` first to plan a large restructuring, then `/arch-review` to execute

## Example Session

```
> /arch-review

Scope: entire codebase

How aggressive should the architectural review be?
> Maximum

Any special QA instructions?
> Run `make test && make lint` after each change

Starting analysis...

Spawning swe-arch-review agent...

Analysis complete:
  Dead code: 4 instances across 3 files
  Blueprint: 5 modules affected, 2 dissolutions
  Behavior-altering: none

Implementing dead code removal...
  Spawning swe-sme-golang...
  QA verification: PASS
  Committed: "refactor: remove dead code (4 instances)"

Implementing blueprint item 1/5: rename parser.go → request.go
  Spawning swe-sme-golang...
  QA verification: PASS
  Committed: "refactor: rename parser to request (domain noun)"

Implementing blueprint item 2/5: request.go absorbs validate() from server.go
  Spawning swe-sme-golang...
  QA verification: FAIL - TestServerValidate broken
  Returning to swe-sme-golang for repair (attempt 1/3)...
  QA verification: PASS
  Committed: "refactor: move validation into request module"

Implementing blueprint item 3/5: dissolve helpers.go
  Spawning swe-sme-golang...
  QA verification: PASS
  Committed: "refactor: dissolve helpers; distribute to domain owners"

[Items 4-5...]

Blueprint complete. Rescanning...

Spawning fresh swe-arch-review agent...
Found: 2 new dead code blocks exposed by reorganization.
No further architectural changes needed.

Implementing dead code removal...
  Committed: "refactor: remove dead code exposed by reorganization"

Rescanning...
No refactoring needed.

## Arch Review Complete

### Statistics
- Commits made: 7
- Net lines changed: -198
- Blueprint items completed: 5/5
- Rescans performed: 2

Running /doc-review to update documentation...

Spawning doc-maintainer agent...
  Updated README.md (renamed parser references → request)
  Updated CLAUDE.md (updated module list)
  Committed: "docs: update documentation after refactoring"
```
