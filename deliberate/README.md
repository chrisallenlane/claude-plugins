# deliberate

Adversarial deliberation process for making decisions. Spawns advocate agents
for each option who argue their cases before a judge (Claude) who renders a
verdict.

Inspired by the legal adversarial system where truth emerges from the collision
of well-argued positions.

## Installation

```bash
claude plugin install deliberate@chrisallenlane
```

Or for development:

```bash
claude --plugin-dir /path/to/deliberate
```

## Usage

```
/deliberate <decision and options>
```

Examples:

```
/deliberate Should we use PostgreSQL or MongoDB for our new project?

/deliberate Pick a web framework: Django, FastAPI, or Flask

/deliberate Build vs buy for our authentication system
```

## How It Works

### Roles

**Judge (Claude):** The main Claude instance running the skill. Identifies
options, spawns advocates, asks probing questions, and renders final judgment.

**Advocates:** Subagents spawned for each option. They argue vigorously for
their assigned option, research and gather evidence, rebut opposing arguments,
and acknowledge genuine weaknesses when challenged (good faith requirement).

### Workflow

1. **Parse the Decision** - Identify options, criteria, and context
2. **Fact-Finding** - Proactively gather context through clarifying questions
3. **Spawn Advocates** - Launch one advocate agent per option (parallel)
4. **Initial Arguments** - Advocates present their cases
5. **Rebuttal Round** - Advocates counter opposing arguments
6. **Judge's Questions** - Probe weaknesses and test claims
7. **Pre-Judgment Disclosure** - Judge shares current thinking; final responses
8. **Iterate or Conclude** - Return to questioning or proceed to judgment
9. **Render Judgment** - Present recommendation with reasoning and trade-offs

The judge may iterate through steps 5-7 up to 10 times before rendering
judgment.

## When to Use

**Good fit:**
- Vendor/tool/library selection
- Architectural decisions with multiple valid approaches
- Build vs buy decisions
- Technology stack choices
- Strategic decisions with trade-offs

**Poor fit:**
- Decisions with a clearly correct answer
- Simple preferences (just ask directly)
- Decisions requiring real-world testing to resolve
- Ethical dilemmas (advocacy framing may be inappropriate)

## Output

The judgment includes:

- **Recommendation** - The winning option
- **Reasoning** - Why this option wins (2-3 key factors)
- **Trade-offs** - What you're sacrificing by not choosing alternatives
- **Confidence** - High/Medium/Low with explanation
- **Key factors** - What most influenced the decision

If the judge cannot decide (options are genuinely equivalent, or critical
information is unavailable), it will present the key factors and explain what
the decision hinges on.

## Philosophy

The adversarial process ensures:

- **All options get their best case made** - no option dismissed prematurely
- **Weaknesses are exposed** - advocates attack each other's positions
- **Evidence is gathered** - advocates research to support their cases
- **Trade-offs are explicit** - the collision of arguments reveals what you're giving up

Advocates argue in good faith: they may emphasize strengths and frame facts
favorably, but they must not fabricate evidence, deny clear weaknesses, or use
fallacious reasoning.

## License

MIT
