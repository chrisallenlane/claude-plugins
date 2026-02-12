# Claude Code Plugins

A cohesive software engineering workflow for Claude Code.

## Workflow

These plugins form a three-stage development workflow:

```
/deliberate  →  /scope  →  /iterate
   decide        plan      implement
```

1. **`/deliberate`** - Arrive at technical decisions through adversarial
   deliberation. Advocate agents argue for each option before a judge renders
   a verdict.

2. **`/scope`** - Explore the problem space through dialogue and codebase
   analysis. Creates detailed tickets for implementation.

3. **`/iterate`** - Execute the work through specialist agents: planning →
   implementation → QA → security review → documentation.

Each stage feeds into the next. Use `/deliberate` to resolve architectural
questions, `/scope` to break decisions into actionable tickets, and `/iterate`
to implement them systematically.

For straightforward work, enter the workflow at any stage. Simple features can
go directly to `/iterate`. Clear requirements can skip `/deliberate`.

## Installation

Add this repository as a marketplace:

```bash
claude plugin marketplace add https://github.com/chrisallenlane/claude-plugins
```

Then install plugins:

```bash
claude plugin install deliberate@chrisallenlane
claude plugin install swe@chrisallenlane
```

## Plugins

### deliberate

Adversarial decision-making. Spawns advocate agents for each option who argue
their cases before a judge (Claude) renders a verdict.

**Good for:** Vendor selection, architectural decisions, build vs buy,
technology choices.

[Documentation](deliberate/README.md)

---

### swe

Software engineering workflow with specialist agents.

**Skills:**
- `/iterate` - Full development cycle through specialist agents
- `/scope` - Problem exploration and ticket creation
- `/refactor` - Autonomous iterative codebase improvement
- `/test-audit` - Interactive test suite quality review
- `/test-mutate` - Mutation testing workflow (verify tests catch bugs)

**Agents:** Specialists for Go, GraphQL, Docker, Makefile, Ansible, Zig, plus
QA, security, performance, refactoring, and documentation.

[Documentation](swe/README.md)

## Development

Test a plugin locally:

```bash
claude --plugin-dir ./deliberate
```
