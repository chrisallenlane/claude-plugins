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

### /refactor - Iterative Code Quality Improvement

Iteratively scans for tactical code quality improvements (DRY, dead code, naming, complexity), implements through specialist agents with QA verification, and loops until no improvements remain. Works within existing architecture.

**Use when:**
- Cleaning up accumulated technical debt
- After a major feature is complete and you want to tidy up
- Routine DRY, dead code, naming, and complexity fixes
- You want a quick, low-risk cleanup pass

**Key principle:** Clarity through red diffs. Always make the least aggressive change available first, and work upward.

[Detailed documentation](skills/refactor/README.md)

### /arch-review - Blueprint-Driven Architectural Improvement

Analyzes codebase architecture via noun analysis, produces a target blueprint, then collaborates with the user to review, refine, and decide what to implement. Changes are made through specialist agents with QA verification.

**Use when:**
- Module boundaries are unclear or responsibilities overlap
- The codebase has grown organically and needs structural rethinking
- Utility grab-bags ("helpers", "utils") need dissolution
- Preparing a codebase for a major new feature that needs clean abstractions

**Key principle:** Recommend boldly, implement collaboratively. The analysis agent surfaces every opportunity; the user decides what to act on.

[Detailed documentation](skills/arch-review/README.md)

### /test-audit - Test Suite Quality Review

Reviews test quality interactively: scans for brittle, tautological, and useless tests, presents findings for user selection, then fixes or removes the selected issues.

**Use when:**
- After a burst of agent-written tests
- When test suite maintenance cost feels disproportionate to value
- Before trusting a test suite you didn't write
- Periodically, to keep test quality from drifting

**Key principle:** Tests exist to catch bugs, not to exist. A bad test is worse than no test.

[Detailed documentation](skills/test-audit/README.md)

### /test-cover - Test Coverage Gap Filler

Analyzes coverage reports (or generates them), identifies untested code paths prioritized by risk, and fills selected gaps through specialist agents. Reports refactoring suggestions for hard-to-test code.

**Use when:**
- Coverage metrics are below target
- After inheriting or onboarding to an under-tested codebase
- Before a release, to strengthen test coverage in critical areas
- After adding new features that may lack thorough testing

**Key principle:** Cover what matters most. Prioritize error handling, security, and business logic over boilerplate.

[Detailed documentation](skills/test-cover/README.md)

### /release-review - Pre-Release Readiness Check

Comprehensive pre-flight check before cutting a release. Scans for debug artifacts, version mismatches, changelog gaps, git hygiene issues, breaking API changes, and license compliance. Runs test suite and build verification. Checks documentation freshness. Interactive — presents findings and lets you decide what to fix.

**Use when:**
- Preparing to tag and release a new version
- Final quality gate before shipping to users
- Validating that a codebase is ready for distribution

**Key principle:** Surface issues, don't silently fix them. Releases deserve human review.

[Detailed documentation](skills/release-review/README.md)

### /test-mutate - Mutation Testing Workflow

Systematically introduces mutations (small deliberate changes) into source code and checks if tests catch them. Surviving mutations reveal genuine coverage gaps that line coverage misses. Multi-session with progress tracking.

**Use when:**
- Verifying test quality for critical code (auth, payment, data processing)
- Finding blind spots that line coverage metrics miss
- Quality gate before shipping high-stakes changes
- After refactoring, to ensure tests still catch bugs

**Key principle:** Mutation score > line coverage. Tests that run code without asserting on behavior give false confidence.

[Detailed documentation](skills/test-mutate/README.md)

### /fix - Automated Bug-Fixing Workflow

Coordinates specialist agents through a bug-fixing cycle: clarify bug, reproduce with failing test, diagnose root cause, implement fix, verify, review, and document. Uses `swe-diagnostician` for root-cause analysis and the appropriate SME for implementation.

**Use when:**
- Fixing a bug that warrants thorough investigation
- You want test-driven reproduction before implementing a fix
- The bug may have related failure modes worth investigating
- You want regression tests alongside the fix

**Key principle:** Diagnose before you fix. Understand why a bug exists, not just what to change.

### /project - Multi-Ticket Orchestration

Orchestrates a batch of tickets as a cohesive project. Creates a project branch, implements each ticket sequentially using `/iterate` in autonomous mode, runs cross-cutting quality passes (`/test-audit`, `/refactor`, `/doc-review`), and presents results for final human review.

**Use when:**
- Implementing a batch of related tickets from your issue tracker
- You want autonomous execution with a single review point at the end
- Multiple tickets share a milestone, tag, or feature area

**Key principle:** Maximize autonomy, minimize accumulated error. Pull the andon cord immediately when something goes wrong.

### /doc-review - Documentation Quality Audit

Spawns a doc-maintainer agent to comprehensively review all project documentation for correctness, completeness, and freshness. Fixes issues autonomously within its authority.

**Use when:**
- After refactoring or architectural changes that may have staled documentation
- As a standalone documentation audit
- Called automatically by `/refactor` and `/arch-review` after completion

**Key principle:** Documentation should reflect the actual state of the codebase, not an aspirational one.

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
| `swe-refactor` | Tactical code quality reviewer (DRY, dead code, naming, complexity) |
| `swe-arch-review` | Architecture reviewer (noun analysis, module boundaries, blueprints) |
| `swe-diagnostician` | Bug root-cause analyst (execution tracing, git archaeology, diagnosis reports) |
| `swe-perf-engineer` | Performance testing and optimization |
| `qa-engineer` | Practical verification and test coverage |
| `qa-test-auditor` | Test quality reviewer (brittle, tautological, useless tests) |
| `qa-coverage-analyst` | Coverage gap analyst (coverage reports, risk prioritization, testability suggestions) |
| `qa-test-mutator` | Mutation testing worker (applies mutations, records results) |
| `qa-release-eng` | Pre-release scanner (debug artifacts, versioning, changelog, git hygiene, breaking changes, licenses) |
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
