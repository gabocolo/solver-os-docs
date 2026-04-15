# Especificacion de Sistema (Nivel 2)

| Campo | Valor |
|-------|-------|
| **Spec ID** | SPEC-L2-manage-skills |
| **Nivel** | L2 — Sistema |
| **Version** | 1.0.0 |
| **Estado** | DRAFT |
| **Parent** | SPEC-L1-spec-agent-portal v1.0.0 |
| **Caso de uso** | CU-06: Gestionar catalogo de skills |
| **Creado por** | Equipo de Arquitectura |
| **Fecha de creacion** | 2026-04-09 |
| **Aprobado por** | Pendiente |
| **Fecha de aprobacion** | Pendiente |
| **PR** | Pendiente |

---

## Descripcion general

Modulo de gestion del catalogo de skills del Spec Agent Portal. Permite al arquitecto registrar, versionar, deprecar y medir skills. Los skills estandarizan COMO ejecutar una clase de tarea con la IA, y estan asociados a prompts para trazabilidad. El sistema calcula metricas por skill (frecuencia, retrabajo) y alerta cuando un skill tiene bajo rendimiento sostenido.

---

## Sistema

**Nombre:** Skill Catalog Service + Metrics Service (skill metrics)

## Caso de uso

**Nombre:** ManageSkillCatalog

**Descripcion:** El arquitecto registra un skill con nombre, tipo (Build/Workflow), version, owner, fase del framework y criterios de uso. El portal asocia el skill a los prompts que lo invocan para trazabilidad. El sistema calcula metricas por skill y alerta si un skill tiene > 20% retrabajo sostenido por 2 sprints. El arquitecto puede deprecar un skill, previniendo su uso en nuevos prompts.

---

## Contrato tecnico

### Input — CreateSkill

| Campo | Tipo | Requerido | Descripcion |
|-------|------|-----------|-------------|
| name | string | Si | Nombre unico del skill (ej: "SK-008 — Generacion de migrations") |
| type | SkillType | Si | BUILD o WORKFLOW |
| version | string | Si | Version semver (ej: "1.0.0") |
| owner | string | Si | Owner tecnico (ej: "Arquitecto", "Lead Backend") |
| phase | string | Si | Fase del framework (ej: "Fase 3 — Generacion") |
| criteria | string | No | Criterios de uso: cuando y como invocar este skill |
| projectId | UUID | No | Proyecto asociado (null = skill global) |

### Input — UpdateSkill

| Campo | Tipo | Requerido | Descripcion |
|-------|------|-----------|-------------|
| skillId | UUID | Si | Skill a actualizar |
| version | string | Si | Nueva version (debe ser mayor que la anterior) |
| criteria | string | No | Criterios de uso actualizados |
| owner | string | No | Nuevo owner (si cambia) |

### Input — DeprecateSkill

| Campo | Tipo | Requerido | Descripcion |
|-------|------|-----------|-------------|
| skillId | UUID | Si | Skill a deprecar |
| reason | string | Si | Razon de deprecacion |
| replacedBy | UUID | No | Skill que lo reemplaza (si existe) |

### Input — GetSkillMetrics

| Campo | Tipo | Requerido | Descripcion |
|-------|------|-----------|-------------|
| skillId | UUID | Si | Skill a consultar |
| period | string | No | Periodo: `sprint`, `month`. Default: `sprint` |

### Output — SkillResponse

| Campo | Tipo | Descripcion |
|-------|------|-------------|
| skillId | UUID | ID del skill |
| name | string | Nombre |
| type | SkillType | BUILD o WORKFLOW |
| version | string | Version actual |
| owner | string | Owner tecnico |
| phase | string | Fase del framework |
| criteria | string | Criterios de uso |
| status | SkillStatus | ACTIVE o DEPRECATED |
| projectId | UUID | Proyecto (null si global) |
| createdAt | datetime | Fecha de creacion |
| updatedAt | datetime | Ultima actualizacion |

### Output — SkillListResponse

| Campo | Tipo | Descripcion |
|-------|------|-------------|
| skills | SkillResponse[] | Lista de skills |
| total | integer | Total de skills |
| active | integer | Skills activos |
| deprecated | integer | Skills deprecados |

### Output — SkillMetricsResponse

| Campo | Tipo | Descripcion |
|-------|------|-------------|
| skillId | UUID | Skill |
| skillName | string | Nombre |
| period | string | Periodo consultado |
| invocations | integer | Veces invocado en prompts |
| uniqueUsers | integer | Usuarios unicos |
| reworkPercentage | decimal | % de retrabajo asociado |
| avgReviewTime | decimal | Tiempo promedio de revision (horas) |
| securityFindings | integer | Hallazgos de seguridad asociados |
| alert | boolean | True si retrabajo > 20% por 2 sprints |
| alertMessage | string | Mensaje de alerta (si aplica) |
| history | SkillMetricHistoryEntry[] | Historial por sprint (ultimos 4) |

### SkillMetricHistoryEntry

| Campo | Tipo | Descripcion |
|-------|------|-------------|
| period | string | Sprint o mes |
| invocations | integer | Invocaciones |
| reworkPercentage | decimal | % retrabajo |

---

## Dependencias permitidas

| Dependencia | Tipo | Proposito |
|-------------|------|----------|
| SkillCatalogRepository | Puerto (persistencia) | CRUD de skills |
| PromptRepository | Puerto (persistencia) | Consultar prompts asociados a skills |
| MetricsService | Puerto (metricas) | Calcular metricas de retrabajo y uso |
| AuditLogRepository | Puerto (auditoria) | Registrar creacion, actualizacion y deprecacion |
| EventPublisher | Puerto (eventos) | Publicar SkillDeprecated para notificar |

---

## Errores

| Codigo | Descripcion | HTTP Status |
|--------|-------------|-------------|
| SKILL_NOT_FOUND | El skill no existe | 404 Not Found |
| SKILL_NAME_DUPLICATE | Ya existe un skill con ese nombre y version | 409 Conflict |
| SKILL_ALREADY_DEPRECATED | El skill ya esta deprecado | 422 Unprocessable |
| INVALID_VERSION | Version no cumple semver o no es mayor que la actual | 422 Unprocessable |
| UNAUTHORIZED | Usuario no tiene rol ARCHITECT para gestionar skills | 403 Forbidden |

---

## Requisitos de ejecucion

| Aspecto | Requisito |
|---------|----------|
| **Latencia — CRUD** | < 500ms (p95) |
| **Latencia — metricas** | < 2s (p95) para calculo de metricas por skill |
| **Observabilidad** | `skill_created_total`, `skill_deprecated_total`, `skill_rework_alert_total` |
| **Autorizacion** | Solo ARCHITECT puede crear, editar y deprecar skills. Todos pueden consultar |
| **Alerta** | Si reworkPercentage > 20% por 2 sprints consecutivos → generar alerta al owner |

---

## Proteccion de datos

| Campo / Flujo | Clasificacion | Tratamiento |
|---------------|---------------|-------------|
| Skill.criteria | Interno | Puede contener informacion tecnica interna |
| SkillMetrics | Interno | Datos agregados sin PII |
| Deprecation reason | Interno | Registrar en AuditLog |

---

## Escenarios de test (GWT)

### GWT-01: Registrar skill exitosamente

```
Given un usuario con rol ARCHITECT
When registra un skill con nombre "SK-008 — Generacion de migrations", tipo BUILD, version "1.0.0", fase "Fase 3"
Then el sistema crea el skill con skillId unico
  And status = ACTIVE
  And registra en AuditLog: SKILL_CREATED
```

### GWT-02: Rechazar skill con nombre duplicado

```
Given un skill existente "SK-001" version "1.0.0"
When un arquitecto intenta crear otro skill con el mismo nombre y version
Then el sistema retorna 409 con error SKILL_NAME_DUPLICATE
```

### GWT-03: Actualizar version de skill

```
Given un skill "SK-001" en version "1.0.0"
When el arquitecto actualiza a version "1.1.0" con nuevos criterios de uso
Then el sistema actualiza version y criteria
  And registra en AuditLog: SKILL_UPDATED
```

### GWT-04: Deprecar skill

```
Given un skill "SK-003" activo
When el arquitecto lo depreca con razon "Reemplazado por SK-008" y replacedBy = SK-008.skillId
Then el sistema:
  1. Cambia status a DEPRECATED
  2. Registra razon y reemplazo
  3. Publica evento SkillDeprecated
  4. Registra en AuditLog
  5. Nuevos prompts no pueden usar este skill
```

### GWT-05: Alerta de retrabajo sostenido

```
Given SK-003 con 22% retrabajo en sprint actual (Sprint 15)
  And 25% retrabajo en sprint anterior (Sprint 14)
  And umbral de alerta = 20% sostenido por 2 sprints
When se calculan metricas de skills
Then el sistema:
  1. Marca SK-003 con alert = true
  2. Genera alertMessage: "SK-003 tiene > 20% retrabajo sostenido por 2 sprints. Considere redisenar o eliminar."
  3. Incrementa metrica skill_rework_alert_total
```

### GWT-06: Listar skills con filtros

```
Given un catalogo con 7 skills (5 BUILD, 2 WORKFLOW, 1 DEPRECATED)
When un usuario consulta skills con filtro type=BUILD y status=ACTIVE
Then el sistema retorna 4 skills BUILD activos
  And total=4, active=4, deprecated=0
```

---

## Referencias

- SPEC-L1-spec-agent-portal v1.0.0 — CU-06
- RF-SKILL-01 a RF-SKILL-06
- Framework AI-Driven Engineering v1.1 — Seccion 7 (Skills)
- solver-os/fase-1/skills/SKILLS-CATALOG.md
