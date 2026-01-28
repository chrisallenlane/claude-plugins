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

Explores problem spaces through iterative dialogue and codebase analysis, then creates detailed tickets in Gitea/GitHub.

**Use when:**
- Planning a complex feature before implementation
- Investigating a bug and documenting findings
- Exploring refactoring opportunities
- Creating well-specified tickets for later implementation

**Key principle:** `/scope` explores and documents. It does NOT implement.

[Detailed documentation](skills/scope/README.md)

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
| `sec-reviewer` | Security vulnerability analysis |
| `doc-maintainer` | Documentation updates and verification |

## Workflow Integration

`/scope` and `/iterate` are complementary:

```
/scope  →  Creates detailed ticket  →  /iterate  →  Implements the ticket
```

Use `/scope` first for complex changes that need exploration, then `/iterate` to implement. For straightforward changes, use `/iterate` directly.

## Requirements

- `git` repository
- For ticket creation: `tea` (Gitea) or `gh` (GitHub) CLI
