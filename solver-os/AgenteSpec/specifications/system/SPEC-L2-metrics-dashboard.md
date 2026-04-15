# Especificacion de Sistema (Nivel 2)

| Campo | Valor |
|-------|-------|
| **Spec ID** | SPEC-L2-metrics-dashboard |
| **Nivel** | L2 — Sistema |
| **Version** | 1.0.0 |
| **Estado** | DRAFT |
| **Parent** | SPEC-L1-spec-agent-portal v1.0.0 |
| **Caso de uso** | CU-05: Dashboard de metricas y trazabilidad |
| **Creado por** | Equipo de Arquitectura |
| **Fecha de creacion** | 2026-04-09 |
| **Aprobado por** | Pendiente |
| **Fecha de aprobacion** | Pendiente |
| **PR** | Pendiente |

---

## Descripcion general

Modulo de metricas y trazabilidad del Spec Agent Portal. Calcula y muestra metricas del framework en tiempo real: specs por estado, cobertura de clasificacion de datos, tiempos de aprobacion, uso de skills y arbol de trazabilidad L1→L2→L3. Soporta filtros por periodo, equipo y nivel. Permite exportar datos en CSV y JSON.

---

## Sistema

**Nombre:** Metrics Service

## Caso de uso

**Nombre:** QueryMetricsAndTraceability

**Descripcion:** El arquitecto o lider tecnico abre el dashboard de metricas, selecciona un proyecto y un periodo. El portal calcula y muestra metricas agregadas (con cache en Redis). El usuario puede hacer drill-down en metricas especificas y exportar los datos.

---

## Contrato tecnico

### Input — GetMetrics

| Campo | Tipo | Requerido | Descripcion |
|-------|------|-----------|-------------|
| projectId | UUID | Si | Proyecto a consultar |
| period | string | No | Periodo: `week`, `sprint`, `month`, `custom`. Default: `sprint` |
| startDate | date | Condicional | Requerido si period = `custom` |
| endDate | date | Condicional | Requerido si period = `custom` |
| level | SpecLevel | No | Filtrar por nivel (L1, L2, L3) |
| team | string | No | Filtrar por equipo (v2) |

### Input — GetTraceabilityTree

| Campo | Tipo | Requerido | Descripcion |
|-------|------|-----------|-------------|
| projectId | UUID | Si | Proyecto |
| rootSpecId | UUID | No | Spec raiz (L1) para mostrar arbol desde ahi. Si null, muestra todas las L1 |

### Input — ExportMetrics

| Campo | Tipo | Requerido | Descripcion |
|-------|------|-----------|-------------|
| projectId | UUID | Si | Proyecto |
| format | string | Si | `csv` o `json` |
| period | string | No | Mismo que GetMetrics |
| metrics | string[] | No | Metricas especificas a exportar. Si null, exporta todas |

### Output — MetricsDashboardResponse

| Campo | Tipo | Descripcion |
|-------|------|-------------|
| projectId | UUID | Proyecto |
| period | string | Periodo consultado |
| specsByStatus | SpecStatusMetric | Specs por estado |
| dataClassificationCoverage | PercentageMetric | % de L1 con clasificacion completa |
| dataProtectionCoverage | PercentageMetric | % de L2 con seccion de proteccion de datos |
| avgCreationTime | DurationMetric | Tiempo promedio desde DRAFT hasta APPROVED |
| avgReviewCycleTime | DurationMetric | Tiempo promedio de ciclo de revision |
| avgReviewCycles | NumberMetric | Ciclos de revision promedio antes de aprobacion |
| skillUsage | SkillUsageMetric[] | Frecuencia de uso por skill |
| skillRework | SkillReworkMetric[] | Retrabajo por skill |
| calculatedAt | datetime | Fecha de calculo |

### SpecStatusMetric

| Campo | Tipo | Descripcion |
|-------|------|-------------|
| draft | integer | Specs en DRAFT |
| inReview | integer | Specs en IN_REVIEW |
| approved | integer | Specs en APPROVED |
| changesRequested | integer | Specs con cambios solicitados |
| deprecated | integer | Specs deprecadas |
| total | integer | Total de specs |

### SkillUsageMetric

| Campo | Tipo | Descripcion |
|-------|------|-------------|
| skillId | UUID | Skill |
| skillName | string | Nombre |
| invocations | integer | Veces invocado en el periodo |
| uniqueUsers | integer | Usuarios unicos que lo usaron |

### SkillReworkMetric

| Campo | Tipo | Descripcion |
|-------|------|-------------|
| skillId | UUID | Skill |
| skillName | string | Nombre |
| reworkPercentage | decimal | % de retrabajo |
| alert | boolean | True si > 20% sostenido por 2 sprints |

### Output — TraceabilityTreeResponse

| Campo | Tipo | Descripcion |
|-------|------|-------------|
| projectId | UUID | Proyecto |
| trees | TraceabilityNode[] | Lista de arboles (uno por L1) |

### TraceabilityNode

| Campo | Tipo | Descripcion |
|-------|------|-------------|
| specId | UUID | ID de la spec |
| title | string | Titulo |
| level | SpecLevel | Nivel |
| version | string | Version |
| status | SpecStatus | Estado actual |
| promptCount | integer | Prompts generados sobre esta spec |
| children | TraceabilityNode[] | Specs hijas (L2s de L1, L3s de L2) |

### Output — ExportResponse

| Campo | Tipo | Descripcion |
|-------|------|-------------|
| fileName | string | Nombre del archivo |
| contentType | string | `text/csv` o `application/json` |
| data | byte[] | Contenido del archivo |

---

## Dependencias permitidas

| Dependencia | Tipo | Proposito |
|-------------|------|----------|
| SpecRepository | Puerto (persistencia) | Consultar specs para calculo de metricas |
| ReviewCycleRepository | Puerto (persistencia) | Consultar ciclos de revision para tiempos |
| PromptRepository | Puerto (persistencia) | Consultar prompts para uso de skills |
| SkillCatalogRepository | Puerto (persistencia) | Obtener info de skills |
| AuditLogRepository | Puerto (persistencia) | Datos para trazabilidad |
| CachePort | Puerto (cache) | Redis: cache de metricas calculadas (TTL 5 min) |

---

## Errores

| Codigo | Descripcion | HTTP Status |
|--------|-------------|-------------|
| PROJECT_NOT_FOUND | El proyecto no existe | 404 Not Found |
| INVALID_PERIOD | Periodo no reconocido o rango de fechas invalido | 400 Bad Request |
| EXPORT_TOO_LARGE | El export excede el limite de registros (10,000) | 413 Payload Too Large |
| UNAUTHORIZED | Usuario no tiene permisos para ver metricas | 403 Forbidden |

---

## Requisitos de ejecucion

| Aspecto | Requisito |
|---------|----------|
| **Latencia — dashboard** | < 3s (p95) para carga completa del dashboard |
| **Latencia — cache hit** | < 200ms |
| **Latencia — export** | < 5s para datasets < 10,000 registros |
| **Cache** | Redis con TTL 5 minutos. Invalidar al cambiar estado de spec |
| **Calculo periodico** | Background job cada 15 minutos para metricas pesadas (historial, tendencias) |
| **Observabilidad** | `metrics_calculation_duration_ms`, `metrics_cache_hit_total`, `metrics_cache_miss_total`, `metrics_export_total` |
| **Autorizacion** | Todos los roles pueden ver metricas (ARCHITECT, LEAD, SENIOR_DEV, QA, PO) |

---

## Proteccion de datos

| Campo / Flujo | Clasificacion | Tratamiento |
|---------------|---------------|-------------|
| Metricas agregadas | Interno | No contienen PII — son datos numericos y porcentajes |
| Trazabilidad | Interno | Muestra IDs y titulos de specs — no contenido sensible |
| Export CSV/JSON | Confidencial | Metricas del proyecto son confidenciales. No exponer externamente |
| Drill-down | Interno | El detalle de una metrica puede mostrar specIds — no PII |

---

## Escenarios de test (GWT)

### GWT-01: Cargar dashboard exitosamente

```
Given un proyecto con 12 specs (5 DRAFT, 3 IN_REVIEW, 4 APPROVED)
  And metricas no estan en cache
When un usuario con rol ARCHITECT carga el dashboard
Then el sistema:
  1. Calcula specs por estado: {draft: 5, inReview: 3, approved: 4, total: 12}
  2. Calcula % clasificacion datos y % proteccion datos
  3. Calcula tiempos promedios
  4. Cachea resultado en Redis (TTL 5 min)
  5. Retorna MetricsDashboardResponse en < 3s
```

### GWT-02: Cache hit

```
Given metricas calculadas hace 2 minutos (dentro del TTL de 5 min)
When un usuario carga el dashboard del mismo proyecto
Then el sistema retorna metricas desde cache en < 200ms
```

### GWT-03: Arbol de trazabilidad

```
Given un proyecto con:
  - L1: Booking (APPROVED)
    - L2: CreateBooking (APPROVED, 3 prompts)
      - L3: AddCoupon (DRAFT)
    - L2: CancelBooking (IN_REVIEW)
When un usuario solicita el arbol de trazabilidad
Then el sistema retorna:
  TraceabilityNode{
    title: "Booking", level: L1, status: APPROVED,
    children: [
      {title: "CreateBooking", level: L2, promptCount: 3, children: [
        {title: "AddCoupon", level: L3, status: DRAFT}
      ]},
      {title: "CancelBooking", level: L2, status: IN_REVIEW}
    ]
  }
```

### GWT-04: Alerta de retrabajo en skill

```
Given SK-003 (Generacion contrato API) con 25% retrabajo en sprint actual
  And 22% retrabajo en sprint anterior (sostenido > 20% por 2 sprints)
When se calculan metricas de skills
Then el sistema marca SK-003 con alert = true
  And el dashboard muestra indicador de alerta para ese skill
```

### GWT-05: Exportar metricas en CSV

```
Given metricas calculadas para un proyecto
When un usuario solicita exportar en formato CSV
Then el sistema genera archivo CSV con headers y datos
  And retorna con contentType "text/csv"
  And el nombre incluye proyecto y fecha: "metrics_account-planning_2026-04-09.csv"
```

### GWT-06: Filtrar por periodo

```
Given un proyecto con specs creadas en multiples sprints
When un usuario filtra metricas por period = "month" y startDate = "2026-03-01"
Then el sistema calcula metricas solo para specs creadas/modificadas en ese mes
  And las metricas agregadas reflejan solo ese periodo
```

---

## Referencias

- SPEC-L1-spec-agent-portal v1.0.0 — CU-05
- RF-METRIC-01 a RF-METRIC-09
- RNF-PERF-04 (dashboard < 3s)
- RNF-OBS-06 (metricas de costos LLM)
