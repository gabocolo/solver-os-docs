# 01 — Arquitectura de Solucion

| Campo | Valor |
|-------|-------|
| **Proyecto** | Spec Agent Portal |
| **Version** | 1.0.0 |
| **Fecha** | 2026-04-09 |
| **Estado** | DRAFT |

---

## Vision general

El Spec Agent Portal es una aplicacion web que operacionaliza el framework AI-Driven Engineering de Tech&Solve. Automatiza el ciclo completo de especificaciones: desde la entrada de negocio en lenguaje natural hasta la spec versionada, aprobada y sincronizada con Azure DevOps.

---

## Arquitectura hexagonal (Ports & Adapters)

Segun ADR-001, el sistema sigue arquitectura hexagonal estricta. El dominio no depende de infraestructura.

```
                    ┌─────────────────────────────────────────────────┐
                    │                   FRONTEND                      │
                    │          Angular 21 + Angular Material           │
                    │     Monaco Editor  |  Tailwind CSS  |  SSR      │
                    └──────────────────────┬──────────────────────────┘
                                           │ HTTP / WebSocket
                    ┌──────────────────────▼──────────────────────────┐
                    │              API GATEWAY / CONTROLLERS           │
                    │         .NET 10 Minimal APIs + Auth Middleware   │
                    │         CORS | Rate Limiting | JWT Validation    │
                    └──────────────────────┬──────────────────────────┘
                                           │
          ┌────────────────────────────────▼────────────────────────────────┐
          │                      APPLICATION LAYER                          │
          │                                                                 │
          │  ┌──────────────┐ ┌──────────────┐ ┌──────────────────────┐    │
          │  │ Spec Service │ │ Governance   │ │ Agent Orchestration  │    │
          │  │              │ │ Service      │ │ Service              │    │
          │  └──────┬───────┘ └──────┬───────┘ └──────────┬───────────┘    │
          │         │                │                     │                │
          │  ┌──────┴───────┐ ┌──────┴───────┐ ┌──────────┴───────────┐    │
          │  │ Review       │ │ Metrics      │ │ DevOps Sync          │    │
          │  │ Service      │ │ Service      │ │ Service              │    │
          │  └──────┬───────┘ └──────┬───────┘ └──────────┬───────────┘    │
          │         │                │                     │                │
          └─────────┼────────────────┼─────────────────────┼────────────────┘
                    │                │                     │
          ┌─────────▼────────────────▼─────────────────────▼────────────────┐
          │                       DOMAIN LAYER                              │
          │                                                                 │
          │  Entities: Project, Specification, ADR, ReviewCycle,            │
          │            Prompt, Skill, Metric, QualityGateCheck,            │
          │            AuditLog, WorkItemSync                              │
          │                                                                 │
          │  Ports (interfaces):                                            │
          │    ISpecRepository, IADRRepository, IReviewCycleRepository,    │
          │    IProjectRepository, IPromptRepository, ISkillRepository,    │
          │    ILLMProvider, IDLPFilter, INotificationService,             │
          │    IEventPublisher, IAuditLogRepository, ICachePort,           │
          │    IDevOpsWorkItemAdapter, IWorkItemSyncRepository             │
          │                                                                 │
          └─────────┬────────────────┬─────────────────────┬────────────────┘
                    │                │                     │
          ┌─────────▼────────┐ ┌─────▼──────────┐ ┌───────▼────────────────┐
          │  INFRASTRUCTURE  │ │   AI ADAPTERS  │ │  INTEGRATION ADAPTERS  │
          │                  │ │                │ │                        │
          │  PostgreSQL 16   │ │  Claude API    │ │  Azure DevOps API v7.1 │
          │  (EF Core +      │ │  (Semantic     │ │  Azure Service Bus     │
          │   JSONB specs)   │ │   Kernel)      │ │  OAuth Provider        │
          │  Redis 7 (cache) │ │  DLP Filter    │ │  Notification (email)  │
          │  Seq (logs)      │ │                │ │                        │
          └──────────────────┘ └────────────────┘ └────────────────────────┘
```

---

## Servicios de dominio

### 1. Spec Management Service
- CRUD de especificaciones (L1, L2, L3)
- Versionado semver automatico
- Jerarquia parent-child (L1 → L2 → L3)
- Validacion de estructura por nivel

### 2. Governance Service
- Gestion de proyectos y ADRs
- Configuracion de quality gates
- Validacion automatica de specs (SK-004)
- Generacion asistida de ADRs

### 3. Agent Orchestration Service
- Generacion de specs con IA (Opus para L1, Sonnet para L2/L3)
- Prompt builder (construccion local, sin LLM)
- DLP pre-prompt y post-response
- Cola asincrona via Azure Service Bus

### 4. Review Service
- Ciclo de revision: DRAFT → IN_REVIEW → APPROVED
- Validacion automatica pre-revision (SK-004)
- Comentarios por seccion
- Firma digital de aprobacion

### 5. Metrics Service
- Calculo de metricas agregadas (specs por estado, tiempos, cobertura)
- Cache en Redis (TTL 5 min)
- Arbol de trazabilidad L1 → L2 → L3
- Exportacion CSV/JSON
- Metricas de skills (uso, retrabajo, alertas)

### 6. DevOps Sync Service
- Sincronizacion automatica spec → work item
- Mapeo: L1 → Feature, L2 → User Story, L3 → Task
- Relaciones parent-child en Azure DevOps
- Tolerante a fallos con reintentos

---

## Comunicacion entre servicios

| Flujo | Tipo | Mecanismo |
|-------|------|-----------|
| Frontend → Backend | Sincrono | HTTP REST + WebSocket (notificaciones) |
| Generacion IA | Asincrono | Azure Service Bus (job queue) |
| Spec aprobada → DevOps | Asincrono | Evento SpecApproved → Service Bus |
| Invalidacion de cache | Asincrono | Evento SpecStatusChanged → Redis invalidation |
| Notificaciones | Asincrono | Service Bus → NotificationService |

---

## Seguridad

### Autenticacion y autorizacion
- OAuth 2.0 con GitHub o Azure AD
- JWT tokens con claims de rol
- RBAC con 5 roles: ARCHITECT, LEAD, SENIOR_DEV, QA, PO

### Matriz de permisos

| Accion | ARCHITECT | LEAD | SENIOR_DEV | QA | PO |
|--------|-----------|------|------------|----|----|
| Crear proyecto | X | | | | |
| Gestionar ADRs | X | X | | | |
| Crear specs | X | X | X | | |
| Aprobar specs | X | X | | | |
| Generar prompts | X | X | X | | |
| Ver metricas | X | X | X | X | X |
| Gestionar skills | X | | | | |
| Configurar DevOps | X | | | | |

### Proteccion de datos (ADR-006)
- DLP pre-prompt: sanitiza input antes de enviar al LLM
- DLP post-response: valida que respuesta no contenga PII
- Masking de PII en logs (OpenTelemetry + filtro)
- PAT de Azure DevOps cifrado en vault
- Datos sinteticos en tests y fixtures

### Seguridad de API
- Rate limiting por usuario y endpoint
- CORS configurado para dominio del portal
- Input validation en todos los endpoints
- No exponer IDs internos en mensajes de error

---

## Observabilidad (3 pilares)

### Logs
- Formato: JSON estructurado
- Destino: Seq (o ELK)
- Filtro PII obligatorio (ADR-006)
- CorrelationId en cada request (ADR-003)

### Trazas
- OpenTelemetry SDK
- Propagacion de contexto entre servicios
- Spans para: LLM calls, DB queries, DevOps API calls

### Metricas
- Metricas de negocio: `spec_created_total`, `spec_approved_total`, `prompt_generated_total`
- Metricas de IA: `llm_generation_duration_ms`, `llm_tokens_consumed_total`
- Metricas de infra: `http_request_duration_ms`, `db_query_duration_ms`
- Metricas de DLP: `pii_filtered_total`, `dlp_violations_total`
- Metricas de DevOps sync: `devops_sync_total`, `devops_sync_failed_total`

---

## Decisiones clave

| Decision | Justificacion |
|----------|--------------|
| PostgreSQL + JSONB para specs | Flexibilidad para contenido variable por nivel + queries SQL para metadata |
| Semantic Kernel para LLM | Abstraccion oficial de Microsoft, multi-provider, integrado con .NET |
| Azure Service Bus para async | Confiabilidad enterprise, dead-letter, reintentos nativos |
| Monaco Editor en frontend | Mismo editor de VS Code, soporte markdown nativo, diff viewer |
| Redis para cache de metricas | Sub-milisegundo, TTL nativo, invalidacion por eventos |
| Seq para logs | Query potente sobre JSON, dashboards, alertas, bajo costo MVP |

---

## Contratos OpenAPI

Los contratos OpenAPI son el **punto de convergencia** entre backend y frontend. Se generan con SK-003 desde la seccion "Contrato tecnico" de cada spec L2.

| Contrato | Spec L2 | Endpoints principales |
|----------|---------|----------------------|
| spec-management-api.yaml | SPEC-L2-create-specs | POST /api/specs, GET /api/specs/{id}, PUT /api/specs/{id}/sections |
| governance-api.yaml | SPEC-L2-manage-projects | POST /api/projects, POST /api/adrs, PUT /api/quality-gates |
| review-api.yaml | SPEC-L2-review-specs | POST /api/reviews, PUT /api/reviews/{id}/decision |
| prompt-builder-api.yaml | SPEC-L2-generate-prompts | POST /api/prompts/build, GET /api/prompts/{id} |
| metrics-api.yaml | SPEC-L2-metrics-dashboard | GET /api/metrics, GET /api/metrics/traceability |
| skills-api.yaml | SPEC-L2-manage-skills | POST /api/skills, PUT /api/skills/{id} |
| devops-sync-api.yaml | SPEC-L2-sync-devops | POST /api/sync/spec/{id} |

> Cada contrato se genera usando OPENAPI-CONTRACT-TEMPLATE.yaml y SK-003. El contrato debe existir en ambos repositorios (backend y frontend).

---

## Contexto por repositorio

Cada repositorio tiene su propio archivo de contexto para la IA:

| Archivo | Repositorio | Stack |
|---------|------------|-------|
| CLAUDE-BACKEND.md | Backend | .NET 10, C#, Hexagonal, PostgreSQL, Redis, Semantic Kernel |
| CLAUDE-FRONTEND.md | Frontend | Angular 21, Angular Material, Tailwind CSS, Monaco Editor |

> La documentacion vive en el repositorio de codigo, NO en la wiki. La wiki sirve para humanos, el repo para la IA (recomendacion de Osvaldo Arias).

---

## Alineacion DDD

El dominio del Spec Agent Portal aplica Domain-Driven Design de forma pragmatica:

- **Bounded Context**: Gestion de especificaciones y gobernanza
- **Agregados**: Specification, ReviewCycle, Project, Skill, Prompt
- **Eventos de dominio**: SpecApproved, SpecStatusChanged (consumidos por DevOps Sync y Metrics)
- **Puertos (ACL)**: ILLMProvider, IDevOpsWorkItemAdapter, IDLPFilter (ADR-001)
- **Lenguaje ubicuo**: definido en Glosario de SPEC-L1 con columna "Nombre en codigo"
