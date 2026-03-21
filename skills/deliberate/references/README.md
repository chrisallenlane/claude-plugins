# /deliberate - Adversarial Decision Making

## Overview

The `/deliberate` skill uses adversarial representation to make decisions. Inspired by the legal adversarial system, it spawns advocate agents who argue for each option, rebut each other's positions, and respond to probing questions. You (the judge) render a verdict with reasoning and trade-offs.

**Key benefits:**
- Every option gets its best case argued — no option dismissed prematurely
- Weaknesses exposed through direct rebuttal
- Advocates research and gather evidence to support their cases
- Trade-offs made explicit through collision of well-argued positions
- Judge can probe weak points and challenge assumptions

## When to Use

**Use `/deliberate` for:**
- Vendor/tool/library selection
- Architectural decisions with multiple valid approaches
- Build vs buy decisions
- Technology stack choices
- Strategic decisions with trade-offs

**Don't use `/deliberate` for:**
- Decisions with a clearly correct answer
- Simple preferences (just ask directly)
- Decisions requiring real-world testing to resolve
- Ethical dilemmas (advocacy framing may be inappropriate)

## Workflow Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│ /deliberate Workflow                                             │
└─────────────────────────────────────────────────────────────────┘

 ┌──────────────────────────────────────────────┐
 │  1. PARSE THE DECISION                       │
 │  ────────────────────────────────────────    │
 │  • Identify options (minimum 2)              │
 │  • Determine criteria                        │
 │  • Gather relevant context                   │
 └──────────────────┬───────────────────────────┘
                    ▼
 ┌──────────────────────────────────────────────┐
 │  2. FACT-FINDING                             │
 │  ────────────────────────────────────────    │
 │  Probe for:                                  │
 │  • Constraints, budget, timeline             │
 │  • History — what's been tried or ruled out  │
 │  • Non-negotiables vs nice-to-haves          │
 │  • Success criteria                          │
 │  (Typically 3-5 clarifying questions)        │
 └──────────────────┬───────────────────────────┘
                    ▼
 ┌──────────────────────────────────────────────┐
 │  3. SPAWN ADVOCATES (parallel)               │
 │  ────────────────────────────────────────    │
 │  One advocate agent per option               │
 │  Each presents initial argument              │
 └──────────────────┬───────────────────────────┘
                    ▼
 ┌──────────────────────────────────────────────┐
 │  4. REBUTTAL ROUND (parallel)                │
 │  ────────────────────────────────────────    │
 │  Each advocate sees all arguments            │
 │  Each responds with rebuttals                │
 └──────────────────┬───────────────────────────┘
                    ▼
 ┌──────────────────────────────────────────────┐
 │  5. JUDGE'S QUESTIONS (optional)             │
 │  ────────────────────────────────────────    │
 │  Probe weaknesses, request clarification,    │
 │  test how advocates handle challenges        │
 └──────────────────┬───────────────────────────┘
                    ▼
 ┌──────────────────────────────────────────────┐
 │  6. PRE-JUDGMENT DISCLOSURE                  │
 │  ────────────────────────────────────────    │
 │  Judge shares current leaning and reasoning  │
 │  Advocates get final opportunity to respond  │
 └──────────────────┬───────────────────────────┘
                    ▼
        ┌───────────────────────┐
        │  7. ITERATE OR        │
        │     CONCLUDE          │
        └───────────┬───────────┘
                    │
           ┌────────┴────────┐
           ▼                 ▼
   New questions?       Sufficient info
   → Back to step 5     → Render judgment
   (max 10 rounds)

 ┌──────────────────────────────────────────────┐
 │  8. RENDER JUDGMENT                          │
 │  ────────────────────────────────────────    │
 │  • Verdict: which option wins                │
 │  • Primary reasoning (2-3 key factors)       │
 │  • Trade-offs acknowledged                   │
 │  • Confidence level (High/Medium/Low)        │
 │                                              │
 │  OR: Unable to decide → present factors      │
 │  the user must weigh personally              │
 └──────────────────────────────────────────────┘
```

## Roles

**Judge (the orchestrator):**
- Identifies options and criteria
- Spawns advocate agents
- Listens to arguments and rebuttals
- Asks probing questions
- Renders final judgment (or admits inability to decide)

**Advocates (spawned agents):**
- Argue vigorously for their assigned option
- Research and gather evidence
- Rebut opposing arguments
- Acknowledge genuine weaknesses when challenged (good faith requirement)

## Example Session

```
> /deliberate Redis vs Memcached for our session store

What decision needs to be made?
Session store for a web application — Redis vs Memcached.

[Fact-finding: 3 questions about scale, persistence needs, existing infra]

Spawning advocates...

[Advocate A - Redis]:
Redis offers persistence, data structures beyond key-value,
pub/sub for session events, and Lua scripting...

[Advocate B - Memcached]:
Memcached is simpler, faster for pure key-value at scale,
multi-threaded, and has a smaller operational footprint...

[Rebuttal round]
[Judge's questions about failure modes and operational complexity]
[Pre-judgment disclosure: leaning Redis]
[Final arguments]

## Judgment

**Recommendation:** Redis

**Reasoning:**
Session persistence across restarts outweighs Memcached's
raw throughput advantage at our scale...

**Trade-offs:**
Higher memory overhead, more operational surface area...

**Confidence:** High — persistence requirement is decisive
```

## Constraints

- **Iteration limit:** Maximum 10 rounds of deliberation (steps 4-6)
- **Model:** All advocates use opus for best reasoning
- **Tool access:** Advocates have full tool access for research
- **Good faith:** Advocates must not fabricate facts or misrepresent options

## Philosophy

The adversarial process ensures all options are robustly explored before a decision is made. The judge's job is not to be passive — ask hard questions, challenge weak arguments, push advocates to address concerns. The quality of the judgment depends on the rigor of the process.
