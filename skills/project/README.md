# /project - Full-Lifecycle Project Workflow

## Overview

The `/project` skill orchestrates an entire project from tickets to release-ready code. It takes batched tickets, implements each batch via the `/batch` workflow in autonomous mode, runs smoke tests, then executes a comprehensive quality pipeline (refactor, arch-review, test-review, doc-review, release-review). The result is a single project branch ready for human review and merge.

**Key benefits:**
- Full project lifecycle in a single invocation
- Multi-batch ticket implementation with dependency-aware ordering
- Smoke testing tailored to your project type
- Comprehensive quality pipeline catches issues at every level
- Maximum autonomy with andon cord escape for genuine blockers
- Three-tier branching (project → batch → topic) keeps work organized
- `PROJECT_PROGRESS.md` provides human-readable progress tracking and crash recovery

## When to Use

**Use `/project` for:**
- Multi-batch projects spanning multiple features or subsystems
- Milestone implementations where tickets are naturally grouped into phases
- Projects where the full quality pipeline adds value (refactoring, arch review, test review, doc review, release review)
- Work where you want to walk away and come back to a finished, polished result

**Don't use `/project` for:**
- A single batch of tickets (use `/batch` directly)
- Single tickets (use `/iterate` or `/bugfix` directly)
- Exploratory work or prototyping
- Projects with heavy user collaboration needed during implementation

**Rule of thumb:** If you have multiple batches of tickets that form a cohesive project, use `/project`. If it's a single batch, use `/batch`.

## Workflow Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│ /project Workflow                                               │
└─────────────────────────────────────────────────────────────────┘

 ┌──────────────────────────────────────────────┐
 │  1. GATHER TICKETS AND BATCHING STRATEGY     │
 │  ────────────────────────────────────────    │
 │  • Which tickets belong to this project?     │
 │  • How are they batched? (tags, explicit)    │
 │  • What's the batch execution order?         │
 └──────────────────┬───────────────────────────┘
                    ▼
 ┌──────────────────────────────────────────────┐
 │  2. DISCUSS SMOKE TESTING PROCEDURES         │
 │  ────────────────────────────────────────    │
 │  • CLI tool: run commands, verify output     │
 │  • MCP server: JSON-RPC commands             │
 │  • Web app: Playwright browser testing       │
 │  • Library: integration tests                │
 │  • API server: hit endpoints                 │
 │  Procedure recorded for steps 6 and 7        │
 └──────────────────┬───────────────────────────┘
                    ▼
 ┌──────────────────────────────────────────────┐
 │  3. PLAN EXECUTION ACROSS BATCHES            │
 │  ────────────────────────────────────────    │
 │  • Per-batch analysis (scope, dependencies)  │
 │  • Cross-batch dependency analysis           │
 │  • Optimal batch ordering                    │
 │  • Present plan to user for approval         │
 └──────────────────┬───────────────────────────┘
                    ▼
 ┌──────────────────────────────────────────────┐
 │  4. CREATE PROJECT BRANCH                    │
 │  ────────────────────────────────────────    │
 │  feat/project-<descriptive-name>             │
 │  from current HEAD                           │
 │  Initialize PROJECT_PROGRESS.md              │
 │                                              │
 │  Andon cord if branch already exists         │
 └──────────────────┬───────────────────────────┘
                    ▼
        ┌───────────────────────┐
        │  PER-BATCH LOOP       │◄──────────────┐
        └───────────┬───────────┘               │
                    ▼                           │
 ┌──────────────────────────────────────────────┐
 │  5a. Create batch branch                     │
 │      feat/batch-<descriptive-name>           │
 │      from project branch                     │
 ├──────────────────────────────────────────────┤
 │  5b. Run /batch (autonomous mode)            │
 │      • Tickets pre-loaded                    │
 │      • Plan approved by orchestrator         │
 │      • Branch creation skipped               │
 │      • Full quality passes within batch      │
 │      • Summary logged (no user wait)         │
 ├──────────────────────────────────────────────┤
 │  5c. Merge batch → project branch            │
 │      git merge --no-ff                       │
 │      Andon cord on merge conflict            │
 ├──────────────────────────────────────────────┤
 │  5d. Post-merge verification                 │
 │      Full test suite + linters               │
 │      Andon cord if tests fail                │
 ├──────────────────────────────────────────────┤
 │  5e. Clean up and checkpoint                 │
 │      Delete batch branch                     │
 │      Update PROJECT_PROGRESS.md              │
 └──────────────────┬───────────────────────────┤
                    ▼                           │
              More batches? ────────────────────┘
                    │
                    ▼ (all batches done)
 ┌──────────────────────────────────────────────┐
 │  6. SMOKE TESTING                            │
 │  ────────────────────────────────────────    │
 │  Execute procedure from step 2               │
 │  • Simple fixes: implement directly          │
 │  • Complex bugs: invoke /bugfix              │
 │  • Design issues: /deliberate, then andon    │
 │  Re-run until clean                          │
 └──────────────────┬───────────────────────────┘
                    ▼
 ┌──────────────────────────────────────────────┐
 │  7. QUALITY PIPELINE                         │
 │  ────────────────────────────────────────    │
 │  7a. /refactor (MAXIMUM aggression)          │
 │  7b. /arch-review (autonomous mode)          │
 │  7c. /refactor again (if 7b made changes)    │
 │  7d. /test-review                            │
 │  7e. /doc-review                             │
 │  7f. /release-review                         │
 │                                              │
 │  Orchestrator may skip passes for            │
 │  trivial projects (logged in report)         │
 └──────────────────┬───────────────────────────┘
                    ▼
 ┌──────────────────────────────────────────────┐
 │  8. FINAL REPORT                             │
 │  ────────────────────────────────────────    │
 │  Present to user:                            │
 │  • Batches implemented with outcomes         │
 │  • Smoke test results                        │
 │  • Quality pipeline results per pass         │
 │  • Deferred items (recommendations)          │
 │  • Statistics (commits, lines, tests)        │
 │  • Branch status (ready to merge)            │
 │                                              │
 │  User decides: merge, more work, or discard  │
 └──────────────────────────────────────────────┘
```

## Workflow Details

### 1. Gather Tickets and Batching Strategy

The orchestrator asks which tickets belong to the project and how they're grouped into batches. Batching will vary by project — tickets might be tagged (`batch-1`, `batch-2`), grouped by milestone, or specified explicitly by the user.

If the batching strategy is unclear, the orchestrator asks. It does not guess.

All tickets are fetched from the issue tracker (GitHub, Gitea, GitLab — auto-detected from `git remote -v`), including title, description, acceptance criteria, labels, and dependencies.

### 2. Discuss Smoke Testing Procedures

Before implementation begins, the orchestrator asks what smoke testing should be performed. This is project-specific:

| Project type | Example smoke test |
|---|---|
| CLI tool | Run the binary with representative commands, verify output |
| MCP server | Build the binary, send JSON-RPC commands, verify responses |
| Web app | Use Playwright MCP for browser testing |
| Library | Run integration tests, verify public API end-to-end |
| API server | Hit key endpoints, verify responses |

The procedure is recorded and reused for smoke testing (step 6) and as QA instructions for quality passes (step 7).

### 3. Plan Execution Across Batches

The orchestrator analyzes all tickets across all batches and produces a cross-batch execution plan:

- **Per-batch:** scope, ticket dependencies, ambiguous tickets
- **Cross-batch:** inter-batch dependencies, optimal ordering, conflict risk areas

The plan is **presented to the user for approval** — this is the primary planned interaction point before autonomous execution begins.

### 4. Create Project Branch

Creates `feat/project-<descriptive-name>` from current HEAD and initializes `PROJECT_PROGRESS.md` with project metadata and batch plan.

If the branch already exists, the orchestrator pulls the andon cord and asks whether to resume or start fresh.

### 5. Per-Batch Execution Loop

For each batch:

**5a. Create batch branch** (`feat/batch-<descriptive-name>`) from the project branch.

**5b. Run `/batch` in autonomous mode.** The full `/batch` workflow runs with overrides:
- Tickets are pre-loaded (no user prompting for ticket specification)
- The batch execution plan is approved by the orchestrator autonomously (using `/deliberate` for unclear ordering decisions)
- Branch creation is skipped — the batch branch is already set up
- Quality passes (refactor + doc-review) run normally within the batch
- The final review summary is logged to `PROJECT_PROGRESS.md` instead of waiting for user input
- Andon cord triggers cascade up to the project orchestrator

**5c. Merge batch branch** into the project branch with `--no-ff` (preserves history). Andon cord on merge conflict.

**5d. Post-merge verification** runs the full test suite and linters. Andon cord if the merge introduced a regression.

**5e. Clean up** by deleting the batch branch and updating `PROJECT_PROGRESS.md`.

### 6. Smoke Testing

After all batches are implemented, the orchestrator executes the smoke testing procedure from step 2. Issues are handled by severity:

- **Straightforward fixes:** implement, verify, commit
- **Complex bugs:** invoke the `/bugfix` workflow
- **Design-level problems:** try `/deliberate` first, andon cord if unresolvable

Smoke tests re-run after fixes until clean.

### 7. Quality Pipeline

Six sequential quality passes, each running its full workflow:

| Pass | Parameters | Notes |
|---|---|---|
| **7a. `/refactor`** | MAXIMUM aggression, smoke test QA instructions, entire codebase | Tactical code cleanup |
| **7b. `/arch-review`** | Entire codebase, autonomous mode (orchestrator reviews blueprint and decides what to implement) | Strategic architectural improvement |
| **7c. `/refactor`** | Same as 7a | Only runs if arch-review made substantive changes (module restructuring, function moves — not just dead code or naming) |
| **7d. `/test-review`** | Full test suite review | Coverage gaps, test quality audit |
| **7e. `/doc-review`** | Full documentation audit | |
| **7f. `/release-review`** | Autonomous mode (orchestrator triages findings) | Pre-release readiness check |

Each pass runs its complete workflow including any embedded sub-passes (e.g., `/refactor` runs its own `/doc-review`). This redundancy is intentional — each agent sees the project with fresh context.

The orchestrator may skip passes for trivial projects. If skipped, the reason is noted in the final report.

**Arch-review autonomous mode:** The orchestrator plays the "user" role — reviewing the analysis, deciding what to implement, and directing execution. Low-risk items (dead code, naming, clear function ownership) are auto-approved. High-impact items (module dissolution, major restructuring) go through `/deliberate`. Items that seem out of scope are deferred to the final report as recommendations.

**Release-review autonomous mode:** The orchestrator triages each finding — auto-fixes mechanical issues, runs `/deliberate` for ambiguous findings, defers user-judgment items to the final report, and pulls the andon cord only for genuinely blocking issues.

### 8. Final Report

A comprehensive summary covering:
- Per-batch ticket outcomes
- Smoke test results and any fixes applied
- Quality pipeline results for each pass
- Deferred items (recommendations the orchestrator chose not to implement)
- Statistics (commits, lines changed, tests added, docs updated)
- Branch status

The user decides: merge the project branch to main, do more work, or discard.

## The Andon Cord

Borrowed from Toyota's production system: when something goes wrong, **stop the line immediately**.

**Before pulling the cord, the orchestrator must:**
1. Attempt autonomous resolution
2. Run `/deliberate` for judgment calls
3. Only escalate if autonomous resolution has failed or is clearly futile

**When pulled:**
1. All work stops
2. User gets: current phase, what went wrong, what autonomous resolution was tried, current branch state, recommended path forward
3. Work resumes only after user guidance

**Triggers:**
- Batch workflow andon cord (cascades up)
- Merge conflict between batch and project branches
- Smoke testing reveals fundamental design issues
- Quality pass reveals blocking issues
- Project branch already exists
- Any situation where continuing would compound errors

## Branching Strategy

```
main
 └── feat/project-v2-release              (project branch)
      ├── feat/batch-core-features           (batch, merged + deleted)
      │    ├── feat/issue-12-add-auth          (topic, merged + deleted)
      │    ├── feat/issue-15-fix-cache         (topic, merged + deleted)
      │    └── feat/issue-18-add-metrics       (topic, merged + deleted)
      └── feat/batch-ui-overhaul             (batch, merged + deleted)
           ├── feat/issue-22-new-dashboard     (topic, merged + deleted)
           └── feat/issue-25-responsive-layout (topic, merged + deleted)
```

- Each ticket gets a topic branch (managed by `/batch`)
- Topic branches merge into the batch branch
- Batch branches merge into the project branch
- The user merges the project branch into main after final review
- `--no-ff` merges at every level preserve history

## State Management

`PROJECT_PROGRESS.md` is maintained at the repository root (gitignored) throughout the workflow. It provides:

- **Human-readable progress:** check it at any time to see where the project stands
- **Crash recovery context:** if the session dies, the next invocation can read the file and understand what was completed

```markdown
# Project: v2-release
Started: 2026-03-05T10:30:00Z
Branch: feat/project-v2-release
Status: Quality Pipeline - Arch Review

## Configuration
- Smoke testing: build binary, send JSON-RPC test commands
- Batches: 2

## Batch Progress

### Batch 1: core-features — COMPLETE
- Tickets: #12, #15, #18
- Commits: 9
- Summary: Auth, caching, metrics implemented

### Batch 2: ui-overhaul — COMPLETE
- Tickets: #22, #25
- Commits: 6
- Summary: Dashboard and responsive layout

## Quality Pipeline
- [x] Refactor (pass 1): 3 commits, -47 lines
- [ ] Arch Review
- [ ] Refactor (pass 2)
- [ ] Test Review
- [ ] Doc Review
- [ ] Release Review

## Issues Log
- 2026-03-05T11:45:00Z: Smoke test found missing error handler on /api/metrics.
  Fixed directly, committed.
```

Updated at every major transition: batch start/complete, quality pass start/complete, andon cord events, smoke test results.

## Available Tools

Beyond the mainline workflow, the orchestrator can invoke:

| Tool | When to use |
|---|---|
| `/deliberate` | Difficult autonomous decisions — spawns adversarial advocates to argue options |
| `/bugfix` | Complex bugs encountered during smoke testing or quality passes |

The orchestrator is encouraged to `/deliberate` before pulling the andon cord for judgment calls. If deliberation doesn't resolve the issue, then escalate.

## Examples

### Example 1: Multi-Batch Project

```
User: /project

Which tickets belong to this project?
> All tickets tagged "v2.0" — they're grouped as batch-1 and batch-2

What smoke testing should be performed?
> Build the binary with `go build ./cmd/server`, then send
  `curl localhost:8080/health` and verify 200 response

## Execution Plan

Batch order:
1. batch-1 (core-features): #12, #15, #18
   - #12 Add auth (foundation for #15)
   - #18 Add metrics (independent)
   - #15 Fix cache (depends on #12 auth)

2. batch-2 (ui-overhaul): #22, #25
   - #22 New dashboard (independent)
   - #25 Responsive layout (depends on #22)

Cross-batch: batch-2 depends on batch-1 (UI consumes new API endpoints)
No ambiguous tickets.

Approve?
> Yes

Creating branch: feat/project-v2-release
Initializing PROJECT_PROGRESS.md

[Batch 1: core-features]
Creating batch branch: feat/batch-core-features
Running /batch (autonomous mode)...
  [#12] Implemented auth — JWT with refresh tokens
  [#18] Implemented metrics — Prometheus endpoint
  [#15] Fixed cache — added RWMutex protection
  /refactor: 1 DRY improvement (-12 lines)
  /doc-review: README updated
Merging feat/batch-core-features → feat/project-v2-release
Post-merge verification: all tests pass

[Batch 2: ui-overhaul]
Creating batch branch: feat/batch-ui-overhaul
Running /batch (autonomous mode)...
  [#22] Implemented dashboard — React components
  [#25] Implemented responsive layout — CSS grid
  /refactor: no changes needed
  /doc-review: 1 update
Merging feat/batch-ui-overhaul → feat/project-v2-release
Post-merge verification: all tests pass

[Smoke Testing]
Building binary... OK
Health check: 200 OK
All smoke tests pass

[Quality Pipeline]
/refactor (MAXIMUM): 5 commits, -89 lines
/arch-review: 2 items implemented (extracted request module, dissolved helpers)
/refactor (pass 2): 2 commits, -31 lines
/test-review: 8 tests added, 2 coverage gaps filled
/doc-review: 3 documentation updates
/release-review: 2 findings resolved (debug printf removed, version bumped)

## Project Complete

### Batches Implemented
- Batch 1 (core-features): 3 tickets
  - #12: Add auth — JWT with refresh tokens
  - #18: Add metrics — Prometheus-compatible /metrics
  - #15: Fix cache — RWMutex protection
- Batch 2 (ui-overhaul): 2 tickets
  - #22: New dashboard — React component tree
  - #25: Responsive layout — CSS grid system

### Smoke Testing
- Result: PASS (no issues)

### Quality Pipeline Results
- Refactor (pass 1): 5 commits, net -89 lines
- Arch Review: 2 items implemented, 1 deferred
- Refactor (pass 2): 2 commits, net -31 lines
- Test Review: 8 tests added, 2 gaps filled
- Doc Review: 3 updates
- Release Review: 2 findings resolved

### Deferred Items
- Arch Review: "Consider extracting response module" — low priority,
  deferred for next cycle

### Statistics
- Total commits: 28
- Net lines changed: +1,247
- Tests added/modified: 31
- Documentation files updated: 5

### Branch Status
- Project branch: feat/project-v2-release
- Base branch: main
- Ready for review and merge
```

### Example 2: Andon Cord During Batch Execution

```
[Batch 2: ui-overhaul]
Creating batch branch: feat/batch-ui-overhaul
Running /batch (autonomous mode)...
  [#22] Implemented dashboard — React components
  [#25] Running /iterate (autonomous)...
  [#25] Acceptance verification: FAIL (attempt 1/3)
  [#25] Acceptance verification: FAIL (attempt 2/3)
  [#25] Acceptance verification: FAIL (attempt 3/3)

⚠ ANDON CORD — Stopping all work

Ticket #25 (Responsive layout) failed acceptance verification
3 times during /iterate step 4 within /batch.

Specific failures:
- CSS grid doesn't render correctly in Safari
- The responsive breakpoints don't match the design spec

Autonomous resolution attempted:
- /deliberate considered: (a) use flexbox fallback, (b) add Safari
  polyfill, (c) adjust breakpoints. Verdict: unclear — the design
  spec may be wrong or the implementation approach may need rethinking.

Current state:
- ✓ Batch 1 (core-features) merged into project branch
- ✗ Batch 2 (ui-overhaul) in progress on feat/batch-ui-overhaul
  - ✓ #22 merged into batch branch
  - ✗ #25 in progress on feat/issue-25-responsive-layout

Recommended path: Clarify the design spec for Safari, then retry.

Awaiting your guidance.
```

### Example 3: Skipped Quality Pass

```
[Quality Pipeline]
/refactor (MAXIMUM): 1 commit, -8 lines
Skipping /arch-review: project scope is trivial (2 small bug fixes,
  no architectural impact). Noted in final report.
Skipping /refactor (pass 2): arch-review was skipped
/test-review: 2 tests added
/doc-review: no changes needed
/release-review: no findings
```

## Integration with Other Skills

| Skill | Relationship |
|---|---|
| `/scope` | Creates tickets that `/project` consumes. Typical flow: `/scope` → organize into batches → `/project`. |
| `/batch` | Runs inside `/project` for each batch. `/project` adds multi-batch coordination, smoke testing, and the quality pipeline. |
| `/iterate` | Runs inside `/batch` for each ticket. The innermost implementation loop. |
| `/refactor` | Runs as project-level quality pass (MAXIMUM aggression) and within each batch (SAFE aggression). |
| `/arch-review` | Runs as project-level quality pass in autonomous mode. |
| `/test-review` | Runs as project-level quality pass. |
| `/doc-review` | Runs as project-level quality pass and within each batch and within `/refactor` and `/arch-review`. |
| `/release-review` | Runs as the final quality pass before reporting. |
| `/deliberate` | Available throughout for difficult autonomous decisions. |
| `/bugfix` | Available for complex bugs found during smoke testing or quality passes. |

**Hierarchy:**
```
/project
├── /batch (per batch)
│   ├── /iterate (per ticket)
│   ├── /refactor (per-batch quality)
│   └── /doc-review (per-batch quality)
├── /refactor (project-level quality)
├── /arch-review (project-level quality)
├── /refactor (conditional second pass)
├── /test-review (project-level quality)
├── /doc-review (project-level quality)
└── /release-review (project-level quality)
```

## Tips

1. **Start with well-specified, well-batched tickets.** The orchestrator runs autonomously — vague tickets or unclear batching leads to andon cord pulls. Use `/scope` to plan tickets first.

2. **Think about batch ordering.** If batch 2 depends on batch 1's changes, make that explicit. The orchestrator analyzes dependencies, but explicit ordering from you is more reliable.

3. **Define concrete smoke tests.** "Make sure it works" is too vague. "Build the binary and run `./tool --help`" is actionable. The more specific your smoke test procedure, the more useful the automated testing.

4. **Check `PROJECT_PROGRESS.md` for status.** If you want to see where a long-running project stands, read this file. It's updated at every major transition.

5. **Trust the quality pipeline.** The redundancy is intentional. Multiple passes with fresh context catch issues that any single pass would miss.

6. **The project branch is your safety net.** Main stays clean. If the project branch is unsatisfactory, discard it.

7. **Review deferred items.** The final report includes items the orchestrator chose not to implement (typically architectural recommendations that seemed out of scope). These are worth reviewing — they may inform the next project.

## Agent Coordination

**Sequential execution:**
- One batch at a time, one quality pass at a time
- Each sub-workflow completes before the next begins
- No parallel execution

**Context management:**
- The project orchestrator is a thin coordinator
- All implementation is delegated to sub-workflows
- Only summary-level state is maintained in the orchestrator's context
- `PROJECT_PROGRESS.md` provides durable state outside the context window

## Abort Conditions

**Abort current batch:**
- Batch workflow's own andon cord triggers (cascades up)
- Merge conflict into project branch
- Post-merge test failures

**Abort quality pass:**
- Unresolvable issues after `/deliberate`
- Skip the pass, log the issue, continue with next pass

**Abort entire workflow:**
- User interrupts
- Multiple consecutive andon cord pulls
- Git repository in unclean state
- Critical system error

**Do NOT abort for:**
- Individual ticket failures within a batch (handled by `/batch`)
- Quality pass recommendations the orchestrator disagrees with (defer them)
- Minor issues that can be noted in the final report
