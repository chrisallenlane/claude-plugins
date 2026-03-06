# /scope-project - Adversarial Project Planning

## Overview

The `/scope-project` skill plans an entire project through adversarial review. It explores the problem space, drafts tickets organized into batches, then pits a planner against an implementer agent to find gaps, ambiguities, and missing work. Only when the implementer is satisfied that every ticket could be implemented without unanswered questions do the tickets go upstream — already tagged with batch labels ready for `/project` to consume.

**Key benefits:**
- Adversarial review catches planning gaps before they become implementation problems
- Batch structure is a first-class planning artifact, not an afterthought
- Implementation notes give implementers a head start on codebase context
- Draft tickets are staged locally before going upstream — easy to revise
- Human clarification is surfaced during planning, not during implementation

## When to Use

**Use `/scope-project` for:**
- Multi-ticket projects that need careful planning before implementation
- Work that naturally divides into phases or batches
- Projects where you want to hand off a complete, well-specified plan to `/project`
- Complex features where gaps between tickets could cause implementation problems

**Don't use `/scope-project` for:**
- A single feature or bug fix (use `/scope` directly)
- Work where the scope is already well-understood and tickets already exist
- Exploratory work where you're not ready to commit to a project structure
- Quick prototypes or throwaway code

**Rule of thumb:** If you'd create more than 3 tickets and they have dependencies between them, use `/scope-project`. If it's a single ticket, use `/scope`.

## Workflow Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│ /scope-project Workflow                                         │
└─────────────────────────────────────────────────────────────────┘

 ┌──────────────────────────────────────────────┐
 │  1. PROJECT DISCOVERY                        │
 │  ────────────────────────────────────────    │
 │  • Dialogue with user about project goals    │
 │  • Probing questions on scope, constraints   │
 │  • Push back on vagueness                    │
 └──────────────────┬───────────────────────────┘
                    ▼
 ┌──────────────────────────────────────────────┐
 │  2. CODEBASE EXPLORATION                     │
 │  ────────────────────────────────────────    │
 │  • Map current architecture                  │
 │  • Identify affected code areas              │
 │  • Understand existing patterns              │
 │  • Review third-party dependencies           │
 └──────────────────┬───────────────────────────┘
                    ▼
 ┌──────────────────────────────────────────────┐
 │  3. DRAFT PROJECT PLAN                       │
 │  ────────────────────────────────────────    │
 │  • Batch structure with ordering rationale   │
 │  • Ticket inventory per batch                │
 │  • Dependencies and risk areas               │
 │  • Present to user for approval              │
 └──────────────────┬───────────────────────────┘
                    ▼
 ┌──────────────────────────────────────────────┐
 │  4-5. DRAFT TICKETS                          │
 │  ────────────────────────────────────────    │
 │  Create .tickets/ staging directory          │
 │  Spawn one subagent per ticket:              │
 │  • Problem statement                         │
 │  • Proposed solution                         │
 │  • Acceptance criteria                       │
 │  • Technical notes + implementation notes    │
 │  • Dependencies and out-of-scope             │
 └──────────────────┬───────────────────────────┘
                    ▼
        ┌───────────────────────┐
        │  ADVERSARIAL REVIEW   │◄──────────────┐
        └───────────┬───────────┘               │
                    ▼                           │
 ┌──────────────────────────────────────────────┐
 │  6a. IMPLEMENTER REVIEWS ALL TICKETS         │
 │  ────────────────────────────────────────    │
 │  Agent: Language SME or general-purpose      │
 │                                              │
 │  Checks:                                     │
 │  • Can I implement without guessing?         │
 │  • Are acceptance criteria testable?         │
 │  • Are dependencies explicit?                │
 │  • Any missing tickets? Overlaps?            │
 │  • Batch assignments sound?                  │
 │  • Code references accurate?                 │
 │                                              │
 │  Verdict: APPROVED or NEEDS REVISION         │
 ├──────────────────────────────────────────────┤
 │  6b. PLANNER ADDRESSES FEEDBACK              │
 │  ────────────────────────────────────────    │
 │  • Resolve blockers (revise tickets)         │
 │  • Answer questions (or ask human)           │
 │  • Accept/decline suggestions                │
 │  • Draft missing tickets if needed           │
 │  • Adjust batch structure if needed          │
 └──────────────────┬───────────────────────────┤
                    ▼                           │
            Implementer approved?               │
            ├─ No  → Fresh implementer ─────────┘
            │        (or escalate if stalemated)
            └─ Yes ▼
 ┌──────────────────────────────────────────────┐
 │  7. PRESENT FINAL TICKETS TO USER            │
 │  ────────────────────────────────────────    │
 │  Summary view of all tickets and batches     │
 │  Review round count and clarifications       │
 │  Wait for user approval                      │
 └──────────────────┬───────────────────────────┘
                    ▼
 ┌──────────────────────────────────────────────┐
 │  8. CUT TICKETS UPSTREAM                     │
 │  ────────────────────────────────────────    │
 │  • Create batch labels (batch-1, batch-2)    │
 │  • Spawn one subagent per ticket             │
 │  • Apply batch labels + other tags           │
 │  • Present issue URLs                        │
 └──────────────────┬───────────────────────────┘
                    ▼
 ┌──────────────────────────────────────────────┐
 │  9. CLEAN UP                                 │
 │  ────────────────────────────────────────    │
 │  Remove .tickets/ directory                  │
 │  Remove .gitignore entry                     │
 └──────────────────────────────────────────────┘
```

## Workflow Details

### 1. Project Discovery

A dialogue with you about the project's goals, scope, and constraints. The orchestrator asks probing questions and pushes back on vagueness — "add authentication" isn't specific enough; "add JWT-based authentication with refresh tokens and role-based access control" is.

### 2. Codebase Exploration

Deep exploration of the codebase to understand the current architecture, affected code areas, existing patterns, and integration points. This happens once at the project level — individual tickets benefit from this shared context rather than each doing their own exploration.

### 3. Draft Project Plan

The orchestrator synthesizes discovery and exploration into a structured plan: batch structure with ordering rationale, ticket inventory per batch, dependencies, and risk areas.

**This is presented to you for approval** — the primary human checkpoint. You can adjust batches, add/remove tickets, reorder, or ask questions. The plan must be approved before ticket drafting begins.

### 4-5. Draft Tickets

One subagent per ticket drafts detailed ticket content into a `.tickets/` staging directory. Each ticket includes:

- **Problem statement** — what problem this ticket solves
- **Proposed solution** — high-level approach
- **Acceptance criteria** — specific, testable
- **Technical notes** — affected files, key functions, patterns to follow
- **Implementation notes** — current function signatures, module boundaries, integration points (the kind of context that saves the implementer from rediscovery)
- **Dependencies** — what must exist before this ticket can start
- **Out of scope** — what explicitly won't be done

### 6. Adversarial Review Loop

The heart of the workflow. An implementer agent reviews all tickets as if assigned to implement them tomorrow, looking for:

| Check                          | What it catches                          |
|--------------------------------|------------------------------------------|
| Implementable without guessing? | Vague requirements, undefined behaviors  |
| Testable acceptance criteria?  | Subjective criteria ("it should be fast") |
| Explicit dependencies?         | Implicit ordering assumptions             |
| Missing tickets?               | Work that falls through the cracks       |
| Overlapping tickets?           | Conflicting changes to the same code     |
| Sound batch assignments?       | Forward dependencies within a batch      |
| Accurate code references?      | Stale file paths, wrong function names   |

The implementer is a language-specific SME when available (Go SME for Go projects, etc.), giving it real implementation perspective.

**The planner addresses each finding** by revising tickets, asking you for clarification, or pushing back. A fresh implementer then re-reviews. This continues until the implementer approves or the process stalemates (at which point you're brought in).

**Asking you for clarification is normal.** The adversarial review surfaces questions that should be answered during planning, not during implementation. A question surfaced here saves much more time than the same question surfaced mid-implementation.

### 7. Present Final Tickets

After implementer approval, the complete ticket set is presented for your final review. You can inspect any ticket in detail and request adjustments before creation.

### 8. Cut Tickets Upstream

Batch labels are created first, then tickets are created with labels applied. Each ticket includes the batch tag so `/project` can consume them directly.

### 9. Clean Up

The `.tickets/` staging directory is removed.

## The `.tickets/` Staging Directory

Draft tickets are staged locally before going upstream:

```
.tickets/
├── batch-1/
│   ├── 01-add-mcp-subcommand.md
│   ├── 02-protocol-handshake.md
│   └── 03-define-tool-schema.md
└── batch-2/
    ├── 01-expose-read-commands.md
    └── 02-expose-write-commands.md
```

Each file is a markdown document with YAML frontmatter (title, batch, order, dependencies, labels) and structured sections. The staging directory is gitignored and deleted after tickets go upstream.

**Why stage locally?** Revising a local file during adversarial review is cheap. Revising an upstream ticket means editing, re-reading, coordinating — more friction. Local staging lets the planner and implementer iterate freely.

## The Adversarial Review

The adversarial review is the distinguishing feature of `/scope-project`. It works because:

**Different perspectives catch different gaps.** The planner thinks about what needs to be done. The implementer thinks about what they'd need to know to do it. These are fundamentally different lenses.

**Fresh instances prevent anchoring.** Each review round spawns a new implementer with a clean context. The new instance isn't anchored to the previous round's findings — it sees the tickets fresh and may notice different issues.

**Convergence, not perfection.** The goal isn't to document every conceivable edge case — it's to reach a state where an implementer could start work without being blocked by unanswered questions. Most projects converge in 2-3 rounds.

**Stalemate triggers human involvement.** If the planner and implementer keep cycling on the same issues, something fundamental is ambiguous. Bringing in the human at this point is the right move — the adversarial process has done its job by surfacing exactly what needs human judgment.

## Examples

### Example 1: MCP Server Project

```
User: /scope-project

I want to add MCP server support to our CLI tool — expose core
functionality as MCP tools over stdio transport.

[Discovery dialogue, codebase exploration...]

## Project Plan

### Batch 1: Core MCP Infrastructure (4 tickets)
1. MCP server subcommand — entry point, stdio, JSON-RPC
2. Protocol handshake — initialize/initialized
3. Tool schema definitions — CLI→MCP mapping
4. Tool dispatch — route calls to handlers

### Batch 2: Tool Implementations (3 tickets)
5. Read commands as MCP tools
6. Write commands as MCP tools
7. MCP resource support

### Batch 3: Polish (3 tickets)
8. Error handling — CLI→JSON-RPC error mapping
9. Integration tests
10. Documentation

Approve?
> Yes

[Drafting tickets...]
[Adversarial review — round 1: 2 blockers, 1 missing ticket found]
[Planner revises, asks human about MCP resources]
[Adversarial review — round 2: APPROVED with minor suggestions]

## Tickets Created (10 tickets, 3 batches)
- batch-1: #31-#34
- batch-2: #35-#37
- batch-3: #38-#40

Ready for: /project all tickets tagged batch-1, batch-2, batch-3
```

### Example 2: Review Catches Missing Work

```
[Adversarial Review — Round 1]

Implementer findings:
- BLOCKER: Ticket #3 says "map CLI commands to MCP tool definitions"
  but no ticket covers what happens when a CLI command requires
  interactive input (confirmation prompts). MCP tools can't do
  interactive prompts. How should these commands behave?
- MISSING TICKET: No ticket for graceful shutdown handling. If the
  stdio pipe closes mid-operation, what happens?
- QUESTION: Ticket #7 says "expose config as MCP resource" but the
  config file contains API keys. Should resources be filtered?

These are all questions that would have blocked implementation.
The planner revises, asks the human about the config filtering,
and adds a missing ticket for shutdown handling.
```

## Integration with Other Skills

| Skill          | Relationship                                                                                                                                       |
|----------------|----------------------------------------------------------------------------------------------------------------------------------------------------|
| `/scope`       | Plans a single ticket interactively. `/scope-project` plans an entire project with adversarial review.                                             |
| `/project`     | Implements what `/scope-project` plans. Tickets go upstream with batch labels that `/project` consumes directly. Typical flow: `/scope-project` → `/project`. |
| `/batch`       | Can also consume `/scope-project`'s tagged tickets if only one batch needs implementation.                                                         |
| `/iterate`     | Can implement individual tickets from `/scope-project` if full `/project` orchestration isn't needed.                                              |
| `/deliberate`  | Available within `/scope-project` for difficult design decisions during planning.                                                                   |

**The full pipeline:**
```
/scope-project  →  /project
    plan             implement
```

## Tips

1. **Be specific during discovery.** The more precise your project description, the better the initial plan. Vagueness at step 1 becomes blockers at step 6.

2. **Engage with the plan review.** Step 3 is your chance to shape the project structure. Catch batch ordering issues and missing work here — it's cheaper than finding them during adversarial review.

3. **Welcome implementer questions.** When the adversarial review surfaces questions for you, that's the workflow working. A question answered during planning saves far more time than the same question asked mid-implementation.

4. **Trust the convergence process.** Multiple review rounds are normal, not a sign of failure. Each round improves ticket quality.

5. **Check the implementation notes.** These are what make `/scope-project` tickets superior to hand-written ones — they contain codebase context (file paths, function signatures, patterns) that an implementer would otherwise have to rediscover.

6. **The batch structure matters.** Tickets go upstream with batch labels. Think about what each batch delivers as a coherent increment — batch 1 should be useful on its own, not just a foundation for batch 2.

## Requirements

- `git` repository (for issue tracker detection and codebase exploration)
- Issue tracker integration (GitHub `gh`, Gitea MCP, GitLab `glab`)
- If no integration is available, ticket content is output for manual creation
