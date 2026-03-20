# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is `claude-swe-workflows`, a Claude Code plugin for software engineering workflows. It provides skills and agents that extend Claude Code's capabilities for systematic development.

## Plugin Structure

```
├── .claude-plugin/
│   ├── marketplace.json # Marketplace manifest (lists plugins for installation)
│   └── plugin.json      # Plugin metadata (name, version, description, license)
├── agents/              # Agent definitions
│   └── agent-name.md    # Agent prompt with YAML frontmatter
├── skills/              # Skill definitions
│   └── skill-name/
│       └── SKILL.md     # Skill prompt with YAML frontmatter
└── README.md
```

## Development

Test the plugin locally:

```bash
claude --plugin-dir .
```

## Writing Skills and Agents

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

The skills form a layered system. Higher-level workflows orchestrate lower-level ones:

```
/implement-project
├── /implement-batch (per batch)
│   └── /implement (per ticket)
└── quality pipeline: /refactor, /review-arch, /review-test, /review-doc, /review-release
```

Planning feeds implementation: `/scope-project` → `/implement-project`, or `/scope` → `/implement`.

Supporting workflows available at any level: `/deliberate` (adversarial decision-making), `/bugfix` (diagnosis-first bug fixing), `/test-mutation` (mutation testing).
