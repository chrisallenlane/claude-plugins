---
name: arch-review
description: Interactive architectural review workflow. Analyzes codebase organization via noun analysis, produces a target blueprint, then collaborates with the user to decide what to implement. Changes are made through SMEs, verified with QA, and committed atomically.
model: opus
---

# Arch Review - Blueprint-Driven Architectural Improvement

Interactive workflow that analyzes codebase architecture, produces a target blueprint via noun analysis, and collaborates with the user to review, refine, and implement it.

## Philosophy

**Clarity through organization is the goal.** Every module should have a clear identity - a domain noun it owns. Functions should live where a reader expects to find them. DRY and Prune serve this organizational goal, not the other way around.

**Recommend boldly, implement collaboratively.** The analysis agent should surface every opportunity it finds, even uncertain ones — the user can always reject a recommendation. But the decision to implement is the user's. Present findings clearly, iterate on the plan together, and let the user direct what happens next.

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
│  5. Present analysis to user                         │
│  6. Iterate on plan with user                        │
│  7. Ask user how to proceed                          │
│  8. Implement dead code removal                      │
│  9. Implement blueprint items iteratively             │
│     ├─ For each item: SME → QA → commit              │
│     └─ On persistent failure: skip item              │
│ 10. Rescan for cascading improvements                │
│     └─ If new findings → present, iterate, proceed   │
│ 11. Completion summary                               │
│ 12. Update documentation (/doc-review)               │
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

### 5. Present Analysis to User

After the analysis agent returns, present its findings to the user. The user needs to see the full picture before deciding what to do.

**Present three things:**

**a) Noun analysis table.** Show the domain model table from the agent's output. This is the analytical foundation — the user should understand the nouns the agent identified and why they matter.

**b) Proposed changes.** Show the blueprint items — modules to change, absorb, dissolve, or rename. For each item, include the agent's rationale. Group by category (dead code removal, renames, moves, absorptions, dissolutions, new modules).

**c) No-change items.** Show the modules the agent evaluated and explicitly decided to leave alone, with their domain justifications. This is important context — the user may disagree and want to add items, or may spot a module the agent missed entirely.

### 6. Iterate on Plan with User

The user now has the full analysis. Give them the opportunity to shape the plan before anything is implemented.

**The user may want to:**
- Remove items they disagree with
- Add items the agent missed
- Modify proposed changes (e.g., "move that function to module X instead of Y")
- Ask questions about specific recommendations ("why did you flag this as dead code?")
- Adjust the scope or aggression level based on what they see
- Reprioritize items

**Continue iterating until the user is satisfied with the plan.** Don't rush this — architectural decisions are consequential and benefit from deliberation.

### 7. Ask User How to Proceed

Once the plan is finalized, ask the user how they'd like to proceed. Don't assume implementation is the goal — the user may have other intentions.

The user will tell you what they want. Follow their direction.

### 8. Implement Dead Code Removal

If the finalized plan includes dead code removal and the user chose to proceed with implementation, implement removal first. This simplifies everything that follows.

- Batch all dead code removals together
- Spawn appropriate SME (or implement directly for mechanical deletions)
- Verify with QA
- Commit atomically

If no dead code removal is in the plan, skip to step 9.

### 9. Implement Blueprint

Work through the finalized blueprint iteratively. Each blueprint item describes a module's target state - what it owns, what it absorbs, what gets renamed or simplified.

#### 9a. Order Blueprint Items

Sequence items for safety:
1. Linter/formatter fixes
2. Renames and stutter fixes (lowest risk)
3. Function moves within existing modules
4. Module absorptions (A absorbs functions from B)
5. Module dissolutions (all of C's functions distributed elsewhere)
6. New module creation

Within each category, prefer items that don't depend on other items.

#### 9b. For Each Blueprint Item

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

#### 9c. Verify Changes

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

#### 9d. Commit Changes

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

#### 9e. Next Item

Proceed to the next blueprint item. Continue until all items are implemented or skipped.

### 10. Rescan for Cascading Improvements

After the entire blueprint is implemented, rescan the codebase for cascading improvements.

**Why rescan:** Reorganizing code often reveals new opportunities. A module that absorbed functions from three sources may now have internal duplication. Dead code that was reachable through dissolved modules may now be unreachable.

**Spawn a fresh `swe-arch-review` agent** (new instance, fresh context). If it reports "No refactoring needed," proceed to the completion summary.

**If it finds new opportunities:** Follow the same interactive cycle — present the findings to the user (step 5), iterate on the plan (step 6), ask how to proceed (step 7), and implement if directed (steps 8-9). Do not implement rescan findings autonomously.

### 11. Completion Summary

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

### 12. Update Documentation

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
- `/arch-review` analyzes architecture and collaborates with the user on what to change
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

Analysis complete. Here's what I found:

## Noun Analysis

| Namespace   | Method             | Noun     | Verb     | Recommendation          |
|-------------|--------------------|----------|----------|-------------------------|
| `Server`    | `parse_request()`  | request  | parse    | Move to `Request`       |
| `Server`    | `validate_request()`| request | validate | Move to `Request`       |
| `Server`    | `send_response()`  | response | send     | Move to `Response`      |
| `App`       | `load_plugins()`   | plugins  | load     | Move to `Plugins`       |
| `App`       | `init_plugins()`   | plugins  | init     | Move to `Plugins`       |

## Proposed Changes

1. **Dead code removal** (4 instances across 3 files)
   - `helpers.go:42` unused `formatDate()` function
   - `server.go:118` commented-out legacy handler
   - ...

2. **Rename** `parser.go` → `request.go` (domain noun)

3. **request.go absorbs** `validate()` from `server.go`
   Rationale: validation is a request concept, not a server concept.

4. **Dissolve** `helpers.go`
   Rationale: no cohesive domain identity. Distribute functions to owners.

5. **Create** `response.go`
   Rationale: response construction logic scattered across 3 modules.

## No-Change Items

- **config.go** — domain noun: config. Clear identity, well-scoped. No change.
- **middleware.go** — domain noun: middleware. Cohesive, correctly placed. No change.
- **logger.go** — domain noun: logger. Single responsibility. No change.

What would you like to change about this plan?

> I agree with 1-4, but let's hold off on creating response.go for now.
  Also, can you explain why you want to dissolve helpers.go? I think
  some of those functions are genuinely shared utilities.

Good question. helpers.go contains 6 functions:
- `formatDate()` — dead code (already flagged for removal)
- `parseURL()` — only used by request handling → belongs in request.go
- `sanitizeInput()` — only used by request validation → belongs in request.go
- `truncateString()` — used by 3 modules for display formatting
- `pluralize()` — used by 2 modules for display formatting
- `retryWithBackoff()` — used by server.go and client.go

You're right that truncateString, pluralize, and retryWithBackoff are
genuinely shared. I'd suggest keeping a focused `display.go` for the
formatting helpers and moving retryWithBackoff to a `retry.go` module.
That dissolves the grab-bag while preserving shared utilities.

> That works. Let's do that instead. How should we proceed?

Updated plan:
1. Remove dead code (4 instances)
2. Rename parser.go → request.go
3. request.go absorbs validate() from server.go
4. Dissolve helpers.go:
   - parseURL, sanitizeInput → request.go
   - truncateString, pluralize → display.go (new)
   - retryWithBackoff → retry.go (new)

How would you like to proceed?

> Let's implement these changes now.

Implementing dead code removal...
  QA verification: PASS
  Committed: "refactor: remove dead code (4 instances)"

Implementing item 1/3: rename parser.go → request.go
  Spawning swe-sme-golang...
  QA verification: PASS
  Committed: "refactor: rename parser to request (domain noun)"

Implementing item 2/3: request.go absorbs validate() from server.go
  Spawning swe-sme-golang...
  QA verification: FAIL - TestServerValidate broken
  Returning to swe-sme-golang for repair (attempt 1/3)...
  QA verification: PASS
  Committed: "refactor: move validation into request module"

Implementing item 3/3: dissolve helpers.go
  Spawning swe-sme-golang...
  QA verification: PASS
  Committed: "refactor: dissolve helpers; distribute to domain owners"

Blueprint complete. Rescanning...

Spawning fresh swe-arch-review agent...

Rescan found 2 new opportunities:
1. Dead code: `server.go:55` — `validateInput()` now unreachable
   after validation moved to request.go
2. Stutter fix: `request.parse_request()` → `request.parse()`

No further structural changes needed.

Would you like to implement these?
> Yes, go ahead.

Implementing rescan items...
  QA verification: PASS
  Committed: "refactor: remove unreachable code and fix naming stutter"

Rescanning...
No further changes found.

## Arch Review Complete

### Statistics
- Commits made: 5
- Net lines changed: -142
- Blueprint items completed: 3/3
- Rescans performed: 2

Running /doc-review to update documentation...
```
