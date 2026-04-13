# Prompt 01 — Implementar Governance Service (Proyectos + ADRs + Quality Gates)

| Campo | Valor |
|-------|-------|
| **Spec** | SPEC-L2-manage-projects v1.0.0 |
| **Skill** | SK-001 — Generacion de codigo de caso de uso |
| **Modelo recomendado** | Sonnet (tarea acotada por spec) |

---

## Prompt

```
## Rol
Actua como un desarrollador senior con experiencia en .NET 10, C# y arquitectura hexagonal.

## Invocacion de especificacion
Aplica la especificacion SPEC-L2-manage-projects version 1.0.0.
[ADJUNTAR: contenido completo de SPEC-L2-manage-projects.md]

## Tarea
Implementa el caso de uso ManageProjectsAndGovernance segun la spec. Esto incluye:
1. Entidad de dominio Project con sus value objects
2. Entidad de dominio ADR
3. Entidad de dominio QualityGateCheck
4. Puertos (interfaces): IProjectRepository, IADRRepository, IQualityGateRepository, ILLMProvider, IDLPFilter, IAuditLogRepository
5. Servicio de aplicacion GovernanceService con operaciones: CreateProject, CreateADR, CreateADRWithAI, ConfigureQualityGates
6. Endpoints Minimal API para cada operacion

## Entrada
- CreateProject: name (string, max 200, unico), description (string, opcional), gitRepoUrl (string, opcional)
- CreateADR: projectId (UUID), number (int), title (string, max 300), status (ADRStatus), content (markdown)
- CreateADRWithAI: projectId (UUID), description (string), context (string, opcional)
- ConfigureQualityGates: projectId (UUID), gates (QualityGateConfig[])

## Salida esperada
- ProjectResponse: projectId, name, description, gitRepoUrl, ownerId, adrs[], qualityGates[], createdAt
- ADRResponse: adrId, number, title, status, content, createdAt
- ADRAIDraft: title, content, model, tokensUsed

## Reglas
- Todas las reglas, dependencias y errores provienen EXCLUSIVAMENTE de la spec.
- NO inventes servicios, reglas ni dependencias fuera de la spec.
- Solo usa las dependencias listadas: ProjectRepository, ADRRepository, QualityGateRepository, LLMProvider, DLPFilter, AuditLogRepository.
- Errores definidos: PROJECT_NAME_DUPLICATE (409), PROJECT_NOT_FOUND (404), ADR_NUMBER_DUPLICATE (409), ADR_NOT_FOUND (404), INVALID_GATE_CONFIG (400), UNAUTHORIZED (403), LLM_UNAVAILABLE (503).
- Codigo en la capa correcta segun ADR-001 (hexagonal).
- Solo ARCHITECT puede crear/editar proyectos. LEAD puede gestionar ADRs.

## Contexto de gobernanza adjunto
- ADR-001: Ports & Adapters — dominio depende de puertos, no de adaptadores
- ADR-002: Idempotencia — crear proyecto es idempotente por nombre
- ADR-003: Trazabilidad — correlationId en cada operacion
- ADR-P001: Sonnet para generacion de ADR asistido
- ADR-P002: Generacion asincrona para ADR con IA via Service Bus

## Restricciones DLP (ADR-006)
- No incluyas datos reales (PII) en el codigo, tests o fixtures — usa datos sinteticos.
- Project.gitRepoUrl es Interno — no exponer en APIs publicas.
- ADR.content es Confidencial — sanitizar antes de enviar a LLM.
- Aplica DLP pre-prompt al generar ADR con IA.

## Restricciones de salida
- Diff pequeno y revisable (< 400 lineas por PR)
- Si el diff es mayor, divide en PRs: 1) entidades + puertos, 2) servicio + endpoints
- Codigo en la capa correcta: Domain/ para entidades y puertos, Application/ para servicios, API/ para endpoints
- Tests incluidos basados en los escenarios GWT de la spec (GWT-01 a GWT-06)
- Cada archivo generado debe tener un proposito claro
```

---

## Contexto a adjuntar

| Documento | Seccion |
|-----------|---------|
| SPEC-L2-manage-projects.md | Completo |
| ADR-001 | Ports & Adapters |
| ADR-002 | Idempotencia |
| ADR-003 | Trazabilidad |
| CODE-STANDARDS.md | Secciones relevantes a .NET |
