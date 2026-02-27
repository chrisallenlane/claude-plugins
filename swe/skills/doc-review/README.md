# /doc-review - Documentation Quality Audit

## Overview

The `/doc-review` skill spawns a doc-maintainer agent to comprehensively review all project documentation. It audits every documentation file for correctness, completeness, freshness, and consistency — then fixes what it can and reports what needs your approval.

**Key benefits:**
- Catches stale docs, broken links, and code-documentation drift
- Autonomous fixes within the agent's authority
- Comprehensive (reviews all docs, not just recently changed ones)
- Offers to commit changes when done

## When to Use

**Use `/doc-review` for:**
- After a major feature or refactoring pass
- Before a release (catch stale docs)
- Periodic documentation hygiene
- When you suspect docs have drifted from the code
- As part of `/refactor` completion (it runs automatically)

**Don't use `/doc-review` for:**
- Updating docs for a specific change (the `/iterate` and `/fix` workflows do this as part of their flow)
- Writing new documentation from scratch (just ask directly)

## Workflow

```
┌─────────────────────────────────────────────────────┐
│                   DOC REVIEW                        │
├─────────────────────────────────────────────────────┤
│  1. Spawn doc-maintainer agent                      │
│     • Discovers all .md files in project            │
│     • Reviews each for quality checklist:           │
│       - Code-documentation consistency              │
│       - Completeness                                │
│       - Link validation                             │
│       - Style consistency                           │
│       - Freshness                                   │
│     • Fixes issues autonomously (within authority)  │
│  2. Report results to user                          │
│     • What was reviewed                             │
│     • What was changed                              │
│     • Issues requiring user approval                │
│  3. Commit (optional, if changes were made)         │
└─────────────────────────────────────────────────────┘
```

## Example Session

```
> /doc-review

Spawning doc-maintainer agent for comprehensive review...

## Documentation Audit Report

### Changes Made
1. README.md — Added missing /fix skill to skills list
2. swe/README.md — Added swe-diagnostician to agents table

### Issues Requiring User Approval
- Three skills missing README.md files (doc-review, fix, project)

### No Issues Found
- Link validation: all links resolve
- Style consistency: headings, code blocks, terminology all consistent
- Freshness: no references to removed features

Commit documentation fixes?
> Yes

Committed: "docs: review and update project documentation"
```

## Tips

1. **Run after `/refactor` or `/arch-review`.** These skills change code structure, which often makes docs stale.

2. **Run before releases.** Stale docs in a release are embarrassing. A quick `/doc-review` catches drift.

3. **Different from `/iterate` step 9.** The `/iterate` workflow's documentation step is scoped to the git diff. `/doc-review` audits everything regardless of recent changes.
