# /project - Multi-Ticket Orchestration Workflow

## Overview

The `/project` skill takes a batch of tickets, plans their execution order, implements each one sequentially using the `/iterate` workflow in autonomous mode, runs cross-cutting quality passes, and presents results for final review. It turns a set of tickets into a single project branch ready to merge.

**Key benefits:**
- Batch execution of multiple tickets without intervention
- Dependency-aware ordering
- Each ticket gets full `/iterate` quality treatment
- Cross-cutting quality passes catch inter-ticket issues
- Topic branches per ticket, merged into a project branch
- Andon cord protocol stops work immediately on failures

## When to Use

**Use `/project` for:**
- Implementing a sprint's worth of tickets
- Milestone or tag-based batches (e.g., "all v2.0 tickets")
- Multiple related tickets that should ship together
- Batches where you want autonomous execution with quality gates

**Don't use `/project` for:**
- Single tickets (use `/iterate` or `/fix` directly)
- Exploratory work or prototyping
- Tickets that need heavy user collaboration during implementation (use `/iterate` interactively)

**Rule of thumb:** If you have 2+ tickets you want implemented as a cohesive unit, use `/project`.

## Workflow Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│ /project Workflow                                                │
└─────────────────────────────────────────────────────────────────┘

 ┌──────────────────────────────────────────────┐
 │  1. RECEIVE TICKET SPECIFICATION             │
 │  ────────────────────────────────────────    │
 │  • Explicit IDs (#12, #15, #18)             │
 │  • Tag/label query ("all tagged v2.0")       │
 │  • Milestone ("Sprint 4")                    │
 └──────────────────┬───────────────────────────┘
                    ▼
 ┌──────────────────────────────────────────────┐
 │  2. DETECT ISSUE TRACKER & FETCH TICKETS     │
 │  ────────────────────────────────────────    │
 │  • GitHub → gh CLI                           │
 │  • Gitea → MCP tools or API                  │
 │  • GitLab → glab CLI                         │
 │  • Fetch: title, body, criteria, labels      │
 │                                              │
 │  Andon cord if tracker unavailable           │
 └──────────────────┬───────────────────────────┘
                    ▼
 ┌──────────────────────────────────────────────┐
 │  3. PROJECT PLANNING                         │
 │  ────────────────────────────────────────    │
 │  • Dependency analysis (explicit + implicit) │
 │  • Execution ordering                        │
 │  • Flag ambiguous tickets                    │
 │  • Present plan to user for approval         │
 └──────────────────┬───────────────────────────┘
                    ▼
 ┌──────────────────────────────────────────────┐
 │  4. CREATE PROJECT BRANCH                    │
 │  ────────────────────────────────────────    │
 │  feat/project-<descriptive-name>             │
 │  from current HEAD                           │
 │                                              │
 │  Andon cord if branch already exists         │
 └──────────────────┬───────────────────────────┘
                    ▼
        ┌───────────────────────┐
        │  PER-TICKET LOOP      │◄──────────────┐
        └───────────┬───────────┘               │
                    ▼                           │
 ┌──────────────────────────────────────────────┐
 │  5a. Create topic branch                     │
 │      feat/issue-<number>-<slug>              │
 ├──────────────────────────────────────────────┤
 │  5b. Run /iterate (autonomous mode)          │
 │      • Requirements pre-loaded from ticket   │
 │      • Full quality pipeline                 │
 │      • Auto-commit with ticket reference     │
 │      • Post comment on ticket (don't close)  │
 │      • Andon cord on 3x acceptance failure   │
 ├──────────────────────────────────────────────┤
 │  5c. Merge topic → project branch            │
 │      git merge --no-ff                       │
 │      Andon cord on merge conflict            │
 ├──────────────────────────────────────────────┤
 │  5d. Post-merge verification gate            │
 │      Full test suite + linters               │
 │      Andon cord if tests fail                │
 ├──────────────────────────────────────────────┤
 │  5e. Delete topic branch                     │
 │      Mark ticket done in orchestrator state  │
 └──────────────────┬───────────────────────────┤
                    ▼                           │
              More tickets? ────────────────────┘
                    │
                    ▼ (all tickets done)
 ┌──────────────────────────────────────────────┐
 │  6. CROSS-CUTTING QUALITY PASSES             │
 │  ────────────────────────────────────────    │
 │  6a. /test-audit (full suite)                │
 │  6b. /refactor (SAFE aggression)             │
 │  6c. /doc-review (full audit)                │
 └──────────────────┬───────────────────────────┘
                    ▼
 ┌──────────────────────────────────────────────┐
 │  7. FINAL REVIEW                             │
 │  ────────────────────────────────────────    │
 │  Present to user:                            │
 │  • Tickets implemented with outcomes         │
 │  • Statistics (commits, lines, tests)        │
 │  • Quality pass results                      │
 │  • Branch status (ready to merge)            │
 │                                              │
 │  User decides: merge, more work, or discard  │
 └──────────────────────────────────────────────┘
```

## Workflow Details

### 1. Receive Ticket Specification
Accept tickets in any of these forms:
- Explicit list: `#12, #15, #18`
- Tag/label query: "all tickets tagged `v2.0`"
- Milestone: "milestone: Sprint 4"
- Search description

### 2. Detect Issue Tracker & Fetch Tickets
Detects the platform from `git remote -v`:
- **GitHub** (`github.com`) — uses `gh` CLI
- **Gitea** — uses `mcp__gitea__*` MCP tools if available
- **GitLab** — uses `glab` CLI if available

Fetches title, body, acceptance criteria, labels, and dependencies for each ticket.

### 3. Project Planning
Analyzes the batch and produces an execution plan:
- **Dependency analysis**: explicit "depends on" links + implicit (shared files/subsystems)
- **Ordering**: dependencies first, then simpler tickets first among independents
- **Flags**: ambiguous or under-specified tickets

The plan is **presented to the user for approval** — this is the one planned interaction point before autonomous execution begins.

### 4. Create Project Branch
Creates `feat/project-<descriptive-name>` from current HEAD. The project branch is the integration point — all topic branches merge into it, and the user merges it into main at the end.

### 5. Per-Ticket Execution Loop
For each ticket in order:

**5a. Create topic branch** (`feat/issue-<number>-<slug>`) off the project branch.

**5b. Run `/iterate` in autonomous mode.** The full `/iterate` workflow runs with these overrides:
- Requirements are pre-loaded from the ticket body (no user prompting)
- Acceptance criteria are derived from the description if not explicit
- Auto-commits with `Fixes #<number>` in the message
- Posts a comment on the ticket summarizing changes (does not close it)
- Andon cord on 3x acceptance verification failure or unresolvable security findings

**5c. Merge topic branch** into the project branch with `--no-ff` (preserves history). Andon cord on merge conflict.

**5d. Post-merge verification** runs the full test suite. Andon cord if the merge introduced a regression.

**5e. Clean up** by deleting the merged topic branch.

### 6. Cross-Cutting Quality Passes
After all tickets are implemented:
- **`/test-audit`**: Full test suite audit — removes tautological tests, rewrites brittle tests
- **`/refactor`**: Conservative pass (SAFE aggression ceiling only)
- **`/doc-review`**: Full documentation audit

These catch issues that span multiple tickets or emerge from their interaction.

### 7. Final Review
Presents a comprehensive summary. The user decides: merge the project branch to main, do more work, or discard.

## The Andon Cord

Borrowed from Toyota's production system: when something goes wrong, **stop the line immediately**.

**Triggers:**
- Acceptance verification fails 3 times
- Unresolvable critical/high security findings
- Post-merge test suite failure
- Merge conflict
- Issue tracker unavailable
- Empty ticket with no description
- Project branch already exists

**What happens:**
1. All work stops immediately
2. User gets: which ticket failed, what step, what went wrong, current branch state
3. Work resumes only after user guidance

The alternative — pressing forward and hoping later steps compensate — leads to compounding errors that are much harder to fix.

## Branching Strategy

```
main
 └── feat/project-sprint-4          (project branch)
      ├── feat/issue-12-add-auth     (topic, merged + deleted)
      ├── feat/issue-15-fix-cache    (topic, merged + deleted)
      └── feat/issue-18-add-metrics  (topic, merged + deleted)
```

- Each ticket gets its own topic branch
- Topic branches merge into the project branch (not main)
- The user merges the project branch into main after final review
- `--no-ff` merges preserve topic branch history

## Examples

### Example 1: Sprint Batch
```
User: /project #12, #15, #18

Detecting issue tracker... GitHub (gh CLI)
Fetching tickets...

## Project Plan

Execution order:
1. #12: Add user authentication — foundation for other tickets
2. #18: Add metrics endpoint — independent, simple
3. #15: Fix cache race condition — depends on #12 auth changes

Ambiguous tickets: none
Estimated scope: 1 medium, 2 small

Approve this plan?
> Yes

Creating branch: feat/project-sprint-4

[Ticket #12] Creating topic branch: feat/issue-12-add-auth
[Ticket #12] Running /iterate (autonomous)...
[Ticket #12] ✓ Implemented, tests pass, committed
[Ticket #12] Merged into project branch
[Ticket #12] Post-merge verification: all tests pass

[Ticket #18] Creating topic branch: feat/issue-18-add-metrics
[Ticket #18] Running /iterate (autonomous)...
[Ticket #18] ✓ Implemented, tests pass, committed
[Ticket #18] Merged into project branch
[Ticket #18] Post-merge verification: all tests pass

[Ticket #15] Creating topic branch: feat/issue-15-fix-cache
[Ticket #15] Running /iterate (autonomous)...
[Ticket #15] ✓ Implemented, tests pass, committed
[Ticket #15] Merged into project branch
[Ticket #15] Post-merge verification: all tests pass

Running cross-cutting quality passes...
- /test-audit: 2 tautological tests removed
- /refactor: 1 DRY improvement (-12 lines)
- /doc-review: README updated with auth docs

## Project Complete

### Tickets Implemented
- #12: Add user authentication — JWT with refresh tokens
- #18: Add metrics endpoint — Prometheus-compatible /metrics
- #15: Fix cache race condition — Added RWMutex protection

### Statistics
- Total commits: 9
- Net lines changed: +847
- Tests added/modified: 23

### Branch Status
- Project branch: feat/project-sprint-4
- Ready to merge into main
```

### Example 2: Andon Cord
```
[Ticket #15] Running /iterate (autonomous)...
[Ticket #15] Acceptance verification: FAIL (attempt 1/3)
[Ticket #15] Acceptance verification: FAIL (attempt 2/3)
[Ticket #15] Acceptance verification: FAIL (attempt 3/3)

⚠ ANDON CORD — Stopping all work

Ticket #15 (Fix cache race condition) failed acceptance verification
3 times during /iterate step 4.

Specific failures:
- TestConcurrentCacheAccess still shows data race under -race flag
- The fix protects reads but not the eviction goroutine

Current state:
- ✓ #12 merged into project branch
- ✓ #18 merged into project branch
- ✗ #15 in-progress on feat/issue-15-fix-cache

Awaiting your guidance.
```

## Integration with Other Skills

| Skill | Relationship |
|-------|-------------|
| `/scope` | Creates tickets that `/project` consumes. Typical flow: `/scope` then `/project`. |
| `/iterate` | Runs inside `/project` for each ticket. `/project` adds batching, ordering, and branching. |
| `/fix` | Not used by `/project` currently — all tickets go through `/iterate`. |
| `/test-audit` | Runs as cross-cutting quality pass after all tickets. |
| `/refactor` | Runs as cross-cutting quality pass (SAFE aggression). |
| `/doc-review` | Runs as cross-cutting quality pass. |

## Tips

1. **Start with well-specified tickets.** `/project` works autonomously — vague tickets lead to andon cord pulls. Use `/scope` to create well-defined tickets first.

2. **Review the execution plan.** Step 3 is your one planned interaction point. Catch ordering issues and ambiguous tickets here.

3. **Trust the andon cord.** When it fires, something genuinely went wrong. Don't try to force past it — address the root cause.

4. **Check cross-cutting results.** The quality passes in step 6 often find inter-ticket issues (duplicate code, conflicting patterns) that per-ticket `/iterate` runs can't see.

5. **The project branch is your safety net.** Main stays clean. If the project branch is unsatisfactory, you can discard it entirely.
