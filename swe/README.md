# SWE Plugin

Software engineering workflow plugin providing skills and specialist agents for systematic development.

## Skills

### /iterate - Automated Development Workflow

Orchestrates a complete development cycle through specialist agents: requirements → planning → implementation → QA → code review → documentation.

**Use when:**
- Building non-trivial features requiring multiple files
- You want quality gates (testing, security review, refactoring suggestions)
- Changes benefit from specialist review (Go, GraphQL, Docker, etc.)
- Practical verification matters (CLI tools, MCP servers, APIs)

**Don't use when:**
- Simple one-line fixes or typos
- Quick prototyping or throwaway code
- Overhead outweighs benefit

[Detailed documentation](skills/iterate/README.md)

### /scope - Problem Space Exploration

Explores problem spaces through iterative dialogue and codebase analysis, then creates detailed tickets in your issue tracker.

**Use when:**
- Planning a complex feature before implementation
- Investigating a bug and documenting findings
- Exploring refactoring opportunities
- Creating well-specified tickets for later implementation

**Key principle:** `/scope` explores and documents. It does NOT implement.

[Detailed documentation](skills/scope/README.md)

### /refactor - Autonomous Codebase Improvement

Analyzes codebase architecture, produces a target blueprint, then iteratively implements it through specialist agents with QA verification at each step. Rescans for cascading improvements until no further changes remain.

**Use when:**
- Cleaning up accumulated technical debt
- After a major feature is complete and you want to tidy up
- Preparing a codebase for handoff or new team members
- You have time allocated specifically for refactoring

**Key principle:** Organization is the goal. Every module should own a clear domain noun, and every function should live where a reader expects to find it.

[Detailed documentation](skills/refactor/README.md)

### /test-audit - Test Suite Quality Review

Reviews test quality interactively: scans for brittle, tautological, and useless tests, presents findings for user selection, then fixes or removes the selected issues.

**Use when:**
- After a burst of agent-written tests
- When test suite maintenance cost feels disproportionate to value
- Before trusting a test suite you didn't write
- Periodically, to keep test quality from drifting

**Key principle:** Tests exist to catch bugs, not to exist. A bad test is worse than no test.

[Detailed documentation](skills/test-audit/README.md)

### /test-mutate - Mutation Testing Workflow

Systematically introduces mutations (small deliberate changes) into source code and checks if tests catch them. Surviving mutations reveal genuine coverage gaps that line coverage misses. Multi-session with progress tracking.

**Use when:**
- Verifying test quality for critical code (auth, payment, data processing)
- Finding blind spots that line coverage metrics miss
- Quality gate before shipping high-stakes changes
- After refactoring, to ensure tests still catch bugs

**Key principle:** Mutation score > line coverage. Tests that run code without asserting on behavior give false confidence.

[Detailed documentation](skills/test-mutate/README.md)

## Agents

Specialist agents spawned by the skills above:

| Agent | Purpose |
|-------|---------|
| `swe-planner` | Decomposes complex tasks into implementation plans |
| `swe-sme-golang` | Go implementation specialist |
| `swe-sme-graphql` | GraphQL schema and resolver specialist |
| `swe-sme-docker` | Dockerfile and container specialist |
| `swe-sme-makefile` | Makefile and build system specialist |
| `swe-sme-ansible` | Ansible automation specialist |
| `swe-sme-zig` | Zig implementation specialist |
| `swe-refactor` | Code quality and refactoring reviewer |
| `swe-perf-engineer` | Performance testing and optimization |
| `qa-engineer` | Practical verification and test coverage |
| `qa-test-auditor` | Test quality reviewer (brittle, tautological, useless tests) |
| `qa-test-mutator` | Mutation testing worker (applies mutations, records results) |
| `sec-reviewer` | Security vulnerability analysis |
| `doc-maintainer` | Documentation updates and verification |

## Workflow Integration

This plugin is part of a three-stage workflow:

```
/deliberate  →  /scope  →  /iterate
   decide        plan      implement
```

Use `/deliberate` (from the deliberate plugin) to resolve architectural
questions, `/scope` to explore and create tickets, and `/iterate` to implement.

Enter at any stage. Complex decisions benefit from the full workflow.
Straightforward changes can go directly to `/iterate`.

## Requirements

- `git` repository
- For ticket creation: integration with your issue tracker (CLI, MCP server, or API)
