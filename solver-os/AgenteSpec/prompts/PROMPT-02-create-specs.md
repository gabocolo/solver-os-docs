# Prompt 02 — Implementar Spec Management + Agent Orchestration

| Campo | Valor |
|-------|-------|
| **Spec** | SPEC-L2-create-specs v1.0.0 |
| **Skill** | SK-001 — Generacion de codigo de caso de uso |
| **Modelo recomendado** | Sonnet (tarea acotada por spec) |

---

## Prompt

```
## Rol
Actua como un desarrollador senior con experiencia en .NET 10, C#, Semantic Kernel y arquitectura hexagonal.

## Invocacion de especificacion
Aplica la especificacion SPEC-L2-create-specs version 1.0.0.
[ADJUNTAR: contenido completo de SPEC-L2-create-specs.md]

## Tarea
Implementa el caso de uso CreateSpecWithAI segun la spec. Esto incluye:
1. Entidad de dominio Specification con sus value objects (SpecLevel, SpecStatus)
2. Puertos: ISpecRepository, ILLMProvider, IDLPFilter, ITemplateProvider, IADRRepository, IEventPublisher, IAuditLogRepository
3. Servicio de aplicacion SpecManagementService: GenerateSpecL1, GenerateSpecL2, GenerateSpecL3, SaveSpecDraft, RegenerateSection
4. Job handler para generacion asincrona (worker que dequeue de Service Bus)
5. Endpoints Minimal API para cada operacion

## Entrada
- GenerateSpecL1: projectId, description (lenguaje natural), additionalContext (opcional)
- GenerateSpecL2: projectId, parentSpecId (L1 APPROVED), useCaseId, technicalContext (opcional)
- GenerateSpecL3: projectId, parentSpecId (L2 APPROVED), changeDescription
- SaveSpecDraft: specId?, projectId, level, parentSpecId?, title, content (JSONB), dataClassification?
- RegenerateSection: specId, section, additionalInstructions?

## Salida esperada
- GenerationJobResponse: jobId, status (QUEUED), estimatedTime
- SpecDraftResponse: specId, projectId, level, parentSpecId, version, status (DRAFT), title, content, dataClassification, model, tokensUsed, createdBy, createdAt

## Reglas
- Todas las reglas, dependencias y errores provienen EXCLUSIVAMENTE de la spec.
- NO inventes servicios ni dependencias fuera de la spec.
- Dependencias permitidas: SpecRepository, LLMProvider, DLPFilter, TemplateProvider, ADRRepository, EventPublisher, AuditLogRepository, Azure Service Bus.
- Errores: PARENT_NOT_APPROVED (422), PARENT_NOT_FOUND (404), PARENT_WRONG_LEVEL (422), DUPLICATE_SPEC (409), INCOMPLETE_STRUCTURE (422), MISSING_DATA_CLASSIFICATION (422), INVALID_VERSION (422), LLM_UNAVAILABLE (503), LLM_TIMEOUT (504), DLP_VIOLATION (500), UNAUTHORIZED (403), SECTION_NOT_FOUND (400).
- Modelo LLM: Opus para L1, Sonnet para L2/L3/regenerar (ADR-P001).
- Generacion es ASINCRONA via Azure Service Bus (ADR-P002).
- Circuit breaker: si LLM falla 3 veces consecutivas, pausar 60s.
- Reintentos: max 3 con backoff exponencial (1s, 2s, 4s).
- Versionado: 1.0.0 al crear, incremento automatico segun semver.

## Contexto de gobernanza adjunto
- ADR-001: Ports & Adapters
- ADR-002: Idempotencia — guardar spec es idempotente por specId (upsert)
- ADR-003: Trazabilidad — correlationId propagado al job
- ADR-006: Clasificacion de datos y DLP
- ADR-P001: Seleccion de modelos LLM
- ADR-P002: Generacion asincrona via Service Bus
- ADR-P003: Almacenamiento en JSONB

## Restricciones DLP (ADR-006)
- No incluyas datos reales (PII) en el codigo, tests o fixtures.
- Input del usuario es Confidencial — DLP pre-prompt antes de enviar al LLM.
- Respuesta del LLM es Confidencial — DLP post-response para detectar PII generada.
- ADRs en contexto son Confidencial — solo incluir los relevantes, no todos.
- AuditLog: registrar modelo y tokens, NO input completo (solo sanitizado).
- Spec content es Confidencial — propiedad intelectual.

## Restricciones de salida
- Dividir en PRs: 1) Entidades + puertos, 2) Servicio + templates, 3) Job handler + Service Bus, 4) Endpoints API
- Cada PR < 400 lineas
- Tests basados en GWT-01 a GWT-08 de la spec
- Codigo en la capa correcta: Domain/, Application/, Infrastructure/LLM/, Infrastructure/Messaging/, API/
```

---

## Contexto a adjuntar

| Documento | Seccion |
|-----------|---------|
| SPEC-L2-create-specs.md | Completo |
| ADR-001 | Ports & Adapters |
| ADR-006 | Data Classification & DLP |
| ADR-P001 | LLM Model Selection |
| ADR-P002 | Async Generation |
| ADR-P003 | JSONB Storage |
| CODE-STANDARDS.md | .NET + Semantic Kernel |
