# claude-swe-workflows

A system of composable software engineering workflows for [Claude Code][cc].
Plan projects, implement tickets, and run quality passes — from a single
ticket to a multi-batch project, using the same layered architecture.

## Installation

```bash
claude plugin marketplace add https://github.com/chrisallenlane/claude-swe-workflows.git
claude plugin install claude-swe-workflows@claude-swe-workflows
```

## How It Works

These workflows form a layered system where higher-level workflows
orchestrate lower-level ones. Each layer adds coordination, quality gates,
and autonomy.

```
/project                              ← full project lifecycle
├── /batch (per batch)                ← multi-ticket orchestration
│   ├── /implement (per ticket)         ← single-ticket implementation
│   │   ├── SME implementation        ← language-specific specialist
│   │   ├── QA verification           ← practical + coverage
│   │   ├── Code review               ← security, refactor, perf
│   │   └── Documentation             ← targeted doc updates
│   ├── /refactor                     ← per-batch cleanup
│   └── /doc-review                   ← per-batch doc audit
├── /refactor (MAXIMUM aggression)    ← project-level cleanup
├── /arch-review                      ← architectural restructuring
├── /refactor (conditional)           ← post-restructuring cleanup
├── /test-review                      ← test suite review
├── /doc-review                       ← documentation audit
└── /release-review                   ← pre-release readiness
```

Planning feeds implementation. `/scope-project` plans a multi-batch
project with adversarial review, producing tagged tickets that `/project`
consumes directly:

```
/scope-project  →  /project
    plan             implement + verify + polish
```

For single tickets: `/scope` plans, `/implement` implements.

Two supporting workflows are available at any level: `/deliberate`
(adversarial decision-making for hard choices) and `/bugfix`
(diagnosis-first bug fixing).

## Choosing a Workflow

Not everything needs the full pipeline. Enter at the level that matches
your task:

| You want to...                                          | Use              |
|---------------------------------------------------------|------------------|
| Implement an entire multi-batch project autonomously    | `/project`       |
| Implement a batch of related tickets                    | `/batch`         |
| Implement a single ticket or feature                    | `/implement`       |
| Plan a multi-batch project with adversarial review      | `/scope-project` |
| Plan a single feature and create a ticket               | `/scope`         |
| Fix a bug with diagnosis and root-cause analysis        | `/bugfix`        |
| Make a hard decision with adversarial deliberation      | `/deliberate`    |
| Clean up code quality (DRY, dead code, naming)          | `/refactor`      |
| Rethink module boundaries and architecture              | `/arch-review`   |
| Review and strengthen the test suite                    | `/test-review`   |
| Verify test quality via mutation testing                | `/test-mutate`   |
| Audit all project documentation                         | `/doc-review`    |
| Pre-release readiness check                             | `/release-review`|

**Rules of thumb:**
- Multiple batches of tickets forming a project? `/project`
- One batch of 2+ related tickets? `/batch`
- One ticket? `/implement` (or `/bugfix` if it's a bug)
- Not sure what to build yet? Start with `/scope` or `/scope-project`

## Skills

### Orchestration

These workflows manage the lifecycle of tickets — from implementation
through quality passes to a merge-ready branch.

#### /project — Full-Lifecycle Project Workflow

Orchestrates an entire project from tickets to release-ready code. Takes
batched tickets, implements each batch via `/batch` in autonomous mode,
runs smoke tests, then executes a comprehensive quality pipeline (refactor,
arch-review, test-review, doc-review, release-review). The result is a
single project branch ready for human review and merge.

Maximizes autonomy — the andon cord (stop-the-line escalation) is the only
planned intervention path.

[Detailed documentation](skills/project/README.md)

#### /batch — Multi-Ticket Orchestration

Takes a batch of tickets, plans their execution order, implements each
sequentially using `/implement` in autonomous mode, runs cross-cutting
quality passes (`/refactor`, `/doc-review`), and presents results for
final review.

[Detailed documentation](skills/batch/README.md)

#### /implement — Single-Ticket Development

Orchestrates a complete development cycle through specialist agents:
requirements → planning → implementation → QA → code review →
documentation. Detects project type and dispatches to language-specific
SMEs (Go, GraphQL, Docker, Makefile, Ansible, Zig).

[Detailed documentation](skills/implement/README.md)

### Planning

These workflows explore problem spaces and produce well-specified tickets
without doing implementation work.

#### /scope-project — Adversarial Project Planning

Plans an entire project through adversarial review. Explores the problem
space, drafts tickets organized into batches, then pits a planner against
an implementer agent to find gaps, ambiguities, and missing work. Only
when the implementer is satisfied do tickets go upstream — already tagged
with batch labels ready for `/project` to consume.

[Detailed documentation](skills/scope-project/README.md)

#### /scope — Problem Space Exploration

Explores problem spaces through iterative dialogue and codebase analysis,
then creates a detailed ticket in your issue tracker. For single features,
bug investigations, or refactoring proposals.

[Detailed documentation](skills/scope/README.md)

### Quality

These workflows improve code, tests, architecture, and documentation.
They run as part of `/project`'s quality pipeline, but each works
standalone too.

#### /refactor — Iterative Code Quality Improvement

Autonomously scans for tactical improvements (DRY violations, dead code,
naming issues, unnecessary complexity), implements through specialist
agents with QA verification, and loops until no improvements remain. Works
within existing architecture — for structural changes, use `/arch-review`.

[Detailed documentation](skills/refactor/README.md)

#### /arch-review — Blueprint-Driven Architectural Improvement

Analyzes codebase architecture via noun analysis, produces a target
blueprint, then collaborates with the user to decide what to implement.
For module boundaries, responsibility overlap, utility grab-bag
dissolution, and structural rethinking.

[Detailed documentation](skills/arch-review/README.md)

#### /test-review — Comprehensive Test Suite Review

Three-phase review: fills coverage gaps, identifies missing fuzz tests,
and audits test quality. Each phase has its own analysis → present →
select → implement → verify cycle.

[Detailed documentation](skills/test-review/README.md)

#### /test-mutate — Mutation Testing

Systematically introduces mutations into source code and checks if tests
catch them. Surviving mutations reveal genuine coverage gaps that line
coverage misses. Multi-session with progress tracking.

[Detailed documentation](skills/test-mutate/README.md)

#### /doc-review — Documentation Quality Audit

Comprehensively reviews all project documentation for correctness,
completeness, and freshness. Fixes issues autonomously within its
authority.

[Detailed documentation](skills/doc-review/README.md)

#### /release-review — Pre-Release Readiness Check

Pre-flight check before cutting a release. Scans for debug artifacts,
version mismatches, changelog gaps, git hygiene issues, breaking API
changes, and license compliance. Interactive — presents findings and lets
you decide what to fix.

[Detailed documentation](skills/release-review/README.md)

### Decision and Diagnosis

#### /deliberate — Adversarial Decision Making

Uses adversarial representation to make decisions. Spawns advocate agents
for each option who argue their cases, rebut each other, and respond to
probing questions before a judge renders a verdict with reasoning and
trade-offs.

[Detailed documentation](skills/deliberate/README.md)

#### /bugfix — Diagnosis-First Bug Fixing

Coordinates specialist agents through a diagnosis-first bug-fixing cycle:
reproduce with a failing test, perform root-cause analysis with git
archaeology, implement a targeted fix, and verify. Same review pipeline as
`/implement`.

[Detailed documentation](skills/bugfix/README.md)

## Agents

Specialist agents spawned by the workflows above:

| Agent                 | Purpose                                                                                               |
|-----------------------|-------------------------------------------------------------------------------------------------------|
| `advocate`            | Argues for a specific option in deliberation proceedings                                              |
| `swe-planner`         | Decomposes complex tasks into implementation plans                                                    |
| `swe-sme-golang`      | Go implementation specialist                                                                          |
| `swe-sme-graphql`     | GraphQL schema and resolver specialist                                                                |
| `swe-sme-docker`      | Dockerfile and container specialist                                                                   |
| `swe-sme-makefile`    | Makefile and build system specialist                                                                  |
| `swe-sme-ansible`     | Ansible automation specialist                                                                         |
| `swe-sme-zig`         | Zig implementation specialist                                                                         |
| `swe-refactor`        | Tactical code quality reviewer (DRY, dead code, naming, complexity)                                   |
| `swe-arch-review`     | Architecture reviewer (noun analysis, module boundaries, blueprints)                                  |
| `swe-diagnostician`   | Bug root-cause analyst (execution tracing, git archaeology, diagnosis reports)                        |
| `swe-perf-engineer`   | Performance testing and optimization                                                                  |
| `qa-engineer`         | Practical verification and test coverage                                                              |
| `qa-test-auditor`     | Test quality reviewer (brittle, tautological, useless tests)                                          |
| `qa-coverage-analyst` | Coverage gap analyst (coverage reports, risk prioritization, testability suggestions)                 |
| `qa-fuzz-analyst`     | Fuzz testing gap analyst (fuzz infrastructure detection, candidate identification)                    |
| `qa-test-mutator`     | Mutation testing worker (applies mutations, records results)                                          |
| `qa-release-eng`      | Pre-release scanner (debug artifacts, versioning, changelog, git hygiene, breaking changes, licenses) |
| `sec-reviewer`        | Security vulnerability analysis                                                                       |
| `doc-maintainer`      | Documentation updates and verification                                                                |

## Development

See [HACKING.md](HACKING.md) for local development and testing instructions.

## Requirements

- `git` repository
- For ticket creation: integration with your issue tracker (CLI, MCP server, or API)

[cc]: https://claude.ai/code
