# Prompt 07 — Implementar DevOps Sync Service

| Campo | Valor |
|-------|-------|
| **Spec** | SPEC-L2-sync-devops v1.0.0 |
| **Skill** | SK-001 — Generacion de codigo de caso de uso |
| **Modelo recomendado** | Sonnet |

---

## Prompt

```
## Rol
Actua como un desarrollador senior con experiencia en .NET 10, C#, Azure DevOps REST API v7.1 y arquitectura hexagonal.

## Invocacion de especificacion
Aplica la especificacion SPEC-L2-sync-devops version 1.0.0.
[ADJUNTAR: contenido completo de SPEC-L2-sync-devops.md]

## Tarea
Implementa el caso de uso SyncSpecToAzureDevOps segun la spec. Esto incluye:
1. Entidad de dominio WorkItemSync con value objects (SyncStatus)
2. Puertos: IAzureDevOpsWorkItemAdapter, ISpecRepository, IWorkItemSyncRepository, IProjectRepository, IEventSubscriber, IAuditLogRepository
3. Adaptador AzureDevOpsWorkItemAdapter: llamadas a Azure DevOps REST API v7.1 para crear work items y relaciones parent-child
4. Servicio de aplicacion DevOpsSyncService: HandleSpecApproved, ConfigureDevOpsProject, RetrySyncRequest
5. Event handler que escucha SpecApproved
6. Endpoints Minimal API para configuracion y reintento manual

## Entrada
- SyncSpecEvent (evento): specId, projectId, level, parentSpecId?, title, version, approvedBy, approvedAt
- ConfigureDevOpsProject: projectId, devOpsOrganization, devOpsProject, devOpsAreaPath?, devOpsPAT
- RetrySyncRequest: syncId

## Salida esperada
- WorkItemSyncResponse: syncId, specId, devOpsWorkItemId, devOpsWorkItemUrl, workItemType, syncStatus, parentWorkItemId, syncedAt, error
- DevOpsConfigResponse: projectId, devOpsOrganization, devOpsProject, devOpsProjectUrl, devOpsAreaPath, isConfigured, lastSyncAt

## Reglas
- Todas las reglas provienen EXCLUSIVAMENTE de la spec.
- Dependencias: AzureDevOpsWorkItemAdapter, SpecRepository, WorkItemSyncRepository, ProjectRepository, EventSubscriber, AuditLogRepository.
- Mapeo: L1 → Feature, L2 → User Story, L3 → Task.
- Relacion parent-child: User Story → parent Feature, Task → parent User Story.
- Errores: DEVOPS_NOT_CONFIGURED (422), DEVOPS_AUTH_FAILED (401), DEVOPS_PROJECT_NOT_FOUND (404), DEVOPS_API_ERROR (502), DEVOPS_TIMEOUT (504), DEVOPS_PARENT_WI_NOT_FOUND (422), SYNC_NOT_FOUND (404), SYNC_ALREADY_COMPLETED (409).
- No bloquear aprobacion: si DevOps falla, spec queda APPROVED.
- Reintentos: max 3 con backoff (5s, 15s, 45s).
- Idempotente: si sync ya SYNCED, no crear duplicado.
- Azure DevOps API:
  - POST /_apis/wit/workitems/${type}?api-version=7.1
  - PATCH /_apis/wit/workitems/{id}?api-version=7.1 (para relacion parent)
  - Auth: Basic con PAT

## Contexto de gobernanza adjunto
- ADR-001: Ports & Adapters — DevOps sync es un adaptador de infraestructura
- ADR-003: Trazabilidad
- ADR-P004: Estrategia de sincronizacion con Azure DevOps

## Restricciones DLP (ADR-006)
- No incluyas datos reales (PII) en el codigo, tests o fixtures.
- devOpsPAT es **Sensible** — cifrado en reposo (vault), NUNCA en logs ni AuditLog.
- Work item description es Confidencial — contiene resumen de spec.
- Acceptance criteria son Confidencial — contienen reglas de negocio.
- En tests usar PAT sintetico: "test-pat-synthetic-000".

## Restricciones de salida
- Dividir en PRs: 1) Entidades + puertos, 2) AzureDevOpsAdapter (HTTP client), 3) DevOpsSyncService + event handler, 4) Endpoints + config
- Cada PR < 400 lineas
- Tests basados en GWT-01 a GWT-08 de la spec
- Mock del Azure DevOps API en tests (no llamar API real)
- Incluir test de tolerancia a fallos (GWT-04)
- Incluir test de idempotencia (GWT-06)
```

---

## Contexto a adjuntar

| Documento | Seccion |
|-----------|---------|
| SPEC-L2-sync-devops.md | Completo |
| ADR-001 | Ports & Adapters |
| ADR-003 | Trazabilidad |
| ADR-P004 | DevOps Sync Strategy |
| Azure DevOps REST API v7.1 docs | Work Items section |
