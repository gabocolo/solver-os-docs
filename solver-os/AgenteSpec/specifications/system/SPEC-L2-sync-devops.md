# Especificacion de Sistema (Nivel 2)

| Campo | Valor |
|-------|-------|
| **Spec ID** | SPEC-L2-sync-devops |
| **Nivel** | L2 — Sistema |
| **Version** | 1.0.0 |
| **Estado** | DRAFT |
| **Parent** | SPEC-L1-spec-agent-portal v1.1.0 |
| **Caso de uso** | CU-08: Sincronizar spec aprobada con Azure DevOps |
| **Creado por** | Equipo de Arquitectura |
| **Fecha de creacion** | 2026-04-09 |
| **Aprobado por** | Pendiente |
| **Fecha de aprobacion** | Pendiente |
| **PR** | Pendiente |

---

## Descripcion general

Modulo de sincronizacion entre el Spec Agent Portal y Azure DevOps. Cuando una spec se aprueba, el portal crea automaticamente un work item en el proyecto Azure DevOps configurado (`https://dev.azure.com/juancolorado0526/IA%20Automatizaciones/`). El tipo de work item depende del nivel de spec: Feature (L1), User Story (L2), Task (L3). Se mantiene la relacion parent-child entre work items segun la jerarquia de specs. La sincronizacion es asincrona y tolerante a fallos: si Azure DevOps no esta disponible, la spec queda aprobada y el sync se reintenta.

---

## Sistema

**Nombre:** DevOps Sync Service (adaptador de integracion)

## Caso de uso

**Nombre:** SyncSpecToAzureDevOps

**Descripcion:** Al publicarse el evento `SpecApproved`, el servicio de sincronizacion crea un work item en Azure DevOps con la informacion relevante de la spec, establece relaciones parent-child, y registra el resultado. Si falla, el error se registra para reintento. El usuario puede ver el link al work item desde el portal y reintentar manualmente si es necesario.

---

## Contrato tecnico

### Input — SyncSpecEvent (evento interno)

| Campo | Tipo | Requerido | Descripcion |
|-------|------|-----------|-------------|
| specId | UUID | Si | Spec recien aprobada |
| projectId | UUID | Si | Proyecto al que pertenece |
| level | SpecLevel | Si | L1, L2 o L3 |
| parentSpecId | UUID | No | Parent spec (para relacion en DevOps) |
| title | string | Si | Titulo de la spec |
| version | string | Si | Version semver |
| approvedBy | UUID | Si | Quien aprobo |
| approvedAt | datetime | Si | Cuando se aprobo |

### Input — ConfigureDevOpsProject

| Campo | Tipo | Requerido | Descripcion |
|-------|------|-----------|-------------|
| projectId | UUID | Si | Proyecto del portal |
| devOpsOrganization | string | Si | Organizacion Azure DevOps (ej: `juancolorado0526`) |
| devOpsProject | string | Si | Nombre del proyecto DevOps (ej: `IA Automatizaciones`) |
| devOpsAreaPath | string | No | Area Path para los work items (default: raiz del proyecto) |
| devOpsPAT | string | Si | Personal Access Token con permisos de Work Items (Write) |

### Input — RetrySyncRequest

| Campo | Tipo | Requerido | Descripcion |
|-------|------|-----------|-------------|
| syncId | UUID | Si | ID del registro de sincronizacion fallido |

### Output — WorkItemSyncResponse

| Campo | Tipo | Descripcion |
|-------|------|-------------|
| syncId | UUID | ID del registro de sincronizacion |
| specId | UUID | Spec sincronizada |
| devOpsWorkItemId | integer | ID del work item creado en Azure DevOps |
| devOpsWorkItemUrl | string | URL completa al work item (clickable) |
| workItemType | string | Feature, User Story o Task |
| syncStatus | SyncStatus | PENDING, SYNCED, FAILED |
| parentWorkItemId | integer | ID del work item parent (si aplica) |
| syncedAt | datetime | Fecha de sincronizacion |
| error | string | Mensaje de error (si FAILED) |

### Output — DevOpsConfigResponse

| Campo | Tipo | Descripcion |
|-------|------|-------------|
| projectId | UUID | Proyecto del portal |
| devOpsOrganization | string | Organizacion configurada |
| devOpsProject | string | Proyecto configurado |
| devOpsProjectUrl | string | URL completa (`https://dev.azure.com/{org}/{project}`) |
| devOpsAreaPath | string | Area Path |
| isConfigured | boolean | Si la configuracion esta completa y validada |
| lastSyncAt | datetime | Ultima sincronizacion exitosa |

---

## Mapeo spec → work item

| Nivel de Spec | Tipo Work Item | Contenido | Relacion Parent |
|---------------|---------------|-----------|-----------------|
| **L1** (Dominio) | **Feature** | Titulo: `[SPEC-L1] {titulo} v{version}`. Descripcion: objetivo de negocio, entidades, reglas de negocio. Tags: `spec-agent`, `L1`, `v{version}` | Ninguna (raiz) |
| **L2** (Sistema) | **User Story** | Titulo: `[SPEC-L2] {titulo} v{version}`. Descripcion: caso de uso, contrato tecnico (resumen). Acceptance Criteria: escenarios GWT. Tags: `spec-agent`, `L2`, `v{version}` | Parent → Feature de la L1 |
| **L3** (Cambio) | **Task** | Titulo: `[SPEC-L3] {titulo} v{version}`. Descripcion: cambio, impacto en contrato, impacto en comportamiento. Tags: `spec-agent`, `L3`, `v{version}` | Parent → User Story de la L2 |

### Campos del work item en Azure DevOps

| Campo Azure DevOps | Fuente | Ejemplo |
|--------------------|--------|---------|
| `System.Title` | `[SPEC-{level}] {spec.title} v{spec.version}` | `[SPEC-L2] CreateBooking v1.0.0` |
| `System.Description` | HTML generado desde contenido de la spec | Descripcion + link al portal |
| `Microsoft.VSTS.Common.AcceptanceCriteria` | Escenarios GWT (L2) o reglas de negocio (L1) | GWT-01, GWT-02... en formato HTML |
| `System.Tags` | `spec-agent; {level}; v{version}` | `spec-agent; L2; v1.0.0` |
| `System.AreaPath` | Configuracion del proyecto | `IA Automatizaciones\Specs` |
| `System.AssignedTo` | No asignar (el equipo asigna desde Boards) | — |
| Custom field o link | URL al spec en el portal | `https://portal/specs/{specId}` |

---

## Dependencias permitidas

| Dependencia | Tipo | Proposito |
|-------------|------|----------|
| AzureDevOpsWorkItemAdapter | Puerto (integracion) | Crear, actualizar y consultar work items via Azure DevOps REST API |
| SpecRepository | Puerto (persistencia) | Leer contenido completo de la spec para generar descripcion |
| WorkItemSyncRepository | Puerto (persistencia) | Registrar estado de sincronizacion |
| ProjectRepository | Puerto (persistencia) | Obtener configuracion de Azure DevOps del proyecto |
| EventSubscriber | Puerto (eventos) | Escuchar evento SpecApproved |
| AuditLogRepository | Puerto (auditoria) | Registrar sincronizacion exitosa o fallida |

---

## Azure DevOps REST API

**Base URL:** `https://dev.azure.com/{organization}/{project}/_apis/wit/workitems`

**Autenticacion:** Personal Access Token (PAT) con scope `Work Items (Read, Write)`

### Crear work item

```
POST https://dev.azure.com/{org}/{project}/_apis/wit/workitems/${type}?api-version=7.1
Authorization: Basic {base64(:PAT)}
Content-Type: application/json-patch+json

[
  { "op": "add", "path": "/fields/System.Title", "value": "[SPEC-L2] CreateBooking v1.0.0" },
  { "op": "add", "path": "/fields/System.Description", "value": "<html>...</html>" },
  { "op": "add", "path": "/fields/Microsoft.VSTS.Common.AcceptanceCriteria", "value": "<html>...</html>" },
  { "op": "add", "path": "/fields/System.Tags", "value": "spec-agent; L2; v1.0.0" },
  { "op": "add", "path": "/fields/System.AreaPath", "value": "IA Automatizaciones" }
]
```

### Crear relacion parent-child

```
PATCH https://dev.azure.com/{org}/{project}/_apis/wit/workitems/{childId}?api-version=7.1

[
  {
    "op": "add",
    "path": "/relations/-",
    "value": {
      "rel": "System.LinkTypes.Hierarchy-Reverse",
      "url": "https://dev.azure.com/{org}/{project}/_apis/wit/workitems/{parentId}"
    }
  }
]
```

---

## Errores

| Codigo | Descripcion | HTTP Status |
|--------|-------------|-------------|
| DEVOPS_NOT_CONFIGURED | El proyecto no tiene configuracion de Azure DevOps | 422 Unprocessable |
| DEVOPS_AUTH_FAILED | PAT invalido o sin permisos suficientes | 401 Unauthorized |
| DEVOPS_PROJECT_NOT_FOUND | El proyecto Azure DevOps no existe o no es accesible | 404 Not Found |
| DEVOPS_API_ERROR | Error generico de la API de Azure DevOps | 502 Bad Gateway |
| DEVOPS_TIMEOUT | Timeout al comunicarse con Azure DevOps | 504 Gateway Timeout |
| DEVOPS_PARENT_WI_NOT_FOUND | No se encontro el work item parent para establecer relacion | 422 Unprocessable |
| SYNC_NOT_FOUND | El registro de sincronizacion no existe | 404 Not Found |
| SYNC_ALREADY_COMPLETED | El work item ya fue creado exitosamente | 409 Conflict |

---

## Requisitos de ejecucion

| Aspecto | Requisito |
|---------|----------|
| **Latencia** | Creacion de work item: < 5s (p95). Depende de Azure DevOps API |
| **Asincronia** | La sincronizacion es asincrona (via evento SpecApproved). No bloquea la aprobacion |
| **Tolerancia a fallos** | Si Azure DevOps no esta disponible, la spec permanece APPROVED. El error se registra con syncStatus = FAILED |
| **Reintentos** | Max 3 reintentos automaticos con backoff exponencial (5s, 15s, 45s). Si falla, queda FAILED para reintento manual |
| **Idempotencia** | Si ya existe un WorkItemSync con syncStatus = SYNCED para esa spec, no crear duplicado |
| **Observabilidad** | `devops_sync_total`, `devops_sync_success_total`, `devops_sync_failed_total`, `devops_sync_duration_ms` |
| **Autorizacion** | La configuracion de Azure DevOps solo la puede hacer ARCHITECT. La sincronizacion se ejecuta automaticamente |
| **Seguridad** | El PAT se almacena cifrado (vault o secret manager). Nunca en logs ni en AuditLog |

---

## Proteccion de datos

| Campo / Flujo | Clasificacion | Tratamiento |
|---------------|---------------|-------------|
| devOpsPAT | **Sensible** | Cifrado en reposo, nunca en logs, nunca en AuditLog. Almacenar en vault |
| Work item description | Confidencial | Contiene resumen de la spec (propiedad intelectual). Solo visible en Azure DevOps con permisos |
| devOpsWorkItemUrl | Interno | URL interna de Azure DevOps |
| Acceptance criteria | Confidencial | Escenarios GWT pueden contener reglas de negocio |

---

## Escenarios de test (GWT)

### GWT-01: Sincronizar spec L2 aprobada como User Story

```
Given una spec L2 "CreateBooking v1.0.0" recien aprobada
  And el proyecto tiene Azure DevOps configurado (org: juancolorado0526, project: IA Automatizaciones)
  And la L1 parent tiene un work item Feature sincronizado (WI #42)
When el evento SpecApproved se procesa
Then el sistema:
  1. Crea un User Story en Azure DevOps con titulo "[SPEC-L2] CreateBooking v1.0.0"
  2. Incluye descripcion con link al portal y resumen del contrato
  3. Incluye acceptance criteria con los escenarios GWT
  4. Agrega tags: "spec-agent; L2; v1.0.0"
  5. Establece relacion Parent con Feature #42
  6. Registra en WorkItemSync: syncStatus = SYNCED, devOpsWorkItemId = {id}
  7. Registra en AuditLog: DEVOPS_WORK_ITEM_CREATED
```

### GWT-02: Sincronizar spec L1 aprobada como Feature

```
Given una spec L1 "Booking v1.0.0" recien aprobada
  And el proyecto tiene Azure DevOps configurado
When el evento SpecApproved se procesa
Then el sistema crea una Feature en Azure DevOps
  And no establece relacion Parent (L1 es raiz)
  And incluye reglas de negocio en acceptance criteria
```

### GWT-03: Sincronizar spec L3 como Task

```
Given una spec L3 "AddCoupon v1.1.0" recien aprobada
  And la L2 parent tiene un work item User Story sincronizado (WI #55)
When el evento SpecApproved se procesa
Then el sistema crea una Task en Azure DevOps
  And establece relacion Parent con User Story #55
  And incluye impacto en contrato y comportamiento en la descripcion
```

### GWT-04: Manejar Azure DevOps no disponible

```
Given una spec recien aprobada
  And Azure DevOps API retorna timeout (503)
When el evento SpecApproved se procesa
Then el sistema:
  1. La spec permanece en estado APPROVED (no se revierte)
  2. Registra WorkItemSync con syncStatus = FAILED y error = "Azure DevOps timeout"
  3. Programa reintento automatico (backoff: 5s, 15s, 45s)
  4. Registra en AuditLog: DEVOPS_SYNC_FAILED
  5. El dashboard muestra indicador de sincronizacion pendiente
```

### GWT-05: Reintento manual de sincronizacion

```
Given una sincronizacion fallida con syncId
  And Azure DevOps ahora esta disponible
When un usuario con rol ARCHITECT solicita reintento manual
Then el sistema:
  1. Reintenta la creacion del work item
  2. Actualiza WorkItemSync: syncStatus = SYNCED
  3. Registra en AuditLog: DEVOPS_SYNC_RETRIED
```

### GWT-06: Evitar duplicacion de work items

```
Given una spec ya sincronizada (WorkItemSync con SYNCED y devOpsWorkItemId = 42)
When el evento SpecApproved se recibe de nuevo (ej: reprocessing)
Then el sistema detecta que ya existe sync exitoso
  And no crea un nuevo work item
  And retorna el work item existente
```

### GWT-07: Configurar Azure DevOps en proyecto

```
Given un usuario con rol ARCHITECT
When configura Azure DevOps para el proyecto con:
  - organization: "juancolorado0526"
  - project: "IA Automatizaciones"
  - PAT: "***" (token valido)
Then el sistema:
  1. Valida la conexion con Azure DevOps API (GET project)
  2. Almacena la configuracion (PAT cifrado en vault)
  3. Registra devOpsProjectUrl: "https://dev.azure.com/juancolorado0526/IA%20Automatizaciones/"
  4. A partir de ahora, toda spec aprobada en este proyecto se sincroniza
```

### GWT-08: Rechazar configuracion con PAT invalido

```
Given un usuario con rol ARCHITECT
When configura Azure DevOps con un PAT invalido o expirado
Then el sistema:
  1. Intenta validar la conexion
  2. Azure DevOps retorna 401
  3. Retorna error DEVOPS_AUTH_FAILED
  4. No guarda la configuracion
```

---

## Flujo de sincronizacion

```
SpecApproved          DevOps Sync Service         Azure DevOps API         WorkItemSync DB
    |                        |                          |                        |
    |-- Evento ------------->|                          |                        |
    |                        |-- Cargar spec completa   |                        |
    |                        |-- Cargar config DevOps   |                        |
    |                        |                          |                        |
    |                        |-- Determinar tipo WI     |                        |
    |                        |   (L1→Feature,           |                        |
    |                        |    L2→User Story,        |                        |
    |                        |    L3→Task)              |                        |
    |                        |                          |                        |
    |                        |-- Generar contenido:     |                        |
    |                        |   titulo, descripcion,   |                        |
    |                        |   acceptance criteria    |                        |
    |                        |                          |                        |
    |                        |-- POST work item ------->|                        |
    |                        |                          |-- Crear WI             |
    |                        |<-- 200 OK (WI #{id}) ----|                        |
    |                        |                          |                        |
    |                        |  [Si tiene parent spec con WI sincronizado]       |
    |                        |-- PATCH relacion -------->|                        |
    |                        |<-- 200 OK ---------------|                        |
    |                        |                          |                        |
    |                        |-- Registrar sync -------------------------------->|
    |                        |   (SYNCED, WI #{id})     |                        |
    |                        |                          |                        |
    |                        |  [Si falla]              |                        |
    |                        |-- Registrar sync -------------------------------->|
    |                        |   (FAILED, error)        |                        |
    |                        |-- Programar reintento    |                        |
```

---

## Referencias

- SPEC-L1-spec-agent-portal v1.1.0 — CU-08, BR-011
- Azure DevOps REST API v7.1: https://learn.microsoft.com/en-us/rest/api/azure/devops/wit/
- Proyecto destino: `https://dev.azure.com/juancolorado0526/IA%20Automatizaciones/`
- ADR-001 (Ports & Adapters) — DevOps sync es un adaptador, no dominio
