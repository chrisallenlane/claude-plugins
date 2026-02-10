# /scope - Problem Space Exploration

Exploratory workflow for understanding problem spaces before implementation. Creates detailed tickets in your issue tracker after thorough codebase exploration and iterative dialogue.

## Purpose

When you need to:
- Plan a feature thoroughly before implementation
- Investigate a complex bug and document findings
- Explore refactoring opportunities in detail
- Understand integration points with third-party systems
- Create well-specified tickets for yourself or others to implement

**Key principle:** `/scope` explores and documents. It does NOT implement.

## Usage

```bash
/scope
```

Then describe what you want to explore:
- "I'm thinking about adding a caching layer to the API"
- "There's a performance issue in the checkout flow I want to investigate"
- "I want to refactor the authentication system to use JWTs"
- "How should we add multi-tenancy to this application?"

## What Happens

1. **Discovery**: Claude asks questions to understand what you want to explore
2. **Exploration**: Deep dive into codebase, third-party APIs, existing patterns
3. **Refinement**: Iterative dialogue to clarify approach, edge cases, trade-offs
4. **Synthesis**: Draft ticket content adapted to the type of work (feature/bug/refactor)
5. **Review**: You review and refine the ticket
6. **Creation**: Claude creates the ticket using available integration

## Example Session

```
User: /scope

User: I want to add rate limiting to our API

Claude: Let me explore the API implementation to understand the current structure...
[Explores codebase]

Claude: I found that you're using Express with custom middleware. I see several endpoints but no current rate limiting. Questions:
1. Should rate limiting be per-IP, per-user, or both?
2. What limits are you thinking? (requests per minute/hour)
3. Should different endpoints have different limits?
4. How should we handle limit exceeded - 429 status code?

User: Per-user authentication, 100 req/hour, same limits for all endpoints, yes 429

Claude: Got it. I also noticed you have Redis available from the caching layer. Should we use Redis for rate limit tracking (distributed), or in-memory (simpler but won't work with multiple servers)?

User: We're planning to scale horizontally soon, so Redis

Claude: Perfect. Here's the ticket I've drafted...
[Shows ticket with problem statement, technical approach, acceptance criteria, etc.]

User: Looks good

Claude: Creating ticket...
✓ Created: https://git.example.com/yourorg/yourrepo/issues/42
```

## Ticket Quality

Tickets created by `/scope` are comprehensive and include:

**Feature proposals:**
- Problem statement
- Proposed solution approach
- Specific acceptance criteria
- Technical implementation notes
- Explicitly out-of-scope items
- Open questions (if any)

**Refactorings:**
- Current vs desired state
- Motivation and benefits
- High-level strategy
- Risk assessment
- Affected components

**Bug fixes:**
- Reproduction and symptoms
- Root cause analysis
- Proposed fix approach
- Prevention strategy
- Verification plan

## Integration with Other Skills

`/scope`, `/iterate`, and `/refactor` are complementary:

- **Use /scope first** when you need to think through a complex change
- **Ticket gets created** with all the exploration and decisions documented
- **Use /iterate later** to actually implement the ticket (or assign to someone else)
- **Use /refactor** for autonomous cleanup of existing code (technical debt, code quality)

You can also use `/iterate` directly for straightforward changes that don't need extensive exploration, or `/refactor` independently when the codebase needs tidying.

## Tips

**Ask lots of questions**: Claude will challenge assumptions and propose alternatives. Engage with this.

**Be specific about uncertainty**: If you're unsure about an approach, say so. The exploration will help clarify.

**Don't skip exploration**: Even if you think you know the solution, let Claude explore. It often finds edge cases or existing patterns you missed.

**Ticket quality matters**: A well-scoped ticket prevents implementation surprises and rework.

## Requirements

- `git` repository (for issue tracker detection)
- Issue tracker integration (optional - CLI, MCP server, or API)
- If not available, Claude will output ticket content for manual creation

## Differences from Normal Conversation

In normal conversation, you might ask "How should I implement X?" and Claude would explore and then help you implement.

With `/scope`, Claude explores and creates a ticket, but **stops before implementation**. This is useful when:
- You want a formal record of the exploration (the ticket)
- You're planning to implement later
- Someone else will implement
- You want to think through multiple changes before starting any
