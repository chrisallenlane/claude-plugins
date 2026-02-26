# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a Claude Code plugin marketplace repository containing distributable plugins. Each plugin provides skills and/or agents that extend Claude Code's capabilities.

## Plugin Structure

Each plugin is a subdirectory with this structure:

```
plugin-name/
├── .claude-plugin/
│   └── plugin.json      # Plugin metadata (name, version, description, license)
├── agents/              # Agent definitions (optional)
│   └── agent-name.md    # Agent prompt with YAML frontmatter
├── skills/              # Skill definitions (optional)
│   └── skill-name/
│       └── SKILL.md     # Skill prompt with YAML frontmatter
└── README.md            # Plugin documentation (optional)
```

The root `.claude-plugin/marketplace.json` registers all plugins for distribution.

## Development

Test a plugin locally:

```bash
claude --plugin-dir ./plugin-name
```

## Writing Plugins

**Skills** (in `skills/*/SKILL.md`):
- YAML frontmatter: `name`, `description`, `model` (opus/sonnet/haiku - lowercase)
- Invoked by user with `/skill-name`
- Define workflows, processes, or specialized behaviors

**Agents** (in `agents/*.md`):
- YAML frontmatter: `name`, `description`, `model` (lowercase)
- Spawned programmatically via `Task` tool with `subagent_type`
- Operate autonomously within defined scope

**Model names must be lowercase** (`opus`, `sonnet`, `haiku`) - capitalized names are not recognized.

## Workflow

The plugins form a three-stage development workflow:

```
/deliberate  →  /scope  →  /iterate
   decide        plan      implement
```

**deliberate**: Adversarial decision-making through advocate agents.

**swe**: Software engineering workflow with `/scope` (planning), `/iterate` (implementation), `/refactor` (tactical cleanup), `/arch-review` (architectural restructuring), `/test-audit` (test quality review), and `/test-mutate` (mutation testing) skills, plus specialist agents.
