# Project Generator

A Claude Code-based system that intakes project context, runs it through specialized expert agents, and produces structured project definitions — from lightweight research briefs to full software project plans.

## What This Is

An orchestration layer that receives data from any project (docs, conversation) and uses expert AI agents + human input to structure, plan, analyze, research, and optionally develop projects.

## Directory Structure

```
project-generator/
├── agents/              # Expert agent definitions (subagents)
│   ├── business-analyst.md
│   ├── architect.md
│   ├── research-analyst.md
│   ├── product-ux.md
│   ├── security-compliance.md
│   └── project-manager.md
├── skills/              # Reusable skills (slash commands)
│   └── update-knowledge.md
├── templates/           # Output document templates
│   ├── project-brief.md
│   ├── use-cases.md
│   ├── requirements.md
│   ├── roadmap.md
│   └── architecture.md
├── lib/                 # Core logic and pipeline definition
│   └── pipeline.md
├── knowledge/           # Domain knowledge base (generated)
│   └── index.md
├── projectMaterial/     # Raw project input (docs, specs, notes added by user)
└── docs/                # Project output (generated per-project)
```

## Pipeline

```
Intake → Knowledge Update → Expert Analysis → Synthesis → Output
```

1. **Intake**: Receive project context (docs/conversation), classify type and scale
2. **Knowledge Update**: Research agents gather current domain intelligence
3. **Expert Analysis**: Specialized agents analyze in parallel waves
4. **Synthesis**: Merge expert outputs, resolve conflicts, apply templates
5. **Output**: Deliver structured project definition

## Expert Agents

| Agent | File | Specialty |
|-------|------|-----------|
| Business Analyst | `agents/business-analyst.md` | Requirements, use cases, stakeholders |
| Architect | `agents/architect.md` | System design, tech stack, infrastructure |
| Research Analyst | `agents/research-analyst.md` | Market research, competitive analysis |
| Product/UX | `agents/product-ux.md` | User experience, feature prioritization |
| Security/Compliance | `agents/security-compliance.md` | Threat modeling, compliance |
| Project Manager | `agents/project-manager.md` | Roadmap, estimation, risk assessment |

## Output Scales

- **Light**: Project brief with findings and recommendations (research/analysis projects)
- **Medium**: Brief + use cases + requirements + roadmap (planning projects)
- **Full**: All medium + architecture + security + detailed project plan (software projects)

## Skills

- **update-knowledge**: Dispatches parallel research agents to gather current domain knowledge. Stores results in `knowledge/`. Designed for future scheduled execution.

## Conventions

- Agent outputs go to `docs/expert-{agent-name}.md` (or `projects/{slug}/output/expert-{agent-name}.md` for per-project pipelines)
- Knowledge briefs go to `knowledge/{domain}-brief.md`
- Pipeline state tracked in `docs/pipeline-state.md`
- Human input needed is flagged with `[HUMAN-INPUT-NEEDED]`
- All templates use `{placeholder}` syntax for variable substitution
- **Skill-centric architecture for product runtime** — projects do NOT define local agents; they define `skills/` that group operational responsibilities by domain. The 6 generic agents in `agents/` are shared across all projects.
- **Every project must include a bootstrap README** at `projects/{slug}/README.md` so any IA or human can self-deploy the environment (install GSD, configure MCPs, register agents/skills, set env vars). Use `templates/project-bootstrap.md` as the starting point.

## Per-Project Structure

```
projects/{slug}/
├── README.md                ← bootstrap guide (mandatory)
├── skills/                  ← product-specific operational skills
│   ├── README.md            ← skills index + migration map
│   └── {skill-name}.md
└── output/                  ← pipeline artifacts (intake, briefs, expert-*, project-definition)
```

When creating a new project, copy `templates/project-bootstrap.md` to `projects/{slug}/README.md` and fill in the placeholders.
