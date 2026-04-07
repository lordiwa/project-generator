---
name: update-knowledge
description: Dispatches research agents to gather current domain knowledge from the internet, updating the project's knowledge base. Part of the pipeline between intake and expert analysis.
model: opus
tools:
  - Read
  - Write
  - Glob
  - Grep
  - WebSearch
  - WebFetch
  - Agent
  - Bash
---

# Update Knowledge Skill

Research and update the knowledge base for a project domain. This skill dispatches multiple research agents in parallel to gather the latest information from the internet.

## When to Run

- **Pipeline stage 2**: Automatically after intake, before expert analysis
- **On demand**: When domain knowledge needs refreshing
- **Scheduled**: (Future) On a cron schedule to keep knowledge current

## Process

### 1. Determine Research Scope

Read the intake document (`docs/intake.md`) or accept a domain/topic from the user. Extract:
- **Primary domain** (e.g., "e-commerce", "healthcare scheduling", "data pipelines")
- **Key technologies** mentioned or implied
- **Industry/vertical** context
- **Geographic/regulatory** scope

### 2. Dispatch Parallel Research Agents

Launch these research threads simultaneously using the Agent tool:

#### Thread A: Domain & Industry
- Current state of the domain/industry
- Key trends in the last 12 months
- Major players and their approaches
- Relevant standards, regulations, best practices

#### Thread B: Technology Landscape
- Current best-in-class tools and frameworks for the domain
- Emerging technologies relevant to the project
- Common architecture patterns for similar projects
- Open source vs commercial options

#### Thread C: Competitive Intelligence
- Existing solutions that address similar problems
- Their strengths, weaknesses, and pricing models
- User reviews and common complaints
- Market gaps and opportunities

#### Thread D: Risks & Considerations
- Common failure modes for similar projects
- Security and compliance requirements for the domain
- Scalability challenges others have faced
- Lessons learned from public post-mortems or case studies

### 3. Synthesize & Store

After all research threads complete:

1. **Synthesize** findings into a structured knowledge brief
2. **Write** to `knowledge/{domain}-brief.md` with this structure:

```markdown
---
domain: {domain}
updated: {YYYY-MM-DD}
confidence: {high|medium|low}
---

# {Domain} Knowledge Brief

## Executive Summary
[2-3 sentence overview of key findings]

## Domain Context
[Industry state, trends, key players]

## Technology Landscape
[Current tools, frameworks, patterns]

## Competitive Analysis
[Existing solutions, gaps, opportunities]

## Risks & Considerations
[Common pitfalls, compliance, security]

## Key Takeaways for This Project
[Actionable insights that should inform expert analysis]

## Sources
[List of sources consulted with dates]
```

3. **Update** `knowledge/index.md` with a pointer to the new brief
4. **Flag** any areas where information was scarce or conflicting (low confidence)

### 4. Report

Output a brief summary of what was researched and key findings, highlighting:
- High-confidence insights that should drive decisions
- Low-confidence areas needing human expert input
- Surprising findings that challenge initial assumptions

## Usage

When invoked, this skill will:
1. Check for `docs/intake.md` — if present, auto-extract domain context
2. If no intake exists, ask the user for the domain/topic to research
3. Run all 4 research threads in parallel
4. Synthesize and store results
5. Return a summary

## Future: Scheduled Updates

This skill is designed to be triggered by scheduled tasks (cron). When scheduled:
- Compare new findings against existing `knowledge/{domain}-brief.md`
- Only update sections with material changes
- Log changes to `knowledge/changelog.md`
- Alert if significant shifts detected (new competitor, regulation change, etc.)
