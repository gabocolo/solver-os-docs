# Spec Agent Portal — Portal de Administracion del Agente de Specs

Portal web para administrar el agente de generacion de especificaciones del framework AI-Driven Engineering de Tech&Solve. Automatiza el ciclo completo: desde la entrada de negocio hasta la spec versionada y aprobada, con gobernanza, quality gates y metricas integradas.

## Estructura AI-Ready Repository

```
AgenteSpec/
├── architecture/                              ← ADRs, C4, principios, arquitectura de solucion
│   ├── 01-SOLUTION-ARCHITECTURE.md            ← Vision, hexagonal, componentes, seguridad
│   ├── 02-SEQUENCE-FLOWS.md                   ← Diagramas de secuencia (7 flujos)
│   ├── 03-DATA-MODEL.md                       ← ER, tablas, enums, indices, ADR-006
│   ├── 04-DEPLOYMENT-AND-INFRA.md             ← Docker Compose, CI/CD, repo structure
│   ├── 05-UI-WIREFRAMES.md                    ← Wireframes (6 pantallas)
│   ├── 06-ASSUMPTIONS.md                      ← Supuestos, riesgos, checklist
│   ├── c4/                                    ← Diagramas C4 en Mermaid
│   └── principles/                            ← Principios de arquitectura
├── specifications/                            ← Specs AI-executable
│   ├── domain/                                ← Specs L1 (dominio)
│   │   └── SPEC-L1-spec-agent-portal.md       ← Dominio: agregados, reglas, eventos, 7 CUs
│   ├── system/                                ← Specs L2 (sistema)
│   │   ├── SPEC-L2-manage-projects.md         ← Proyectos y gobernanza
│   │   ├── SPEC-L2-create-specs.md            ← Creacion asistida por IA
│   │   ├── SPEC-L2-review-specs.md            ← Revision y aprobacion
│   │   ├── SPEC-L2-generate-prompts.md        ← Prompt builder
│   │   ├── SPEC-L2-metrics-dashboard.md       ← Metricas y trazabilidad
│   │   ├── SPEC-L2-manage-skills.md           ← Catalogo de skills
│   │   └── SPEC-L2-sync-devops.md             ← Sync con Azure DevOps
│   ├── changes/                               ← Specs L3 (cambios incrementales)
│   └── context/                               ← Contexto inicial capturado
├── contracts/                                 ← Punto de convergencia back/front
│   ├── openapi/                               ← Contratos OpenAPI 3.1 YAML (7 contratos)
│   └── events/                                ← Schemas de eventos de dominio (JSON)
│       ├── SpecApproved.json
│       ├── SpecStatusChanged.json
│       └── PromptGenerated.json
├── security/                                  ← Seguridad y DLP
│   ├── threat-models/THREAT-MODEL-SPEC-AGENT.md ← Analisis STRIDE
│   ├── policies/QUALITY-GATES-PROJECT.md      ← Quality gates
│   └── dlp/DLP-PATTERNS.md                    ← Patrones DLP pre/post-prompt
├── tests/                                     ← Pruebas (TDD — tests antes de codigo)
│   ├── unit/                                  ← Tests GWT (48 escenarios)
│   ├── integration/                           ← Tests de integracion
│   ├── contract/                              ← Tests de contrato OpenAPI
│   ├── e2e/                                   ← Tests end-to-end
│   └── fixtures/                              ← Datos sinteticos (NUNCA PII real)
├── src/                                       ← Codigo de producto
│   ├── backend/                               ← .NET 10 Hexagonal (Domain/Application/Infrastructure/Api)
│   └── frontend/                              ← Angular 21 (core/shared/features/auth)
├── observability/                             ← Logs, metricas, trazas, alertas
│   ├── metrics/METRICS-DEFINITIONS.md         ← 20+ metricas custom definidas
│   ├── logs/                                  ← Configuracion Serilog + PII masking
│   ├── tracing/                               ← OpenTelemetry config
│   └── alerts/                                ← Reglas de alertamiento
├── governance/                                ← ADRs del proyecto (decisiones locales)
│   └── adrs/ADR-P001..ADR-P005               ← 5 ADRs del proyecto
├── prompts/                                   ← 15 prompts (7 BE + 7 FE + 1 scaffold)
├── accelerators/                              ← Acelerador transversal + oportunidades
├── CLAUDE-BACKEND.md                          ← Contexto IA backend (.NET 10)
└── CLAUDE-FRONTEND.md                         ← Contexto IA frontend (Angular 21)
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
