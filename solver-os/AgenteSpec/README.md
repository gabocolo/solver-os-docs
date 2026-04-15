# Spec Agent Portal — Portal de Administracion del Agente de Specs

Portal web para administrar el agente de generacion de especificaciones del framework AI-Driven Engineering de Tech&Solve. Automatiza el ciclo completo: desde la entrada de negocio hasta la spec versionada y aprobada, con gobernanza, quality gates y metricas integradas.

## Estructura

```
AgenteSpec/
├── accelerators/
│   ├── ACCELERATOR-SPEC-AGENT-PORTAL.md       ← Diseno del acelerador transversal
│   └── AGENTIZATION-OPPORTUNITIES.md          ← Matriz de oportunidades + ROI
├── architecture/
│   ├── README.md                              ← Indice de arquitectura y stack
│   ├── 01-SOLUTION-ARCHITECTURE.md            ← Vision, hexagonal, componentes, seguridad
│   ├── 02-SEQUENCE-FLOWS.md                   ← Diagramas de secuencia (7 flujos)
│   ├── 03-DATA-MODEL.md                       ← ER, tablas, enums, indices, ADR-006
│   ├── 04-DEPLOYMENT-AND-INFRA.md             ← Docker Compose, CI/CD, repo structure
│   ├── 05-UI-WIREFRAMES.md                    ← Wireframes ASCII (6 pantallas)
│   └── 06-ASSUMPTIONS.md                      ← Supuestos, riesgos, checklist
├── governance/
│   ├── README.md                              ← Indice de gobernanza del proyecto
│   ├── adrs/
│   │   ├── ADR-P001-llm-model-selection.md    ← Seleccion de modelos por tarea
│   │   ├── ADR-P002-async-generation.md       ← Generacion asincrona via Service Bus
│   │   ├── ADR-P003-spec-storage-jsonb.md     ← Almacenamiento JSONB en PostgreSQL
│   │   ├── ADR-P004-devops-sync-strategy.md   ← Sincronizacion con Azure DevOps
│   │   └── ADR-P005-dlp-filter-architecture.md ← Arquitectura DLP bidireccional
│   └── quality-gates/
│       └── QUALITY-GATES-PROJECT.md           ← Quality gates del proyecto (Gate 1-3)
├── CLAUDE-BACKEND.md                              ← Contexto IA para repo backend (.NET 10)
├── CLAUDE-FRONTEND.md                             ← Contexto IA para repo frontend (Angular 21)
├── prompts/
│   ├── README.md                              ← Indice y orden de implementacion BE+FE
│   ├── PROMPT-00-FE-ARCHITECTURE.md           ← Prompt FE: Scaffold Angular 21
│   ├── PROMPT-01-manage-projects.md           ← Prompt BE: Governance Service
│   ├── PROMPT-01-FE-projects.md               ← Prompt FE: Projects module
│   ├── PROMPT-02-create-specs.md              ← Prompt BE: Spec Management + Agent
│   ├── PROMPT-02-FE-create-specs.md           ← Prompt FE: Spec Editor + AI wizard
│   ├── PROMPT-03-review-specs.md              ← Prompt BE: Review Service
│   ├── PROMPT-03-FE-review-specs.md           ← Prompt FE: Review Panel
│   ├── PROMPT-04-generate-prompts.md          ← Prompt BE: Prompt Builder
│   ├── PROMPT-04-FE-generate-prompts.md       ← Prompt FE: Prompt Builder UI
│   ├── PROMPT-05-metrics-dashboard.md         ← Prompt BE: Metrics Service
│   ├── PROMPT-05-FE-metrics-dashboard.md      ← Prompt FE: Metrics Dashboard
│   ├── PROMPT-06-manage-skills.md             ← Prompt BE: Skill Catalog
│   ├── PROMPT-06-FE-manage-skills.md          ← Prompt FE: Skill Catalog UI
│   └── PROMPT-07-sync-devops.md               ← Prompt BE: DevOps Sync Service
└── specs/
    ├── SPEC-L1-spec-agent-portal.md           ← Dominio: entidades, reglas, 7 CUs
    ├── SPEC-L2-manage-projects.md             ← Sistema: proyectos y gobernanza
    ├── SPEC-L2-create-specs.md                ← Sistema: creacion asistida por IA
    ├── SPEC-L2-review-specs.md                ← Sistema: revision y aprobacion
    ├── SPEC-L2-generate-prompts.md            ← Sistema: prompt builder
    ├── SPEC-L2-metrics-dashboard.md           ← Sistema: metricas y trazabilidad
    ├── SPEC-L2-manage-skills.md               ← Sistema: catalogo de skills
    └── SPEC-L2-sync-devops.md                 ← Sistema: sync con Azure DevOps
```

## Trazabilidad

```
L1: Spec Agent Portal (dominio)
├── L2: ManageProjects v1.0.0 (proyectos + gobernanza)
│     ├── 6 dependencias, 6 escenarios GWT
│     ├── PROMPT-01 (BE) → SK-001
│     └── PROMPT-01-FE (FE) → Angular Projects module
├── L2: CreateSpecs v1.0.0 (creacion asistida por IA)
│     ├── 8 dependencias, 8 escenarios GWT
│     ├── PROMPT-02 (BE) → SK-001
│     └── PROMPT-02-FE (FE) → Angular Spec Editor + AI wizard
├── L2: ReviewSpecs v1.0.0 (revision + validacion auto)
│     ├── 7 dependencias, 8 escenarios GWT
│     ├── PROMPT-03 (BE) → SK-001
│     └── PROMPT-03-FE (FE) → Angular Review Panel
├── L2: GeneratePrompts v1.0.0 (prompt builder)
│     ├── 6 dependencias, 6 escenarios GWT
│     ├── PROMPT-04 (BE) → SK-001
│     └── PROMPT-04-FE (FE) → Angular Prompt Builder UI
├── L2: MetricsDashboard v1.0.0 (metricas + trazabilidad)
│     ├── 6 dependencias, 6 escenarios GWT
│     ├── PROMPT-05 (BE) → SK-001
│     └── PROMPT-05-FE (FE) → Angular Metrics Dashboard
├── L2: ManageSkills v1.0.0 (catalogo de skills)
│     ├── 5 dependencias, 6 escenarios GWT
│     ├── PROMPT-06 (BE) → SK-001
│     └── PROMPT-06-FE (FE) → Angular Skill Catalog UI
└── L2: SyncDevOps v1.0.0 (sync Azure DevOps)
      ├── 6 dependencias, 8 escenarios GWT
      └── PROMPT-07 (BE) → SK-001 (sin UI)
```

## Gobernanza del proyecto

### ADRs del proyecto (decisiones locales)
| ADR | Decision | Derivado de |
|-----|----------|-------------|
| ADR-P001 | Opus para L1, Sonnet para L2/L3, Haiku para validacion | Framework Seccion 11 |
| ADR-P002 | Generacion asincrona via Azure Service Bus | ADR-002 |
| ADR-P003 | Specs en PostgreSQL JSONB + metadata relacional | ADR-001 |
| ADR-P004 | Sync event-driven con Azure DevOps (no bloquea aprobacion) | ADR-003 |
| ADR-P005 | DLP bidireccional regex-based configurable | ADR-006 |

### Quality Gates
| Gate | Que valida | Bloqueante |
|------|-----------|------------|
| Gate 1 | Spec: estructura + clasificacion + ADRs + revision humana | Si |
| Gate 2 | Codigo: tests + SAST + SCA + DLP scan | Si |
| Gate 3 | PR: revision senior + proteccion datos | Si |

## Stack tecnologico

| Capa | Tecnologia |
|------|-----------|
| Frontend | Angular 21 + TypeScript + Angular Material + Tailwind |
| Backend | .NET 10 + C# + Minimal APIs + Semantic Kernel |
| Database | PostgreSQL 16 + Redis 7 |
| Messaging | Azure Service Bus |
| Work Items | Azure DevOps REST API v7.1 |
| Auth | OAuth 2.0 (GitHub / Azure AD) |
| LLM | Claude API (Opus / Sonnet / Haiku) |
| Observabilidad | OpenTelemetry + Seq |

## Contexto por repositorio

Cada repositorio tiene su propio archivo de contexto para la IA (recomendacion de Osvaldo Arias):

| Archivo | Repositorio | Stack |
|---------|------------|-------|
| [CLAUDE-BACKEND.md](CLAUDE-BACKEND.md) | Backend | .NET 10, C#, Hexagonal, PostgreSQL, Redis |
| [CLAUDE-FRONTEND.md](CLAUDE-FRONTEND.md) | Frontend | Angular 21, Material, Tailwind, Monaco Editor |

> La documentacion y el contexto viven en el repositorio de codigo, NO en la wiki. La wiki sirve para humanos, el repo para la IA.

## Contratos OpenAPI

Los contratos OpenAPI son el **punto de convergencia** entre backend y frontend. Se generan con SK-003 desde la spec L2 y deben existir en ambos repositorios.

## Orden recomendado de implementacion

Backend y frontend se ejecutan **en paralelo** por caso de uso:

```
Paso 0: PROMPT-00-FE (scaffold Angular 21 — una sola vez)

Paso 1: PROMPT-01 (BE) + PROMPT-01-FE (FE) — en paralelo
Paso 2: PROMPT-02 (BE) + PROMPT-02-FE (FE) — en paralelo
Paso 3: PROMPT-03 (BE) + PROMPT-03-FE (FE) — en paralelo
Paso 4: PROMPT-04 (BE) + PROMPT-04-FE (FE) — en paralelo
Paso 5: PROMPT-06 (BE) + PROMPT-06-FE (FE) — en paralelo
Paso 6: PROMPT-05 (BE) + PROMPT-05-FE (FE) — en paralelo
Paso 7: PROMPT-07 (BE solo — sin UI)
```

Cada prompt genera un PR < 400 lineas. Se recomienda subdividir los prompts mas grandes (02, 03, 07) en multiples PRs segun las indicaciones de cada prompt.

## Conceptos DDD aplicados

El dominio del Spec Agent Portal aplica Domain-Driven Design de forma pragmatica:

| Concepto DDD | Donde se define | Ejemplo |
|-------------|----------------|---------|
| Bounded Context | Spec L1 — Limite del contexto | Spec Agent Portal: gestion de specs y gobernanza |
| Agregados | Spec L1 — Modelo de dominio | Specification (raiz), ReviewCycle (raiz) |
| Lenguaje ubicuo | Spec L1 — Glosario con "Nombre en codigo" | Spec, Skill, QualityGate, ADR |
| Eventos de dominio | Spec L1 → Spec L2 (contrato tecnico) | SpecApproved, SpecStatusChanged |
| Puertos (ACL) | Spec L2 — Dependencias permitidas | ILLMProvider, IDevOpsWorkItemAdapter |
