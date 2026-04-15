# Especificacion de Sistema (Nivel 2)

| Campo | Valor |
|-------|-------|
| **Spec ID** | SPEC-L2-create-specs |
| **Nivel** | L2 — Sistema |
| **Version** | 1.0.0 |
| **Estado** | DRAFT |
| **Parent** | SPEC-L1-spec-agent-portal v1.0.0 |
| **Caso de uso** | CU-02: Crear spec asistida por IA |
| **Creado por** | Equipo de Arquitectura |
| **Fecha de creacion** | 2026-04-09 |
| **Aprobado por** | Pendiente |
| **Fecha de aprobacion** | Pendiente |
| **PR** | Pendiente |

---

## Descripcion general

Modulo central del Spec Agent Portal. Permite crear especificaciones en los 3 niveles (L1, L2, L3) con asistencia de IA. El usuario describe el dominio o cambio en lenguaje natural, el agente genera un borrador completo con la estructura correcta, y el usuario edita y guarda como DRAFT. Implementa DLP pre/post-prompt, seleccion automatica de modelo LLM y generacion asincrona.

---

## Sistema

**Nombre:** Spec Management Service + Agent Orchestration Service

## Caso de uso

**Nombre:** CreateSpecWithAI

**Descripcion:** El desarrollador senior selecciona un proyecto y el nivel de spec a crear. Para L1, describe el dominio en lenguaje natural; para L2, selecciona la L1 parent aprobada y un caso de uso a detallar; para L3, selecciona la L2 parent y describe el cambio. El agente IA genera un borrador completo que el usuario revisa, edita y guarda como DRAFT.

---

## Contrato tecnico

### Input — GenerateSpecL1

| Campo | Tipo | Requerido | Descripcion |
|-------|------|-----------|-------------|
| projectId | UUID | Si | Proyecto destino |
| description | string | Si | Descripcion del dominio en lenguaje natural (problema, entidades, reglas) |
| additionalContext | string | No | Contexto adicional (restricciones, integraciones conocidas) |

### Input — GenerateSpecL2

| Campo | Tipo | Requerido | Descripcion |
|-------|------|-----------|-------------|
| projectId | UUID | Si | Proyecto destino |
| parentSpecId | UUID | Si | Spec L1 parent (debe estar APPROVED) |
| useCaseId | string | Si | ID del caso de uso de la L1 a detallar (ej: CU-01) |
| technicalContext | string | No | Contexto tecnico adicional (stack, integraciones, restricciones) |

### Input — GenerateSpecL3

| Campo | Tipo | Requerido | Descripcion |
|-------|------|-----------|-------------|
| projectId | UUID | Si | Proyecto destino |
| parentSpecId | UUID | Si | Spec L2 parent (debe estar APPROVED) |
| changeDescription | string | Si | Descripcion del cambio en lenguaje natural |

### Input — SaveSpecDraft

| Campo | Tipo | Requerido | Descripcion |
|-------|------|-----------|-------------|
| specId | UUID | Si | ID de la spec (si ya existe) o null (si nueva) |
| projectId | UUID | Si | Proyecto destino |
| level | SpecLevel | Si | L1, L2 o L3 |
| parentSpecId | UUID | Condicional | Requerido para L2 y L3 |
| title | string | Si | Titulo de la spec (max 300 chars) |
| content | object | Si | Contenido estructurado segun nivel (JSONB) |
| dataClassification | object | Condicional | Requerido para L1 |

### Input — RegenerateSection

| Campo | Tipo | Requerido | Descripcion |
|-------|------|-----------|-------------|
| specId | UUID | Si | Spec existente |
| section | string | Si | Seccion a regenerar (ej: "entidades", "reglas", "casosDeUso") |
| additionalInstructions | string | No | Instrucciones adicionales para la regeneracion |

### Output — GenerationJobResponse

| Campo | Tipo | Descripcion |
|-------|------|-------------|
| jobId | UUID | ID del job de generacion (para tracking asincrono) |
| status | string | Estado: QUEUED |
| estimatedTime | integer | Tiempo estimado en segundos |

### Output — SpecDraftResponse

| Campo | Tipo | Descripcion |
|-------|------|-------------|
| specId | UUID | ID de la spec |
| projectId | UUID | Proyecto |
| level | SpecLevel | Nivel |
| parentSpecId | UUID | Parent (si aplica) |
| version | string | Version semver asignada automaticamente |
| status | SpecStatus | DRAFT |
| title | string | Titulo |
| content | object | Contenido completo estructurado |
| dataClassification | object | Clasificacion de datos (L1) |
| model | string | Modelo LLM usado (si fue generado por IA) |
| tokensUsed | integer | Tokens consumidos (si fue generado por IA) |
| createdBy | UUID | Autor |
| createdAt | datetime | Fecha |

---

## Dependencias permitidas

| Dependencia | Tipo | Proposito |
|-------------|------|----------|
| SpecRepository | Puerto (persistencia) | CRUD de specs, versionado, busqueda |
| LLMProvider | Puerto (IA) | Generacion de borradores (Opus para L1, Sonnet para L2/L3) |
| DLPFilter | Puerto (seguridad) | Sanitizacion pre-prompt y validacion post-response |
| TemplateProvider | Puerto (config) | Obtener template de spec segun nivel (L1/L2/L3) |
| ADRRepository | Puerto (persistencia) | Obtener ADRs vigentes del proyecto para incluir en contexto |
| EventPublisher | Puerto (eventos) | Publicar evento SpecCreated |
| AuditLogRepository | Puerto (auditoria) | Registrar invocaciones al LLM y creacion de specs |
| Azure Service Bus | Puerto (cola) | Enqueue jobs de generacion asincrona |

---

## Errores

| Codigo | Descripcion | HTTP Status |
|--------|-------------|-------------|
| PARENT_NOT_APPROVED | El spec parent no esta en estado APPROVED | 422 Unprocessable |
| PARENT_NOT_FOUND | El spec parent no existe | 404 Not Found |
| PARENT_WRONG_LEVEL | L2 requiere parent L1, L3 requiere parent L2 | 422 Unprocessable |
| DUPLICATE_SPEC | Ya existe una spec con mismo titulo y nivel en DRAFT/IN_REVIEW | 409 Conflict |
| INCOMPLETE_STRUCTURE | Faltan campos obligatorios para el nivel | 422 Unprocessable |
| MISSING_DATA_CLASSIFICATION | Spec L1 sin clasificacion de datos | 422 Unprocessable |
| INVALID_VERSION | Version no cumple formato semver | 422 Unprocessable |
| LLM_UNAVAILABLE | LLM no disponible (circuit breaker abierto o timeout) | 503 Service Unavailable |
| LLM_TIMEOUT | Generacion excedio timeout de 120s | 504 Gateway Timeout |
| DLP_VIOLATION | DLP post-response detecto PII en la respuesta del LLM | 500 Internal Error |
| UNAUTHORIZED | Usuario no tiene permisos para crear specs | 403 Forbidden |
| SECTION_NOT_FOUND | Seccion a regenerar no existe en la spec | 400 Bad Request |

---

## Requisitos de ejecucion

| Aspecto | Requisito |
|---------|----------|
| **Latencia — L1** | Generacion IA: < 120s (p95). Opus para contexto complejo |
| **Latencia — L2/L3** | Generacion IA: < 60s (p95). Sonnet para tarea acotada |
| **Latencia — CRUD** | Guardar/editar spec: < 500ms (p95) |
| **Latencia — Regenerar seccion** | < 30s (p95). Sonnet |
| **Asincronia** | Toda generacion IA es asincrona via Azure Service Bus. Frontend recibe resultado por WebSocket o polling |
| **Circuit breaker** | Si LLM falla 3 veces consecutivas, pausar 60s y mostrar mensaje al usuario |
| **Reintentos** | Max 3 reintentos con backoff exponencial (1s, 2s, 4s) |
| **Idempotencia** | Guardar spec es idempotente por specId (upsert) |
| **Observabilidad** | `spec_created_total`, `llm_generation_duration_ms`, `llm_generation_total` (por modelo), `llm_tokens_consumed_total` |
| **Autorizacion** | ARCHITECT, LEAD y SENIOR_DEV pueden crear specs |
| **Versionado** | Version 1.0.0 al crear. Incremento automatico segun reglas semver al actualizar |

---

## Proteccion de datos

| Campo / Flujo | Clasificacion | Tratamiento |
|---------------|---------------|-------------|
| Input del usuario (descripcion) | Confidencial | DLP pre-prompt: eliminar datos Sensibles innecesarios antes de enviar al LLM |
| ADRs en contexto | Confidencial | Incluir solo ADRs relevantes, no todos |
| Respuesta del LLM | Confidencial | DLP post-response: validar que no contenga PII generada o inferida |
| Spec content | Confidencial | Propiedad intelectual, almacenar cifrado en reposo |
| dataClassification | Interno | Metadata de gobernanza |
| AuditLog de invocacion | Interno | Registrar: modelo, tokens, timestamp. NO registrar input completo (solo sanitizado) |

**Regla DLP**: el `DLPFilter` opera en 2 fases:
1. **Pre-prompt**: antes de enviar al LLM, remueve/reemplaza datos clasificados como Sensible
2. **Post-response**: valida que la respuesta no contenga patrones de PII (emails, documentos, telefonos)

---

## Escenarios de test (GWT)

### GWT-01: Generar spec L1 exitosamente

```
Given un proyecto existente con ADRs configurados
  And un usuario con rol SENIOR_DEV
  And el LLM (Opus) esta disponible
When solicita generar L1 con descripcion "Sistema de reservas de habitaciones con validacion de disponibilidad y precios"
Then el sistema:
  1. Arma contexto: input + template L1 + ADRs vigentes + estandares
  2. Sanitiza el contexto (DLP pre-prompt)
  3. Encola job en Azure Service Bus
  4. Retorna 202 Accepted con jobId
  5. El job genera borrador con: entidades, reglas, casos de uso, clasificacion de datos
  6. Notifica via WebSocket al usuario
  7. Registra invocacion en AuditLog (modelo: Opus, tokens: N)
```

### GWT-02: Generar spec L2 desde L1 aprobada

```
Given una spec L1 en estado APPROVED (SPEC-L1-booking v1.0.0)
  And un usuario con rol SENIOR_DEV
When solicita derivar L2 para el caso de uso CU-01 (CreateBooking)
  And proporciona contexto tecnico adicional
Then el sistema:
  1. Extrae entidades y reglas de la L1
  2. Arma contexto: L1 completa + CU seleccionado + template L2 + ADRs
  3. Genera borrador L2 con: contrato tecnico, dependencias, errores, proteccion de datos
  4. Asigna parentSpecId = L1.specId
  5. Asigna version 1.0.0
```

### GWT-03: Rechazar L2 si parent no esta aprobado

```
Given una spec L1 en estado DRAFT
When un usuario intenta derivar una L2 desde esa L1
Then el sistema retorna 422 con error PARENT_NOT_APPROVED
```

### GWT-04: Generar spec L3 con advertencia de tamanio

```
Given una spec L2 en estado APPROVED
When un usuario describe un cambio que el agente estima > 400 lineas de codigo
Then el sistema genera el borrador L3 normalmente
  And incluye una advertencia: "El cambio estimado supera 400 lineas. Considere dividir en multiples L3"
```

### GWT-05: Manejar LLM no disponible

```
Given el circuit breaker del LLM esta abierto (3 fallos consecutivos)
When un usuario solicita generar una spec con IA
Then el sistema retorna 503 con error LLM_UNAVAILABLE
  And muestra mensaje: "El servicio de IA no esta disponible. Puede crear la spec manualmente."
  And el portal sigue operativo para CRUD manual de specs
```

### GWT-06: Regenerar seccion individual

```
Given una spec L1 en estado DRAFT con contenido generado
  And un usuario con permisos de edicion
When solicita regenerar solo la seccion "casosDeUso" con instruccion adicional "Agregar caso de uso para cancelacion"
Then el sistema envia al LLM solo la seccion con el contexto de la spec
  And reemplaza la seccion en el borrador
  And mantiene el resto de secciones intactas
```

### GWT-07: DLP detecta PII en respuesta del LLM

```
Given una generacion de spec en curso
When la respuesta del LLM contiene un email real (ej: juan@empresa.com)
Then el DLP post-response detecta el patron
  And reemplaza con dato sintetico (ej: user@example.com)
  And registra el hallazgo en metricas (pii_filtered_total)
  And retorna la spec sanitizada al usuario
```

### GWT-08: Guardar spec manualmente sin IA

```
Given un usuario con rol SENIOR_DEV
When crea una spec L1 manualmente completando todos los campos del formulario
  And incluye clasificacion de datos por entidad
  And guarda como DRAFT
Then el sistema persiste la spec con version 1.0.0
  And valida estructura basica (campos obligatorios)
  And registra en AuditLog
```

---

## Referencias

- SPEC-L1-spec-agent-portal v1.0.0 — CU-02
- RF-SPEC-01 a RF-SPEC-10
- RF-AGENT-01 a RF-AGENT-09
- RF-DLP-01, RF-DLP-02
- RNF-PERF-01, RNF-PERF-02, RNF-PERF-07
- RNF-AVAIL-02 a RNF-AVAIL-04
- ADR-006 (Data Classification & DLP)
