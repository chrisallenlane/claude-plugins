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

## Current Plugins

**deliberate**: Adversarial decision-making through advocate agents. Judge spawns advocates for each option, runs arguments/rebuttals, renders judgment.

**tea**: Reference skill for Gitea CLI commands.

**swe**: Software engineering workflow with `/iterate` and `/scope` skills, plus specialist agents for Go, Docker, GraphQL, Makefile, Ansible, Zig, and QA/security review.
