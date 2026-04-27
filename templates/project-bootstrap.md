# {PROJECT_NAME} — Bootstrap Guide

> **Para humanos y para IAs**: este README es la guía de auto-despliegue del proyecto. Una IA con Claude Code (o un humano) debe poder seguir estos pasos secuencialmente y dejar el entorno listo para empezar a trabajar sobre {PROJECT_NAME}.
>
> **Esta plantilla** se copia a `projects/{slug}/README.md` al crear cualquier proyecto nuevo. Reemplaza los placeholders `{...}` con los valores reales antes de hacer commit.

---

## 0. Contexto rápido

- **Producto**: {PRODUCT_ONE_LINER}
- **Stack**: {TECH_STACK} (ej. Vue 3 + Firebase, Next.js + Supabase, etc.)
- **Compliance crítico**: {COMPLIANCE_REQUIREMENTS} (ej. LOPDP, GDPR, HIPAA, ninguno)
- **Documentación canónica**: ver `output/project-definition.md` y `output/expert-*.md`.

---

## 1. Estructura del proyecto

```
projects/{slug}/
├── README.md                        ← este archivo (bootstrap)
├── skills/                          ← skills operativos del producto (específicos)
│   ├── README.md                    ← mapa de skills
│   ├── {skill-1}.md
│   ├── {skill-2}.md
│   └── ...
└── output/                          ← artefactos del pipeline
    ├── intake.md
    ├── {domain}-brief.md
    ├── expert-business-analyst.md
    ├── expert-research-analyst.md
    ├── expert-product-ux.md
    ├── expert-architect.md
    ├── expert-security-compliance.md
    ├── expert-project-manager.md
    ├── project-definition.md
    └── pipeline-state.md
```

**Importante**: ningún proyecto contiene agentes locales. Los 6 agentes genéricos (business-analyst, architect, product-ux, project-manager, research-analyst, security-compliance) viven en la raíz del repo (`agents/`) y son compartidos. Lo único específico del producto son los **skills** en `skills/`.

---

## 2. Pre-requisitos

| Herramienta | Cómo verificar | Si falta |
|---|---|---|
| Claude Code CLI | `claude --version` | Instalar: https://claude.com/claude-code |
| Node.js ≥ 20 | `node --version` | nodejs.org o nvm |
| Git | `git --version` | git-scm.com |
| {STACK_CLI} (ej. firebase, supabase) | `{cli} --version` | `npm install -g {cli}` |

---

## 3. Despliegue paso a paso

### 3.1 — Instalar plugins de Claude Code

```bash
# GSD (Get Shit Done) workflow
/plugin install gsd
/gsd-help    # verificar
```

Si GSD ya está global, saltar.

### 3.2 — Configurar MCP servers

Editar `.claude/settings.json` en la raíz del repo:

```json
{
  "mcpServers": {
    "context7": {
      "command": "npx",
      "args": ["-y", "@upstash/context7-mcp@latest"]
    }
    // Añadir MCPs específicos del stack:
    // "firebase": {...}, "supabase": {...}, "playwright": {...}, etc.
  }
}
```

**MCPs recomendados por dominio** (elegir según proyecto):

| Necesidad | MCP |
|---|---|
| Docs actualizadas de librerías | `context7` |
| Firebase (Firestore, Functions, Auth) | `firebase` (firebase-tools experimental:mcp) |
| Supabase | `supabase-mcp` |
| Web scraping / SEO research | `firecrawl` |
| Búsquedas de código en repos remotos | `exa` |
| Browser automation / E2E | `playwright` |
| GitHub PRs/issues | `github` |

Verificar después: `claude mcp list`.

### 3.3 — Registrar agentes y skills en Claude Code

Claude Code descubre agentes en `.claude/agents/` y skills en `.claude/skills/`.

**Bash**:
```bash
mkdir -p .claude/agents .claude/skills

# 6 agentes genéricos del repo
for f in agents/*.md; do
  cp "$f" ".claude/agents/$(basename $f)"
done

# Skills globales del pipeline
cp skills/*.md .claude/skills/

# Skills específicos del proyecto
for f in projects/{slug}/skills/*.md; do
  [ "$(basename $f)" = "README.md" ] && continue
  cp "$f" ".claude/skills/$(basename $f)"
done
```

**PowerShell (Windows)**:
```powershell
New-Item -ItemType Directory -Force -Path .claude\agents, .claude\skills
Get-ChildItem agents\*.md | Copy-Item -Destination .claude\agents\
Get-ChildItem skills\*.md | Copy-Item -Destination .claude\skills\
Get-ChildItem projects\{slug}\skills\*.md -Exclude README.md | Copy-Item -Destination .claude\skills\
```

Reiniciar Claude Code. Los skills se invocan vía `/{skill-name}` y los agentes vía `Agent` tool.

### 3.4 — Inicializar GSD

```bash
/gsd-new-project
```

Apuntar al output existente:
- `projects/{slug}/output/intake.md`
- `projects/{slug}/output/project-definition.md`
- `projects/{slug}/output/expert-*.md`

### 3.5 — Variables de entorno

Crear `.env.local` (añadir a `.gitignore`):

```env
# {STACK} credentials
{ENV_VAR_1}=
{ENV_VAR_2}=

# Third-party services (rellenar según el proyecto)
```

---

## 4. Cómo trabajar después del bootstrap

### Tareas operativas del producto

Usar los slash commands de los skills definidos en `projects/{slug}/skills/`. Listar con `ls projects/{slug}/skills/`.

### Tareas de planning/análisis

Usar los 6 agentes genéricos:

| Tarea | Agente |
|---|---|
| Requisitos, casos de uso | `agents/business-analyst.md` |
| Arquitectura, tech stack | `agents/architect.md` |
| UX, journeys, features | `agents/product-ux.md` |
| Threat model, compliance | `agents/security-compliance.md` |
| Market research | `agents/research-analyst.md` |
| Roadmap, riesgos | `agents/project-manager.md` |

### Implementación con GSD

```bash
/gsd-progress       # ver estado
/gsd-next           # avanzar al siguiente paso
/gsd-discuss-phase  # discutir antes de planear
/gsd-plan-phase     # crear PLAN.md
/gsd-execute-phase  # ejecutar con commits atómicos
```

---

## 5. Checklist post-bootstrap

- [ ] `claude --version` OK
- [ ] `/gsd-help` lista comandos
- [ ] `claude mcp list` muestra los MCPs conectados
- [ ] `.claude/agents/` tiene los 6 agentes genéricos
- [ ] `.claude/skills/` tiene los skills (globales + del proyecto)
- [ ] `/{algun-skill-del-proyecto}` aparece invocable
- [ ] `.env.local` tiene las credenciales mínimas
- [ ] CLI del stack autenticado (`firebase login`, `supabase login`, etc.)

---

## 6. Notas de convención (válidas para todos los proyectos)

- **No duplicar agentes**: los 6 genéricos viven en `agents/` raíz, no se copian a `projects/`.
- **Skills específicos en `projects/{slug}/skills/`**, no en `skills/` raíz.
- **Outputs en `output/` son artefactos congelados** — no reescribir a menos que sea por refactor explícito.
- **Documentar el patrón skill-centric**: agrupar por dominio operativo, no por rol granular. Si el proyecto tiene >6 agentes específicos, casi seguro hay solapamientos — fusionar en skills.
- **Lectura prioritaria** al arrancar:
  1. `output/project-definition.md` (síntesis)
  2. `skills/README.md` (mapa de skills)
  3. `output/expert-*.md` (profundidad por dominio)
- **Tech stack del usuario** (preferencia documentada): Vue 3 + Firebase. Validar contra el `expert-architect.md` del proyecto antes de proponer alternativas.
