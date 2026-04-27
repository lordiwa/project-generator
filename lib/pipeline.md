---
name: pipeline
description: Core orchestration logic for the project generator pipeline. Defines stages, agent routing, and output assembly.
---

# Project Generator Pipeline

## Pipeline Stages

```
┌────────┐  ┌──────────┐  ┌─────────┐  ┌───────────┐  ┌────────┐  ┌────────────┐
│ Intake │─▶│Knowledge │─▶│ Expert  │─▶│Structure &│─▶│ Output │─▶│ Bootstrap  │
│        │  │  Update  │  │Analysis │  │ Synthesis │  │        │  │  Package   │
└────────┘  └──────────┘  └─────────┘  └───────────┘  └────────┘  └────────────┘
```

### Stage 1: Intake

**Trigger**: User provides project context (docs, conversation, or both)

**Actions**:
1. Collect all input material (documents, conversation history, links)
2. Classify project type: `software` | `research` | `analysis` | `planning`
3. Determine output scale: `light` | `medium` | `full`
4. Extract initial keywords and domain identifiers

**Output**: `docs/intake.md` — structured summary of raw input with classification

### Stage 2: Knowledge Update

**Trigger**: After intake, before expert analysis

**Actions**:
1. Dispatch research agents to gather current domain knowledge
2. Search for latest trends, tools, competitors, regulations
3. Store findings in `knowledge/` directory
4. Create a domain knowledge brief for expert agents

**Output**: `knowledge/{domain}-brief.md` — current domain intelligence

### Stage 3: Expert Analysis

**Trigger**: Intake + knowledge are ready

**Actions** (agents run in parallel where possible):

| Agent | Runs When | Depends On |
|-------|-----------|------------|
| **Research Analyst** | Always | intake, knowledge brief |
| **Business Analyst** | Always | intake, knowledge brief |
| **Product/UX** | software, planning | intake, research findings |
| **Architect** | software | requirements, research findings |
| **Security/Compliance** | software | requirements, architecture |
| **Project Manager** | Always | all other expert outputs |

**Execution Order**:
- **Wave 1** (parallel): Research Analyst + Business Analyst
- **Wave 2** (parallel): Product/UX + Architect (after Wave 1)
- **Wave 3**: Security/Compliance (after architecture)
- **Wave 4**: Project Manager (after all experts)

**Output**: `docs/expert-{agent-name}.md` for each agent

### Stage 4: Structure & Synthesis

**Trigger**: All expert analyses complete

**Actions**:
1. Synthesize expert outputs into unified project definition
2. Resolve conflicts between expert recommendations
3. Apply output template based on scale (light/medium/full)
4. Identify items needing human expert review
5. Generate final deliverables

**Output**: Populated templates in `docs/`

### Stage 5: Output

**Deliverables by scale**:

#### Light (research/analysis)
- `docs/project-brief.md` — Problem statement, key findings, recommendations

#### Medium (planning)
- `docs/project-brief.md` — Problem statement and scope
- `docs/use-cases.md` — Structured use cases
- `docs/requirements.md` — Prioritized requirements
- `docs/roadmap.md` — Phased roadmap

#### Full (software)
- All medium deliverables, plus:
- `docs/architecture.md` — System design and tech stack
- `docs/security.md` — Security assessment and requirements
- `docs/project-plan.md` — Detailed project plan with estimates

### Stage 6: Bootstrap Package (mandatory for all scales)

**Trigger**: After Stage 5 outputs are written.

**Purpose**: Make the project self-deployable. Any IA or human entering `projects/{slug}/` must be able to install dependencies, configure MCPs, register agents/skills, and start working without external context.

**Actions**:
1. Copy `templates/project-bootstrap.md` to `projects/{slug}/README.md`.
2. Replace placeholders with project-specific values:
   - `{PROJECT_NAME}`, `{slug}`, `{PRODUCT_ONE_LINER}`
   - `{TECH_STACK}` (derived from `expert-architect.md`)
   - `{COMPLIANCE_REQUIREMENTS}` (derived from `expert-security-compliance.md`)
   - MCP list (derived from tech stack and skills)
   - `{ENV_VAR_*}` (derived from `expert-architect.md` integrations)
   - `{STACK_CLI}` (firebase, supabase, etc.)
3. Identify operational skills that should live in `projects/{slug}/skills/` based on the expert outputs:
   - Look for clusters of operational responsibilities in `expert-product-ux.md` (user journeys), `expert-business-analyst.md` (operational use cases), and `expert-security-compliance.md` (audit/reporting).
   - Group them by cohesive operational domain — never one-skill-per-role. If you propose >6 skills, you are over-fragmenting; consolidate.
   - Generate `projects/{slug}/skills/{skill}.md` files following the format of existing skills (frontmatter + when to invoke + responsibilities + workflow + integrations + outputs).
   - Generate `projects/{slug}/skills/README.md` with the skills index.
4. Verify the bootstrap by walking the README's checklist (Section 5) end-to-end on a clean checkout.

**Output**:
- `projects/{slug}/README.md` (bootstrap guide, mandatory)
- `projects/{slug}/skills/*.md` (operational skills, project-specific)
- `projects/{slug}/skills/README.md` (skills index)

**Rule**: this stage is **never skipped**, even for `light` scale projects. A research-only project still gets a minimal README pointing to the brief and saying "no skills needed — read-only output."

**Rationale**: skill-centric architecture means projects do not redefine the 6 generic agents in `agents/`. They define skills that group operational responsibilities by domain. The bootstrap README is the entry point that ties agents + skills + MCPs + env into a self-deploying package.

## Human Expert Integration

At any stage, the pipeline can:
- **Tag questions** for human review with `[HUMAN-INPUT-NEEDED]` markers
- **Pause and ask** when AI confidence is low on domain-specific decisions
- **Incorporate feedback** — human responses feed back into the relevant stage

## Pipeline State

Track pipeline progress in `docs/pipeline-state.md`:
```markdown
## Pipeline State
- **Project**: [name]
- **Type**: [software|research|analysis|planning]
- **Scale**: [light|medium|full]
- **Current Stage**: [1-6]
- **Stage Status**:
  - Intake: [pending|complete]
  - Knowledge Update: [pending|in-progress|complete]
  - Expert Analysis: [pending|in-progress|complete]
  - Synthesis: [pending|in-progress|complete]
  - Output: [pending|complete]
  - Bootstrap Package: [pending|complete]   ← README + skills/ + skills/README.md
```

A project is **not considered done** until Stage 6 is complete. The pipeline finishes only when `projects/{slug}/README.md` exists, all placeholders are filled, and the post-bootstrap checklist passes on a clean checkout.
