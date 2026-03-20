# /review-source - Code Health Assessment

## Overview

The `/review-source` skill assesses source code health across all languages in a project. It detects which languages are present, dispatches language-specific SME agents (or generalists for unsupported languages) to review code quality, and produces a consolidated health report.

**Key benefits:**
- Language-aware: each language is reviewed by a specialist who knows its idioms and best practices
- Generalist fallback: languages without SMEs still get reviewed for universal quality issues
- Summary-level output: health ratings and narrative assessments, not line-by-line findings
- Actionable: clearly indicates whether refactoring is recommended and suggests next steps

**This is a diagnostic tool.** It tells you the state of your code. It does not make changes. Use `/refactor` to act on findings.

## When to Use

**Use `/review-source` for:**
- Getting a health check on a codebase you're unfamiliar with
- Deciding whether to run `/refactor`
- After a long period of development to assess accumulated quality debt
- Before major releases to check overall code health
- When onboarding to a project and wanting to understand code quality

**Don't use `/review-source` for:**
- Finding specific bugs (use `/bugfix` or `/audit-source`)
- Making changes (use `/refactor`)
- Reviewing architecture (use `/review-arch`)
- Reviewing test quality (use `/review-test`)
- Security assessment (use `/audit-source`)

**Rule of thumb:** If you want to know "how healthy is this code?" — use `/review-source`. If you want to fix what's wrong — use `/refactor`.

## Workflow

```
┌──────────────────────────────────────────────────────┐
│                  SOURCE REVIEW                       │
├──────────────────────────────────────────────────────┤
│  1. Detect languages in project                      │
│  2. User selects which languages to review           │
│  3. Dispatch review agents (parallel)                │
│     ├─ SME agent for supported languages             │
│     └─ Generalist for unsupported languages          │
│  4. Aggregate and present health report              │
└──────────────────────────────────────────────────────┘
```

### 1. Detect Languages

Scans for source files, maps extensions to languages, and identifies which languages have SME agents available.

### 2. User Selection

User chooses which languages to review (default: all) and optionally narrows scope to specific directories.

### 3. Dispatch Reviews

One agent per language, all running in parallel. SME agents review for language-specific quality (idiomatic usage, modernness, conventions). Generalists review for universal quality (naming, structure, complexity, anti-patterns).

### 4. Health Report

Consolidated report with per-language health ratings (Excellent/Good/Fair/Poor), narrative summaries, refactoring recommendations, and suggested next steps.

## Agents Used

| Agent             | Role                                                 |
|-------------------|------------------------------------------------------|
| `swe-sme-*`      | Language-specific code review (10 languages supported) |
| `general-purpose` | Generalist review for unsupported languages           |

## Supported Languages

Go, Zig, JavaScript, TypeScript, HTML, CSS, GraphQL, Docker, Makefile, Ansible — all have dedicated SME agents. All other languages are reviewed by a generalist.

## Resource Usage

Lightweight. Review agents are read-only and run in parallel. A project with 5 languages spawns 5 agents simultaneously — fast and non-destructive.
