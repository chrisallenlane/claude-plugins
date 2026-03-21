# /review-perf - Performance Review

## Overview

The `/review-perf` skill reviews a project for performance issues across two domains: **compute performance** (algorithms, memory, CPU, benchmarking) and **web performance** (caching, asset delivery, loading strategy, Core Web Vitals). It detects the project type and dispatches the appropriate specialist(s).

**Key benefits:**
- Automatic domain detection: dispatches web reviewer, compute reviewer, or both based on project content
- Two specialized lenses: network-bound performance and compute-bound performance are fundamentally different — each gets a specialist
- Cross-domain synthesis: identifies issues that span both domains (e.g., SSR bottlenecks that are both compute and loading problems)

**This is a diagnostic tool.** It identifies performance problems and recommends fixes. It does not implement them.

## When to Use

**Use `/review-perf` for:**
- Evaluating performance posture of a project
- Pre-release performance checks
- Auditing after significant architectural or feature changes
- Understanding where time is spent (network vs. compute)

**Don't use `/review-perf` for:**
- Implementing optimizations (ask directly after reviewing findings, or use `/implement`)
- Accessibility review (use `/review-a11y`)
- General code quality (use `/review-source`)
- Security review (use `/audit-source`)

## Workflow

```
┌──────────────────────────────────────────────────────┐
│                 PERFORMANCE REVIEW                   │
├──────────────────────────────────────────────────────┤
│  1. Detect project type (web, non-web, or both)      │
│  2. Determine scope                                  │
│  3. Dispatch performance reviewer(s) in parallel     │
│  4. Present consolidated report                      │
└──────────────────────────────────────────────────────┘
```

### 1. Detect Project Type

Scans for web indicators (HTML/JSX/TSX, web frameworks, build tool configs, service workers) and compute indicators (backend source files, existing benchmarks, database code). Full-stack projects trigger both reviewers.

### 2. Determine Scope

User chooses what to review: both domains (default), web only, compute only, or a specific directory.

### 3. Dispatch Reviewers

Applicable reviewers run in parallel. Each performs a comprehensive review within its domain and produces findings classified by severity.

### 4. Consolidated Report

Findings from both domains merged into a single report with a cross-cutting analysis section that identifies issues spanning both domains.

## Agents Used

| Agent                   | Domain                                                              |
|-------------------------|---------------------------------------------------------------------|
| `swe-perf-reviewer`     | Compute performance (algorithms, memory, benchmarks, profiling)     |
| `swe-web-perf-reviewer` | Web performance (caching, assets, loading, Core Web Vitals)         |

## Resource Usage

Lightweight. Reviewer agents are read-only and run in parallel. Non-destructive.
