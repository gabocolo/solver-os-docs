# Especificacion de Sistema (Nivel 2)

| Campo | Valor |
|-------|-------|
| **Spec ID** | SPEC-L2-manage-projects |
| **Nivel** | L2 — Sistema |
| **Version** | 1.0.0 |
| **Estado** | DRAFT |
| **Parent** | SPEC-L1-spec-agent-portal v1.0.0 |
| **Caso de uso** | CU-01: Gestionar proyecto y gobernanza |
| **Creado por** | Equipo de Arquitectura |
| **Fecha de creacion** | 2026-04-09 |
| **Aprobado por** | Pendiente |
| **Fecha de aprobacion** | Pendiente |
| **PR** | Pendiente |

---

## Descripcion general

Modulo de gestion de proyectos y gobernanza del Spec Agent Portal. Permite a un arquitecto crear proyectos, registrar ADRs (manualmente o asistidos por IA), configurar quality gates y visualizar el dashboard de gobernanza. Es el primer paso del flujo del framework: establecer las reglas antes de especificar.

---

## Sistema

**Nombre:** Governance Service + Project Management

## Caso de uso

**Nombre:** ManageProjectsAndGovernance

**Descripcion:** El arquitecto crea un proyecto con nombre, descripcion y URL de repositorio Git. Luego configura la gobernanza: registra ADRs criticos (manualmente o asistidos por IA), define estandares aplicables y configura los quality gates del proyecto. El portal muestra un dashboard de gobernanza con el estado completo.

---

## Contrato tecnico

### Input — CreateProject

| Campo | Tipo | Requerido | Descripcion |
|-------|------|-----------|-------------|
| name | string | Si | Nombre unico del proyecto (max 200 chars) |
| description | string | No | Descripcion del proyecto |
| gitRepoUrl | string | No | URL del repositorio Git asociado |

### Input — CreateADR

| Campo | Tipo | Requerido | Descripcion |
|-------|------|-----------|-------------|
| projectId | UUID | Si | Proyecto al que pertenece |
| number | integer | Si | Numero secuencial (auto-incrementable) |
| title | string | Si | Titulo del ADR (max 300 chars) |
| status | ADRStatus | Si | Estado inicial: PROPOSED o ACCEPTED |
| content | string | Si | Contenido en markdown |

### Input — CreateADRWithAI

| Campo | Tipo | Requerido | Descripcion |
|-------|------|-----------|-------------|
| projectId | UUID | Si | Proyecto al que pertenece |
| description | string | Si | Descripcion de la decision en lenguaje natural |
| context | string | No | Contexto adicional (alternativas consideradas, restricciones) |

### Input — ConfigureQualityGates

| Campo | Tipo | Requerido | Descripcion |
|-------|------|-----------|-------------|
| projectId | UUID | Si | Proyecto a configurar |
| gates | QualityGateConfig[] | Si | Lista de gates con configuracion |

### QualityGateConfig

| Campo | Tipo | Requerido | Descripcion |
|-------|------|-----------|-------------|
| gateNumber | integer | Si | Numero del gate (1, 2, 3) |
| name | string | Si | Nombre descriptivo |
| validations | string[] | Si | Que valida este gate |
| blocking | boolean | Si | Si bloquea la transicion al siguiente paso |

### Output — ProjectResponse

| Campo | Tipo | Descripcion |
|-------|------|-------------|
| projectId | UUID | ID del proyecto creado |
| name | string | Nombre |
| description | string | Descripcion |
| gitRepoUrl | string | URL del repo |
| ownerId | UUID | ID del arquitecto que creo |
| adrs | ADRSummary[] | Lista de ADRs del proyecto |
| qualityGates | QualityGateConfig[] | Configuracion de gates |
| createdAt | datetime | Fecha de creacion |

### Output — ADRResponse

| Campo | Tipo | Descripcion |
|-------|------|-------------|
| adrId | UUID | ID del ADR |
| number | integer | Numero secuencial |
| title | string | Titulo |
| status | ADRStatus | Estado |
| content | string | Contenido completo |
| createdAt | datetime | Fecha de creacion |

### Output — ADRAIDraft

| Campo | Tipo | Descripcion |
|-------|------|-------------|
| title | string | Titulo sugerido por IA |
| content | string | Borrador del ADR con contexto, alternativas, consecuencias |
| model | string | Modelo LLM usado |
| tokensUsed | integer | Tokens consumidos |

---

## Dependencias permitidas

| Dependencia | Tipo | Proposito |
|-------------|------|----------|
| ProjectRepository | Puerto (persistencia) | CRUD de proyectos |
| ADRRepository | Puerto (persistencia) | CRUD de ADRs |
| QualityGateRepository | Puerto (persistencia) | Configuracion de quality gates |
| LLMProvider | Puerto (IA) | Generacion asistida de ADRs (opcional) |
| DLPFilter | Puerto (seguridad) | Sanitizacion pre-prompt al generar ADR con IA |
| AuditLogRepository | Puerto (auditoria) | Registro de acciones criticas |

---

## Errores

| Codigo | Descripcion | HTTP Status |
|--------|-------------|-------------|
| PROJECT_NAME_DUPLICATE | Ya existe un proyecto con ese nombre | 409 Conflict |
| PROJECT_NOT_FOUND | El proyecto no existe | 404 Not Found |
| ADR_NUMBER_DUPLICATE | Ya existe un ADR con ese numero en el proyecto | 409 Conflict |
| ADR_NOT_FOUND | El ADR no existe | 404 Not Found |
| INVALID_GATE_CONFIG | Configuracion de quality gate invalida | 400 Bad Request |
| UNAUTHORIZED | Usuario no tiene rol ARCHITECT | 403 Forbidden |
| LLM_UNAVAILABLE | LLM no disponible para generar ADR asistido | 503 Service Unavailable |

---

## Requisitos de ejecucion

| Aspecto | Requisito |
|---------|----------|
| **Latencia** | CRUD: < 500ms (p95). Generacion ADR con IA: < 60s (p95) |
| **Idempotencia** | Crear proyecto es idempotente por nombre (retorna existente si duplicado con mismos datos) |
| **Observabilidad** | Logs estructurados con correlationId. Metricas: `project_created_total`, `adr_created_total`, `llm_adr_generation_duration_ms` |
| **Asincronia** | Generacion de ADR con IA es asincrona via Azure Service Bus |
| **Autorizacion** | Solo ARCHITECT puede crear/editar proyectos y gestionar ADRs. LEAD puede gestionar ADRs |

---

## Proteccion de datos

| Campo | Clasificacion | Tratamiento |
|-------|---------------|-------------|
| Project.gitRepoUrl | Interno | No exponer en APIs publicas |
| ADR.content | Confidencial | Sanitizar antes de enviar a LLM para generacion asistida |
| User context (al generar ADR con IA) | Confidencial | DLP pre-prompt: eliminar datos sensibles innecesarios |

---

## Escenarios de test (GWT)

### GWT-01: Crear proyecto exitosamente

```
Given un usuario autenticado con rol ARCHITECT
When crea un proyecto con nombre "Account Planning", descripcion y URL de repo
Then el sistema crea el proyecto con un projectId unico
  And registra la accion en AuditLog
  And retorna 201 Created con el proyecto creado
```

### GWT-02: Rechazar proyecto con nombre duplicado

```
Given un proyecto existente con nombre "Account Planning"
When un arquitecto intenta crear otro proyecto con el mismo nombre
Then el sistema retorna 409 Conflict con error PROJECT_NAME_DUPLICATE
```

### GWT-03: Registrar ADR manualmente

```
Given un proyecto existente con projectId valido
  And un usuario con rol ARCHITECT
When registra un ADR con numero 001, titulo "Ports & Adapters" y contenido en markdown
Then el sistema persiste el ADR asociado al proyecto
  And asigna adrId unico
  And registra en AuditLog
```

### GWT-04: Generar ADR asistido por IA

```
Given un proyecto existente
  And un usuario con rol ARCHITECT
  And el LLM esta disponible
When solicita generar ADR con descripcion "Definir estrategia de testing con 3 niveles"
Then el sistema sanitiza el input (DLP pre-prompt)
  And envia al LLM (Sonnet) con contexto del proyecto
  And retorna un borrador de ADR con titulo, contexto, alternativas y consecuencias
  And registra la invocacion en AuditLog (modelo usado, tokens consumidos)
```

### GWT-05: Configurar quality gates

```
Given un proyecto existente
  And un usuario con rol ARCHITECT
When configura Gate 1 con validaciones [estructura, clasificacion, ADRs] y blocking=true
  And Gate 2 con validaciones [tests, SAST, SCA, DLP] y blocking=true
  And Gate 3 con validaciones [revision senior] y blocking=true
Then el sistema persiste la configuracion de los 3 gates
```

### GWT-06: Rechazar acceso a usuario sin permisos

```
Given un usuario autenticado con rol SENIOR_DEV
When intenta crear un proyecto
Then el sistema retorna 403 Forbidden con error UNAUTHORIZED
```

---

## Referencias

- SPEC-L1-spec-agent-portal v1.0.0 — CU-01
- RF-GOV-01 a RF-GOV-07
- RNF-SEC-01, RNF-SEC-02 (autenticacion y autorizacion)
- ADR-001 (Ports & Adapters)
