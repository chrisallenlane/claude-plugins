# /review-a11y - Accessibility Audit

## Overview

The `/review-a11y` skill audits web content against WCAG 2.2 Level AA. It detects web content files (HTML, JSX/TSX, Vue, Svelte, CSS, templates), dispatches accessibility auditor agents, and produces a consolidated report prioritized by real-world user impact.

**Key benefits:**
- Impact-focused: issues are classified by how severely they affect people using assistive technology, not by abstract compliance checklists
- Automated + manual: runs automated tooling (axe-core, pa11y) when available, then layers manual inspection for issues tools miss
- Actionable: every finding includes the affected users, the WCAG criterion, and a specific fix recommendation

**This is a diagnostic tool.** It identifies accessibility barriers and recommends fixes. It does not implement them.

## When to Use

**Use `/review-a11y` for:**
- Evaluating accessibility posture of a web project
- Pre-release checks for WCAG compliance
- Auditing after significant UI changes
- Understanding what assistive technology users experience

**Don't use `/review-a11y` for:**
- Non-web projects (CLI tools, APIs, libraries without UI)
- Implementing fixes (just ask directly after reviewing findings)
- General code quality (use `/review-source`)
- Security review (use `/audit-source`)

**Rule of thumb:** If you want to know "can people with disabilities use this?" — use `/review-a11y`.

## Workflow

```
┌──────────────────────────────────────────────────────┐
│               ACCESSIBILITY REVIEW                   │
├──────────────────────────────────────────────────────┤
│  1. Detect web content                               │
│  2. Determine scope                                  │
│  3. Dispatch accessibility auditor(s)                │
│  4. Present consolidated audit report                │
└──────────────────────────────────────────────────────┘
```

### 1. Detect Web Content

Scans for web content files: HTML, JSX/TSX, Vue, Svelte, CSS/SCSS/Less, and HTML templates. Aborts early if no web content is found.

### 2. Determine Scope

User chooses what to audit: entire project (default), a specific directory, or specific files.

### 3. Dispatch Auditors

For small scopes, a single accessibility auditor agent. For large scopes, partitioned by directory with multiple agents running in parallel. Each agent runs automated tooling (if available) and performs manual inspection covering keyboard navigation, semantic structure, ARIA usage, dynamic content, color contrast, and media accessibility.

### 4. Audit Report

Consolidated report with issues classified as CRITICAL (blocks access entirely), HIGH (significantly degrades experience), or LOW (minor issue or enhancement). Each finding includes the affected user group, the relevant WCAG criterion, and a specific remediation recommendation.

## Agents Used

| Agent                  | Role                                              |
|------------------------|---------------------------------------------------|
| `qa-web-a11y-reviewer` | WCAG conformance audit with impact prioritization |

## Resource Usage

Lightweight. Auditor agents are read-only and run in parallel. Non-destructive.
