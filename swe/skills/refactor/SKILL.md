---
name: refactor
description: Autonomous iterative refactoring workflow. Scans codebase, implements changes through SMEs, verifies with QA, commits atomically, and loops until no improvements remain. Prefers least aggressive changes first.
model: opus
---

# Refactor - Iterative Codebase Improvement

Autonomous refactoring workflow that iteratively improves code quality, always preferring the least aggressive change available, until no further opportunities exist.

## Philosophy

**Red diffs over green diffs.** The goal is to make the codebase SMALLER and CLEANER. Every iteration should delete more than it adds. DRY is your superpower.

**Err on the side of trying.** When uncertain whether a refactoring is worthwhile, attempt it anyway. Git makes failed experiments free - the workflow will revert changes that don't pass QA. Missed opportunities are invisible; failed attempts teach you something. Be bold in what you try, knowing that version control provides the safety net.

## Workflow Overview

```
┌─────────────────────────────────────────────────────┐
│                  REFACTORING LOOP                   │
├─────────────────────────────────────────────────────┤
│  1. Spawn fresh swe-refactor agent (full scan)      │
│  2. Select least aggressive changes available       │
│  3. If none remain → exit                           │
│  4. Spawn SME agent (implement batch)               │
│  5. Spawn QA agent (verify)                         │
│     ├─ PASS → commit, goto 1                        │
│     └─ FAIL → retry (max 3), then abort batch       │
└─────────────────────────────────────────────────────┘
```

## Workflow Details

### 1. Determine Scope

**Default:** Entire codebase.

**If user specifies scope:** Respect that scope (directory, files, module). Pass scope constraint to all spawned agents.

### 2. Select Aggression Ceiling

**Ask the user:** "How aggressive should refactoring be?"

Present these options:
- **Maximum**: Attempt all improvements, including aggressive restructuring (class splits, module reorganization, removing abstractions)
- **High**: Go up to MODERATE changes (extract methods, SRP, error handling, complex DRY) but skip major restructuring
- **Low**: Only SAFEST and SAFE changes (formatters, linters, dead code, simple DRY, renaming)
- **Let's discuss**: Talk through the situation to determine the right level

The workflow still proceeds from least aggressive to more aggressive - this setting determines how far up the ladder to climb before stopping.

### 3. Gather Focus Areas

**Ask the user:** "Is there anything specific you want the refactoring to focus on?"

**If provided:** Pass these focus areas to the `swe-refactor` agent on every scan. The agent should prioritize finding opportunities that match the user's focus, while still scanning for all patterns.

**Examples of focus areas:**
- "Aggressively use namespaces to decompose large files and reduce stutter"
- "Focus on eliminating duplication in the API handlers"
- "Prioritize dead code removal - we just removed a major feature"
- "Look for opportunities to consolidate configuration into a central config struct"

**If none provided:** Standard balanced scan across all patterns.

### 4. Gather QA Instructions

**Ask the user:** "Are there any special verification steps for the QA agent? For example: visual checks, manual testing commands, specific scenarios to validate."

**If provided:** Pass these instructions to the QA agent on every verification cycle, in addition to standard test suite execution.

**Examples of custom QA instructions:**
- "After each change, start the app, take a screenshot, and verify it renders correctly"
- "Run `make demo` and check that output matches expected behavior"
- "Hit the `/health` endpoint and verify 200 response"
- "Verify the CLI still produces valid output for `./tool --help`"

**If none provided:** QA agent runs standard verification (test suite, linters, formatters).

### 5. Aggression Philosophy

**Always make the least aggressive change available, up to the user's chosen ceiling (step 2).**

The aggression levels (from `swe-refactor` agent) are:

1. **SAFEST:** Formatters, linters, dead code removal
2. **SAFE:** DRY (simple), early exit, inlining, renaming
3. **MODERATE:** Extract methods, SRP, error handling, type safety, complex DRY
4. **AGGRESSIVE:** Large class splitting, namespace reorganization, removing abstractions

These aren't gates to pass through sequentially. Instead:
- Each pass, prefer the least aggressive changes available
- More aggressive changes naturally "bubble up" as gentler options are exhausted
- Stop when reaching the user's ceiling (e.g., if ceiling is High/MODERATE, skip AGGRESSIVE recommendations)
- Earlier refactorings may unlock new gentle changes (rescan catches these)

### 6. Iterative Refactoring Loop

For each iteration:

#### 6a. Scan for Opportunities

**Spawn fresh `swe-refactor` agent:**
- Agent performs FULL scan across all aggression levels
- Pass scope if user specified one
- Agent returns structured recommendations organized by risk level
- Fresh instance each pass (context management)

**Why full scan every time:**
- Refactoring creates new opportunities (consolidating duplicates may reveal higher-order patterns)
- Cascading improvements are the goal
- Fresh scan catches what previous changes unlocked

**Prompt the agent with:**
```
Scan for ALL refactoring opportunities across all aggression levels.
Scope: [entire codebase | user-specified scope]
Return recommendations organized by risk level (SAFEST → AGGRESSIVE).
Focus on changes that will produce RED diffs (net code reduction).
```

**Orchestrator selects least aggressive changes:**
- From the full scan, select the LEAST aggressive recommendations available
- Batch and implement those
- Rescan - previous changes may have unlocked new gentle options
- Aggressive changes naturally surface as gentler ones are exhausted
- If no recommendations at any level: workflow complete

#### 6b. Plan Implementation

Review recommendations from scan. Group related changes into atomic batches.

**Batching criteria:**
- Changes to the same module/package
- Logically related refactorings (e.g., all DRY violations of the same pattern)
- Changes that must be done together to maintain consistency

**For each batch, prepare:**
- Clear list of changes to make
- Files affected
- Expected outcome (lines removed, patterns eliminated)
- Which SME agent is appropriate (based on language/framework)

#### 6c. Implement Changes

**Detect appropriate SME and spawn based on primary file type in batch:**
- Go: `swe-sme-golang`
- Dockerfile: `swe-sme-docker`
- Makefile: `swe-sme-makefile`
- GraphQL: `swe-sme-graphql`
- Ansible: `swe-sme-ansible`
- Zig: `swe-sme-zig`

**For languages without a dedicated SME** (Python, JavaScript, Rust, etc.): implement directly as orchestrator, following language idioms and project conventions.

**For mixed-language batches**: split into per-language batches, or implement directly if changes are mechanical (e.g., dead code removal across file types).

**Prompt the SME with:**
```
Implement the following refactorings:
[List of specific changes from batch]

These changes should:
- Follow existing project conventions
- Maintain all existing behavior
- Result in net code reduction where possible

Report when complete.
```

**SME implements and reports back.**

#### 6d. Verify Changes

**Spawn `qa-engineer` agent:**
- Run test suite
- Run linters/formatters
- Execute any custom QA instructions gathered in step 4
- Verify no regressions introduced
- Report pass/fail with specifics

**On PASS:** Proceed to commit.

**On FAIL:**
1. Return failure details to SME for repair
2. SME attempts fix
3. Re-verify with QA
4. Track failure count for this batch

**After 3 consecutive failures for a batch:**
- Revert all changes for that batch (`git checkout -- .`)
- Log the failure (what was attempted, why it failed)
- Continue with next batch
- Include in final summary as "aborted batch"

#### 6e. Commit Changes

**Create atomic commit for successful batch:**

```bash
git add [specific files modified in this batch]
git diff --staged  # verify only intended changes
git commit -m "$(cat <<'EOF'
refactor: [brief description of changes]

[Details of what was refactored and why]

Co-Authored-By: Claude Opus 4.5 <noreply@anthropic.com>
EOF
)"
```

**Commit guidelines:**
- Stage only files modified by the current batch (not `git add -A`)
- Verify staged changes before committing
- Use `refactor:` prefix in commit message
- Keep batches atomic (one logical change per commit)

#### 6f. Loop

Return to step 6a with fresh agent instance.

**Loop continues until:**
- No opportunities found at any aggression level (success)
- User interrupts
- Critical error

### 7. Completion Summary

When workflow completes, present summary:

```
## Refactoring Complete

### Statistics
- Commits made: N
- Net lines changed: -XXX (target: negative)
- Batches completed: N
- Batches aborted: N

### Changes by Category
- Dead code removal: N instances
- DRY consolidation: N instances
- [etc.]

### Aborted Batches (if any)
- [Batch description]: [reason for failure]
```

## Agent Coordination

**Fresh instances for context management:**
- Spawn NEW `swe-refactor` agent for each scan pass
- This prevents context accumulation in the scanner
- Orchestrator (you) maintains only summary state

**Sequential execution:**
- One agent at a time
- Wait for completion before spawning next
- No parallel agent execution

**State to maintain (as orchestrator):**
- Completed batches (brief log)
- Aborted batches (with reasons)
- Failure count per active batch
- Running totals for summary

## Abort Conditions

**Abort current batch:**
- 3 consecutive QA failures
- Revert changes, log failure, continue with next batch

**Abort entire workflow:**
- User interrupts
- Git repository in unclean state that can't be resolved
- Critical system error

**Agent failures:**
- Spawn failure: retry once, then abort workflow with error
- Malformed output: log issue, skip batch, continue
- Timeout: treat as failure, apply retry logic

**Do NOT abort for:**
- Individual batch failures (skip and continue)
- Warnings from linters (fix or note, don't abort)

## Integration with Other Skills

**Relationship to `/iterate`:**
- `/iterate` is a feature development workflow that optionally invokes `swe-refactor` for code review after implementation
- `/refactor` is a dedicated refactoring workflow that uses `swe-refactor` as its core scanner in an autonomous loop
- Same agent, different workflows: one-shot review vs. iterative improvement

**Relationship to `/scope`:**
- `/scope` explores and creates tickets
- `/refactor` implements improvements autonomously
- Could use `/scope` first to plan a large refactoring, then `/refactor` to execute

## Example Session

```
> /refactor

Scope: entire codebase

How aggressive should refactoring be?
> High (up to MODERATE changes)

Any specific focus areas?
> Prioritize DRY - there's a lot of copy-paste in the handlers

Any special QA instructions?
> Run `make test && make lint` after each change

Starting iterative refactoring...

[Pass 1]
Spawning swe-refactor agent for scan...
Found opportunities across levels:
  SAFEST: 8 dead code blocks, 2 lint failures
  SAFE: 3 DRY violations
  MODERATE: 1 extract method opportunity
Selecting least aggressive: dead code + lint (SAFEST)
Spawning swe-sme-golang for implementation...
Implementation complete.
Spawning qa-engineer for verification...
All tests pass.
Committed: "refactor: remove dead code and fix lint issues"

[Pass 2]
Spawning swe-refactor agent for scan...
Found opportunities across levels:
  SAFEST: (none)
  SAFE: 3 DRY violations, 1 new early-exit opportunity
  MODERATE: 1 extract method opportunity
Selecting least aggressive: DRY + early-exit (SAFE)
Spawning swe-sme-golang for implementation...
Implementation complete.
Spawning qa-engineer for verification...
Test failure: TestParseConfig
Returning to swe-sme-golang for repair (attempt 1/3)...
Repair complete.
Spawning qa-engineer for verification...
All tests pass.
Committed: "refactor: consolidate duplicate parsing logic"

[Pass 3]
Spawning swe-refactor agent for scan...
Found opportunities across levels:
  SAFEST: 1 new dead code block (exposed by DRY consolidation)
  SAFE: (none)
  MODERATE: 1 extract method opportunity
Selecting least aggressive: dead code (SAFEST)
...

[Passes 4-7...]

[Pass 8]
Spawning swe-refactor agent for scan...
No opportunities found at any level.

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
```
