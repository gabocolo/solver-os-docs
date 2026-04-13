# Especificacion de Sistema (Nivel 2)

| Campo | Valor |
|-------|-------|
| **Spec ID** | SPEC-L2-review-specs |
| **Nivel** | L2 — Sistema |
| **Version** | 1.0.0 |
| **Estado** | DRAFT |
| **Parent** | SPEC-L1-spec-agent-portal v1.0.0 |
| **Caso de uso** | CU-03: Revisar y aprobar spec + CU-07: Validacion automatica |
| **Creado por** | Equipo de Arquitectura |
| **Fecha de creacion** | 2026-04-09 |
| **Aprobado por** | Pendiente |
| **Fecha de aprobacion** | Pendiente |
| **PR** | Pendiente |

---

## Descripcion general

Modulo de revision y aprobacion de especificaciones. Implementa el Gate 1 del framework: validacion automatica de estructura (SK-004) seguida de revision humana con comentarios, decision formal (APPROVED/CHANGES_REQUESTED) y firma digital. Gestiona el ciclo de estados DRAFT → IN_REVIEW → APPROVED con auditoria completa.

---

## Sistema

**Nombre:** Spec Management Service + Governance Service (Validation)

## Caso de uso

**Nombre:** ReviewAndApproveSpec

**Descripcion:** El autor envia una spec de DRAFT a IN_REVIEW. El sistema ejecuta validacion automatica (estructura, clasificacion de datos, ADRs, semver). Si pasa, el revisor asignado recibe notificacion, lee la spec, agrega comentarios por seccion y decide: APPROVED (con firma) o CHANGES_REQUESTED (devuelve a DRAFT). El ciclo completo se registra en AuditLog.

---

## Contrato tecnico

### Input — SubmitForReview

| Campo | Tipo | Requerido | Descripcion |
|-------|------|-----------|-------------|
| specId | UUID | Si | Spec a enviar a revision |
| reviewerId | UUID | Si | Revisor asignado (debe ser ARCHITECT o LEAD, diferente al autor) |

### Input — AddComment

| Campo | Tipo | Requerido | Descripcion |
|-------|------|-----------|-------------|
| reviewId | UUID | Si | Ciclo de revision activo |
| section | string | Si | Seccion de la spec a comentar (ej: "contrato", "reglas", "errores") |
| comment | string | Si | Texto del comentario |

### Input — SubmitDecision

| Campo | Tipo | Requerido | Descripcion |
|-------|------|-----------|-------------|
| reviewId | UUID | Si | Ciclo de revision activo |
| decision | ReviewDecision | Si | APPROVED o CHANGES_REQUESTED |
| summary | string | No | Resumen de la decision |

### Output — ValidationResult

| Campo | Tipo | Descripcion |
|-------|------|-------------|
| passed | boolean | Si la validacion paso (sin hallazgos criticos) |
| findings | ValidationFinding[] | Lista de hallazgos |

### ValidationFinding

| Campo | Tipo | Descripcion |
|-------|------|-------------|
| type | string | CRITICAL (bloquea) o WARNING (no bloquea) |
| field | string | Campo o seccion con el hallazgo |
| message | string | Descripcion del problema |
| rule | string | Regla de validacion que fallo (ej: MISSING_DATA_CLASSIFICATION) |

### Output — ReviewCycleResponse

| Campo | Tipo | Descripcion |
|-------|------|-------------|
| reviewId | UUID | ID del ciclo |
| specId | UUID | Spec bajo revision |
| reviewerId | UUID | Revisor |
| status | ReviewStatus | Estado actual |
| comments | Comment[] | Lista de comentarios |
| decision | ReviewDecision | Decision (si completado) |
| validationResult | ValidationResult | Resultado de validacion automatica |
| createdAt | datetime | Inicio del ciclo |
| completedAt | datetime | Fin del ciclo (si completado) |

### Output — SpecStatusResponse

| Campo | Tipo | Descripcion |
|-------|------|-------------|
| specId | UUID | Spec |
| previousStatus | SpecStatus | Estado anterior |
| newStatus | SpecStatus | Nuevo estado |
| approvedBy | UUID | Revisor (si APPROVED) |
| approvedAt | datetime | Fecha de aprobacion |
| reviewCycleCount | integer | Numero de ciclos de revision totales |

---

## Dependencias permitidas

| Dependencia | Tipo | Proposito |
|-------------|------|----------|
| SpecRepository | Puerto (persistencia) | Leer y actualizar estado de specs |
| ReviewCycleRepository | Puerto (persistencia) | CRUD de ciclos de revision |
| SpecValidator | Puerto (validacion) | Ejecutar validacion automatica (SK-004) |
| NotificationService | Puerto (notificacion) | Notificar al revisor cuando hay spec para revisar |
| EventPublisher | Puerto (eventos) | Publicar SpecApproved, SpecChangesRequested |
| AuditLogRepository | Puerto (auditoria) | Registrar decisiones, firmas, cambios de estado |
| DevOpsWorkItemService | Puerto (integracion) | Crear work items en Azure DevOps al aprobar spec (via evento SpecApproved) |

---

## Errores

| Codigo | Descripcion | HTTP Status |
|--------|-------------|-------------|
| SPEC_NOT_FOUND | La spec no existe | 404 Not Found |
| SPEC_NOT_IN_DRAFT | Solo specs en DRAFT pueden enviarse a revision | 422 Unprocessable |
| SPEC_NOT_IN_REVIEW | La spec no esta en IN_REVIEW para ser aprobada/rechazada | 422 Unprocessable |
| SELF_REVIEW_NOT_ALLOWED | El revisor no puede ser el mismo que el autor | 422 Unprocessable |
| REVIEWER_NOT_AUTHORIZED | El revisor debe tener rol ARCHITECT o LEAD | 403 Forbidden |
| VALIDATION_FAILED | La validacion automatica tiene hallazgos criticos que bloquean | 422 Unprocessable |
| REVIEW_NOT_FOUND | El ciclo de revision no existe | 404 Not Found |
| REVIEW_ALREADY_COMPLETED | El ciclo de revision ya fue completado | 422 Unprocessable |
| DEVOPS_SYNC_FAILED | No se pudo crear el work item en Azure DevOps (no bloquea aprobacion) | 200 OK (warning) |

---

## Requisitos de ejecucion

| Aspecto | Requisito |
|---------|----------|
| **Latencia — validacion** | < 10s (p95) para validacion automatica completa |
| **Latencia — cambio de estado** | < 500ms (p95) |
| **Latencia — notificacion** | < 500ms (p95) |
| **Observabilidad** | `spec_submitted_for_review_total`, `spec_approved_total`, `spec_changes_requested_total`, `validation_duration_ms`, `review_cycle_duration_hours` |
| **Autorizacion** | Enviar a revision: ARCHITECT, LEAD, SENIOR_DEV. Aprobar: solo ARCHITECT y LEAD |
| **Integridad** | Cambio de estado atomico: si falla el registro en AuditLog, revertir cambio de estado |

---

## Validacion automatica (SK-004) — Detalle

La validacion se ejecuta al transicionar DRAFT → IN_REVIEW:

| Validacion | Tipo | Aplica a | Regla |
|-----------|------|----------|-------|
| Campos obligatorios | CRITICAL | L1/L2/L3 | Todos los campos requeridos segun nivel deben estar presentes |
| Semver valido | CRITICAL | L1/L2/L3 | Version cumple formato X.Y.Z y es mayor que version anterior |
| Clasificacion de datos | CRITICAL | L1 | dataClassification presente con al menos 1 entidad clasificada |
| Proteccion de datos | CRITICAL | L2 | Si la L1 parent tiene campos Sensible, la L2 debe tener seccion de proteccion |
| ADRs referenciados | WARNING | L1/L2 | Debe referenciar ADR-001 si usa ports/adapters, ADR-006 si tiene PII |
| Spec no duplicada | CRITICAL | L1/L2/L3 | No existe otra spec activa con mismo titulo y nivel |
| Parent aprobado | CRITICAL | L2/L3 | El parent esta en estado APPROVED |
| Escenarios de test | WARNING | L2 | Al menos 3 escenarios GWT definidos |

---

## Proteccion de datos

| Campo / Flujo | Clasificacion | Tratamiento |
|---------------|---------------|-------------|
| ReviewCycle.comments | Interno | Pueden contener referencias a reglas de negocio — no exponer externamente |
| Firma de aprobacion (reviewer + timestamp) | Interno | Registro inmutable en AuditLog |
| Notificaciones | Interno | No incluir contenido de la spec en la notificacion, solo referencia |

---

## Escenarios de test (GWT)

### GWT-01: Enviar a revision exitosamente

```
Given una spec L1 en estado DRAFT con estructura completa y clasificacion de datos
  And un revisor valido con rol ARCHITECT (diferente al autor)
When el autor envia la spec a revision
Then el sistema:
  1. Ejecuta validacion automatica (SK-004) — pasa
  2. Cambia estado a IN_REVIEW
  3. Crea un ReviewCycle con status PENDING
  4. Notifica al revisor
  5. Registra en AuditLog: SPEC_SUBMITTED_FOR_REVIEW
```

### GWT-02: Bloquear revision por validacion fallida

```
Given una spec L1 en estado DRAFT sin clasificacion de datos
When el autor intenta enviar a revision
Then la validacion detecta MISSING_DATA_CLASSIFICATION (CRITICAL)
  And retorna 422 con ValidationResult.passed = false
  And la spec permanece en DRAFT
```

### GWT-03: Aprobar spec

```
Given una spec L2 en estado IN_REVIEW con review cycle activo
  And un revisor con rol ARCHITECT
When el revisor decide APPROVED con resumen "Contrato tecnico correcto"
Then el sistema:
  1. Cambia estado de la spec a APPROVED
  2. Registra approvedBy y approvedAt
  3. Completa el ReviewCycle con decision APPROVED
  4. Registra en AuditLog: SPEC_APPROVED (firma: revisor + timestamp)
  5. Publica evento SpecApproved
  6. El evento SpecApproved dispara la sincronizacion con Azure DevOps (ver SPEC-L2-sync-devops)
```

### GWT-04: Solicitar cambios

```
Given una spec en estado IN_REVIEW
When el revisor agrega 2 comentarios en secciones "contrato" y "errores"
  And decide CHANGES_REQUESTED con resumen "Falta definir error de timeout"
Then el sistema:
  1. Cambia estado a DRAFT (CHANGES_REQUESTED)
  2. Completa el ReviewCycle con decision CHANGES_REQUESTED
  3. Los comentarios quedan asociados al review cycle
  4. Notifica al autor con los comentarios
```

### GWT-05: Rechazar auto-revision

```
Given una spec creada por usuario "dev-senior-1"
When "dev-senior-1" intenta asignarse como revisor
Then el sistema retorna 422 con error SELF_REVIEW_NOT_ALLOWED
```

### GWT-06: Rechazar revisor sin permisos

```
Given una spec en IN_REVIEW
When un usuario con rol QA intenta aprobar
Then el sistema retorna 403 con error REVIEWER_NOT_AUTHORIZED
```

### GWT-07: Registrar multiples ciclos de revision

```
Given una spec que fue rechazada 2 veces (CHANGES_REQUESTED)
  And el autor corrigio y envio a revision por tercera vez
When el revisor aprueba en el tercer ciclo
Then la spec queda con reviewCycleCount = 3
  And los 3 ciclos completos estan registrados con comentarios, decisiones y tiempos
```

### GWT-08: Sincronizar con Azure DevOps al aprobar

```
Given una spec L2 aprobada exitosamente
  And el proyecto tiene configurado devOpsProjectUrl
When el evento SpecApproved se publica
Then el sistema crea un User Story en Azure DevOps con:
  - Titulo: "[SPEC-L2] CreateBooking v1.0.0"
  - Descripcion con link a la spec en el portal
  - Criterios de aceptacion extraidos de los escenarios GWT
  - Tags: spec-agent, L2, v1.0.0
  And registra la sincronizacion en WorkItemSync
  And muestra el link al work item en la vista de la spec
```

---

## Referencias

- SPEC-L1-spec-agent-portal v1.1.0 — CU-03, CU-07, CU-08
- RF-REVIEW-01 a RF-REVIEW-10
- RF-VALID-01 a RF-VALID-06
- RNF-PERF-03 (validacion < 10s)
- RNF-SEC-02 (autorizacion por roles)
