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
├── prompts/
│   ├── README.md                              ← Indice y orden de implementacion
│   ├── PROMPT-01-manage-projects.md           ← Prompt: Governance Service
│   ├── PROMPT-02-create-specs.md              ← Prompt: Spec Management + Agent
│   ├── PROMPT-03-review-specs.md              ← Prompt: Review Service
│   ├── PROMPT-04-generate-prompts.md          ← Prompt: Prompt Builder
│   ├── PROMPT-05-metrics-dashboard.md         ← Prompt: Metrics Service
│   ├── PROMPT-06-manage-skills.md             ← Prompt: Skill Catalog
│   └── PROMPT-07-sync-devops.md               ← Prompt: DevOps Sync Service
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
│     └── PROMPT-01 → SK-001
├── L2: CreateSpecs v1.0.0 (creacion asistida por IA)
│     ├── 8 dependencias, 8 escenarios GWT
│     └── PROMPT-02 → SK-001
├── L2: ReviewSpecs v1.0.0 (revision + validacion auto)
│     ├── 7 dependencias, 8 escenarios GWT
│     └── PROMPT-03 → SK-001
├── L2: GeneratePrompts v1.0.0 (prompt builder)
│     ├── 6 dependencias, 6 escenarios GWT
│     └── PROMPT-04 → SK-001
├── L2: MetricsDashboard v1.0.0 (metricas + trazabilidad)
│     ├── 6 dependencias, 6 escenarios GWT
│     └── PROMPT-05 → SK-001
├── L2: ManageSkills v1.0.0 (catalogo de skills)
│     ├── 5 dependencias, 6 escenarios GWT
│     └── PROMPT-06 → SK-001
└── L2: SyncDevOps v1.0.0 (sync Azure DevOps)
      ├── 6 dependencias, 8 escenarios GWT
      └── PROMPT-07 → SK-001
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

## Orden recomendado de implementacion

```
1. PROMPT-01 → Governance Service (base del sistema)
2. PROMPT-02 → Spec Management + Agent (funcionalidad core)
3. PROMPT-03 → Review Service (flujo de aprobacion)
4. PROMPT-04 → Prompt Builder
5. PROMPT-06 → Skill Catalog
6. PROMPT-05 → Metrics Dashboard
7. PROMPT-07 → DevOps Sync (integracion externa)
```

Cada prompt genera un PR < 400 lineas. Se recomienda subdividir los prompts mas grandes (02, 03, 07) en multiples PRs segun las indicaciones de cada prompt.
