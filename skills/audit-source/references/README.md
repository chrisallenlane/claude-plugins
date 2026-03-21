# /audit-source - White-Box Security Audit

## Overview

The `/audit-source` skill orchestrates a comprehensive security assessment of the project's source code using both defensive and offensive analysis. A blue-teamer evaluates the defensive posture first, then a lead red-teamer performs reconnaissance informed by the defensive gaps. Dedicated red-teamers investigate each attack vector in depth. Findings are synthesized, exploit chains are explored, and the process iterates until no new chains emerge.

**Key benefits:**
- Dual-perspective analysis: defensive evaluation (blue team) feeds offensive reconnaissance (red team)
- Deep investigation: each attack vector gets a dedicated agent with full context
- Chain discovery: findings are cross-referenced to identify multi-step exploit chains
- Iterative convergence: the process loops until no new chains emerge
- Actionable output: findings include concrete attacks, not abstract warnings

## When to Use

**Use `/audit-source` for:**
- Pre-release security assurance on important releases
- After significant feature additions that change the attack surface
- When onboarding a new codebase and you want to understand its security posture
- Periodic security audits (quarterly, annually, etc.)
- When you need to understand both what's exploitable and why the defenses failed

**Don't use `/audit-source` for:**
- Routine development (the blue-teamer runs automatically during `/implement` and `/bugfix`)
- Quick security sanity checks (spawn `sec-blue-teamer` directly)
- Runtime security testing (this is static source analysis only)

**Rule of thumb:** If you'd hire a pentester for this, `/audit-source` is the right tool. If you just want a security review of your latest changes, `/implement` and `/bugfix` already include one.

## Workflow

```
┌──────────────────────────────────────────────────────┐
│                   AUDIT WORKFLOW                     │
├──────────────────────────────────────────────────────┤
│  1. Determine scope                                  │
│  2. Spawn blue-teamer (defense evaluation)           │
│     └─ Output: control inventory + gaps + depth      │
│  3. Spawn lead red-teamer (reconnaissance)           │
│     └─ Input: blue-teamer's defense evaluation       │
│     └─ Output: attack surface + ranked vector list   │
│  4. For each high-confidence vector:                 │
│     └─ Spawn focused red-teamer (deep investigation) │
│  5. Synthesize findings                              │
│     ├─ If exploit chains found → goto 4 (new vector) │
│     └─ If no new chains → proceed                    │
│  6. Present consolidated findings to user            │
│  7. Optionally route findings to fixers              │
└──────────────────────────────────────────────────────┘
```

### 1. Determine Scope

The skill asks about scope, areas of concern, and areas to skip. By default, the audit targets production code only — test code, dev-only dependencies, generated code, and vendored code are excluded. Users can override these defaults. User concerns inform prioritization but don't replace systematic analysis.

### 2. Defense Evaluation (Blue Team)

A `sec-blue-teamer` agent performs a full defense evaluation: control inventory, consistency checking, defense-in-depth assessment, configuration review, dependency hygiene, and secrets audit. Its report is passed directly to the lead red-teamer.

### 3. Reconnaissance (Red Team Lead)

A `sec-red-teamer` agent maps the attack surface and produces a prioritized target list. The blue team's findings make reconnaissance significantly more targeted — known defensive gaps become priority targets.

### 4. Deep Investigation (Focused Red-Teamers)

Each high-priority target gets a dedicated `sec-red-teamer` agent with a full context window focused on that single attack vector. Agents run sequentially so findings accumulate for chain analysis.

### 5. Chain Analysis

Findings are cross-referenced to identify exploit chains — combinations of individually low/medium-severity findings that together create a high/critical-severity exploit. New chains trigger additional focused investigation. The loop converges when no new chains are found (capped at 3 iterations).

### 6. Consolidated Report

A single report combining the blue team's defensive assessment, the red team's offensive findings, and any exploit chains discovered. Presented interactively — CRITICAL findings first.

### 7. Remediation Routing (Optional)

Findings can be routed to appropriate SME agents for implementation. Each fix is verified by `qa-engineer` and committed atomically.

## Agents Used

 | Agent             | Role                                              |
 | ----------------- | ------------------------------------------------- |
 | `sec-blue-teamer` | Defensive posture evaluation                      |
 | `sec-red-teamer`  | Offensive reconnaissance and exploitation         |
 | `swe-sme-*`       | Implement remediation fixes (optional)            |
 | `qa-engineer`     | Verify fixes don't break functionality (optional) |

## Resource Usage

This skill is deliberately heavy. A full audit of a medium-sized codebase may spawn 5-10+ agents and take significant time. That's by design — shallow security reviews miss the vulnerabilities that matter.
