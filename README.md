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
/implement-project                              ← full project lifecycle
├── /implement-batch (per batch)                ← multi-ticket orchestration
│   ├── /implement (per ticket)         ← single-ticket implementation
│   │   ├── SME implementation        ← language-specific specialist
│   │   ├── QA verification           ← practical + coverage
│   │   ├── Code review               ← security, refactor, perf
│   │   └── Documentation             ← targeted doc updates
│   ├── /refactor                     ← per-batch cleanup
│   └── /review-doc                   ← per-batch doc audit
├── /refactor (MAXIMUM aggression)    ← project-level cleanup
├── /review-arch                      ← architectural restructuring
├── /refactor (conditional)           ← post-restructuring cleanup
├── /review-test                      ← test suite review
├── /review-doc                       ← documentation audit
└── /review-release                   ← pre-release readiness
```

Planning feeds implementation. `/scope-project` plans a multi-batch
project with adversarial review, producing tagged tickets that `/implement-project`
consumes directly:

```
/scope-project  →  /implement-project
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
| Implement an entire multi-batch project autonomously    | `/implement-project`       |
| Implement a batch of related tickets                    | `/implement-batch`         |
| Implement a single ticket or feature                    | `/implement`       |
| Plan a multi-batch project with adversarial review      | `/scope-project` |
| Plan a single feature and create a ticket               | `/scope`         |
| Fix a bug with diagnosis and root-cause analysis        | `/bugfix`        |
| Make a hard decision with adversarial deliberation      | `/deliberate`    |
| Clean up code quality (DRY, dead code, naming)          | `/refactor`      |
| Rethink module boundaries and architecture              | `/review-arch`   |
| Review and strengthen the test suite                    | `/review-test`   |
| Verify test quality via mutation testing                | `/test-mutation`   |
| Audit all project documentation                         | `/review-doc`    |
| Pre-release readiness check                             | `/review-release`|
| Audit web content for accessibility barriers            | `/review-a11y`   |
| Assess code health across all project languages         | `/review-source` |
| Review performance (compute and/or web)                 | `/review-perf`   |
| Perform a white-box security audit                      | `/audit-source`  |

**Rules of thumb:**
- Multiple batches of tickets forming a project? `/implement-project`
- One batch of 2+ related tickets? `/implement-batch`
- One ticket? `/implement` (or `/bugfix` if it's a bug)
- Not sure what to build yet? Start with `/scope` or `/scope-project`

## Skills

### Orchestration

These workflows manage the lifecycle of tickets — from implementation
through quality passes to a merge-ready branch.

#### /implement-project — Full-Lifecycle Project Workflow

Orchestrates an entire project from tickets to release-ready code. Takes
batched tickets, implements each batch via `/implement-batch` in autonomous mode,
runs smoke tests, then executes a comprehensive quality pipeline (refactor,
review-arch, review-test, review-doc, review-release). The result is a
single project branch ready for human review and merge.

Maximizes autonomy — the andon cord (stop-the-line escalation) is the only
planned intervention path.

[Detailed documentation](skills/implement-project/SKILL.md)

#### /implement-batch — Multi-Ticket Orchestration

Takes a batch of tickets, plans their execution order, implements each
sequentially using `/implement` in autonomous mode, runs cross-cutting
quality passes (`/refactor`, `/review-doc`), and presents results for
final review.

[Detailed documentation](skills/implement-batch/SKILL.md)

#### /implement — Single-Ticket Development

Orchestrates a complete development cycle through specialist agents:
requirements → planning → implementation → QA → code review →
documentation. Detects project type and dispatches to language-specific
SMEs (Go, GraphQL, Docker, Makefile, Ansible, Zig, HTML, CSS,
JavaScript, TypeScript).

[Detailed documentation](skills/implement/SKILL.md)

### Planning

These workflows explore problem spaces and produce well-specified tickets
without doing implementation work.

#### /scope-project — Adversarial Project Planning

Plans an entire project through adversarial review. Explores the problem
space, drafts tickets organized into batches, then pits a planner against
an implementer agent to find gaps, ambiguities, and missing work. Only
when the implementer is satisfied do tickets go upstream — already tagged
with batch labels ready for `/implement-project` to consume.

[Detailed documentation](skills/scope-project/SKILL.md)

#### /scope — Problem Space Exploration

Explores problem spaces through iterative dialogue and codebase analysis,
then creates a detailed ticket in your issue tracker. For single features,
bug investigations, or refactoring proposals.

[Detailed documentation](skills/scope/SKILL.md)

### Quality

These workflows improve code, tests, architecture, and documentation.
They run as part of `/implement-project`'s quality pipeline, but each works
standalone too.

#### /refactor — Iterative Code Quality Improvement

Autonomously scans for tactical improvements (DRY violations, dead code,
naming issues, unnecessary complexity), implements through specialist
agents with QA verification, and loops until no improvements remain. Works
within existing architecture — for structural changes, use `/review-arch`.

[Detailed documentation](skills/refactor/SKILL.md)

#### /review-arch — Blueprint-Driven Architectural Improvement

Analyzes codebase architecture via noun analysis, produces a target
blueprint, then collaborates with the user to decide what to implement.
For module boundaries, responsibility overlap, utility grab-bag
dissolution, and structural rethinking.

[Detailed documentation](skills/review-arch/SKILL.md)

#### /review-test — Comprehensive Test Suite Review

Three-phase review: fills coverage gaps, identifies missing fuzz tests,
and audits test quality. Each phase has its own analysis → present →
select → implement → verify cycle.

[Detailed documentation](skills/review-test/SKILL.md)

#### /test-mutation — Mutation Testing

Systematically introduces mutations into source code and checks if tests
catch them. Surviving mutations reveal genuine coverage gaps that line
coverage misses. Multi-session with progress tracking.

[Detailed documentation](skills/test-mutation/SKILL.md)

#### /review-doc — Documentation Quality Audit

Comprehensively reviews all project documentation for correctness,
completeness, and freshness. Fixes issues autonomously within its
authority.

[Detailed documentation](skills/review-doc/SKILL.md)

#### /review-release — Pre-Release Readiness Check

Pre-flight check before cutting a release. Scans for debug artifacts,
version mismatches, changelog gaps, git hygiene issues, breaking API
changes, and license compliance. Interactive — presents findings and lets
you decide what to fix.

[Detailed documentation](skills/review-release/SKILL.md)

#### /review-a11y — Accessibility Audit

Audits web content against WCAG 2.2 Level AA. Detects web content files
(HTML, JSX/TSX, Vue, Svelte, CSS, templates), dispatches accessibility
auditor agents to evaluate conformance, and produces a consolidated
report prioritized by real-world user impact. Advisory only — no
changes made.

[Detailed documentation](skills/review-a11y/SKILL.md)

#### /review-source — Code Health Assessment

Assesses source code health across all languages in the project. Detects
languages, dispatches SME agents for specialist review (or generalists for
unsupported languages), and produces a consolidated health report with
per-language ratings. Advisory only — no changes made. Use to decide
whether `/refactor` is needed.

[Detailed documentation](skills/review-source/SKILL.md)

#### /review-perf — Performance Review

Reviews a project for performance issues across two domains: compute
performance (algorithms, memory, CPU, benchmarking) and web performance
(caching, asset delivery, loading strategy, Core Web Vitals). Detects the
project type and dispatches the appropriate specialist(s) in parallel.
Advisory only — no changes made.

[Detailed documentation](skills/review-perf/SKILL.md)

### Security

#### /audit-source — White-Box Security Audit

Orchestrates a comprehensive security assessment of the project's source code
using both defensive and offensive analysis. A blue-teamer evaluates the
defensive posture first, then red-teamers attack informed by the defensive
gaps. Dedicated red-teamers investigate each attack vector in depth. Findings
are synthesized, exploit chains are explored, and the process iterates until
no new chains emerge. Heavy and thorough by design.

### Decision and Diagnosis

#### /deliberate — Adversarial Decision Making

Uses adversarial representation to make decisions. Spawns advocate agents
for each option who argue their cases, rebut each other, and respond to
probing questions before a judge renders a verdict with reasoning and
trade-offs.

[Detailed documentation](skills/deliberate/SKILL.md)

#### /bugfix — Diagnosis-First Bug Fixing

Coordinates specialist agents through a diagnosis-first bug-fixing cycle:
reproduce with a failing test, perform root-cause analysis with git
archaeology, implement a targeted fix, and verify. Same review pipeline as
`/implement`.

[Detailed documentation](skills/bugfix/SKILL.md)

## Agents

Specialist agents spawned by the workflows above:

| Agent                       | Purpose                                                                                               |
|-----------------------------|-------------------------------------------------------------------------------------------------------|
| `advocate`                  | Argues for a specific option in deliberation proceedings                                              |
| `swe-planner`               | Decomposes complex tasks into implementation plans                                                    |
| `swe-sme-golang`            | Go implementation specialist                                                                          |
| `swe-sme-graphql`           | GraphQL schema and resolver specialist                                                                |
| `swe-sme-docker`            | Dockerfile and container specialist                                                                   |
| `swe-sme-makefile`          | Makefile and build system specialist                                                                  |
| `swe-sme-ansible`           | Ansible automation specialist                                                                         |
| `swe-sme-zig`               | Zig implementation specialist                                                                         |
| `swe-sme-html`              | HTML structure, semantics, and accessibility specialist                                               |
| `swe-sme-css`               | CSS styling, layout, and responsive design specialist                                                 |
| `swe-sme-javascript`        | Vanilla JavaScript implementation specialist                                                          |
| `swe-sme-typescript`        | TypeScript implementation and type design specialist                                                  |
| `swe-code-reviewer`         | Tactical code quality reviewer (DRY, dead code, naming, complexity)                                   |
| `swe-arch-reviewer`         | Architecture reviewer (noun analysis, module boundaries, blueprints)                                  |
| `swe-bug-investigator`      | Bug root-cause investigator (execution tracing, git archaeology, diagnosis reports)                    |
| `swe-perf-reviewer`         | Compute performance reviewer (algorithmic complexity, benchmarking, profiling, optimization)           |
| `swe-web-perf-reviewer`     | Web performance reviewer (caching, asset delivery, loading strategy, Core Web Vitals)                  |
| `qa-engineer`               | Practical verification and test coverage                                                              |
| `qa-web-a11y-reviewer`      | WCAG accessibility reviewer (keyboard navigation, ARIA, contrast, semantic structure)                  |
| `qa-test-reviewer`          | Test quality reviewer (brittle, tautological, useless tests)                                          |
| `qa-test-coverage-reviewer` | Coverage gap reviewer (coverage reports, risk prioritization, testability suggestions)                 |
| `qa-test-fuzz-reviewer`     | Fuzz testing gap reviewer (fuzz infrastructure detection, candidate identification)                    |
| `qa-test-mutator`           | Mutation testing worker (applies mutations, records results)                                           |
| `qa-release-engineer`       | Pre-release scanner (debug artifacts, versioning, changelog, git hygiene, breaking changes, licenses)  |
| `sec-blue-teamer`           | Defensive security analyst (control inventory, consistency, defense-in-depth, configuration)            |
| `sec-red-teamer`            | Adversarial security analyst (attack surface mapping, exploitation, trust boundary analysis)            |
| `doc-maintainer`            | Documentation updates and verification                                                                 |

## Development

See [HACKING.md](HACKING.md) for local development and testing instructions.

## Versioning

This project follows [Semantic Versioning](https://semver.org/). Skills (slash commands like `/implement`, `/review-perf`, etc.) are the public interface. Subagent names are internal implementation details and may be renamed or restructured without constituting a breaking change.

## Requirements

- `git` repository
- For ticket creation: integration with your issue tracker (CLI, MCP server, or API)

[cc]: https://claude.ai/code
