# Claude Code Plugins

Personal Claude Code plugins by Chris Allen Lane.

## Installation

Add this repository as a marketplace:

```bash
claude plugin marketplace add https://github.com/chrisallenlane/claude-plugins
```

Then install plugins:

```bash
claude plugin install deliberate@chrisallenlane
```

## Development

Test a plugin locally:

```bash
claude --plugin-dir ./deliberate
```

## Plugins

### tea

Reference skill for the Gitea CLI interface (`tea`). Provides command syntax
for working with issues and comments.

**Usage:** `/tea` to see available commands.

---

### deliberate

Adversarial deliberation process for making decisions. Spawns advocate agents
for each option who argue their cases before a judge (Claude) who renders a
verdict.

**Usage:** `/deliberate` followed by your decision and options.

**Good for:**
- Vendor/tool/library selection
- Architectural decisions
- Build vs buy decisions
- Technology stack choices

**How it works:**
1. Parse the decision and identify options
2. Spawn advocate agents (one per option)
3. Advocates present arguments in parallel
4. Rebuttal round
5. Judge asks probing questions
6. Render judgment with reasoning and trade-offs
