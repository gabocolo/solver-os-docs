# Prompt 05 — Implementar Metrics Service

| Campo | Valor |
|-------|-------|
| **Spec** | SPEC-L2-metrics-dashboard v1.0.0 |
| **Skill** | SK-001 — Generacion de codigo de caso de uso |
| **Modelo recomendado** | Sonnet |

---

## Prompt

```
## Rol
Actua como un desarrollador senior con experiencia en .NET 10, C#, Redis y arquitectura hexagonal.

## Invocacion de especificacion
Aplica la especificacion SPEC-L2-metrics-dashboard version 1.0.0.
[ADJUNTAR: contenido completo de SPEC-L2-metrics-dashboard.md]

## Tarea
Implementa el caso de uso QueryMetricsAndTraceability segun la spec. Esto incluye:
1. Value objects: MetricsDashboardResponse, SpecStatusMetric, SkillUsageMetric, SkillReworkMetric, TraceabilityNode, ExportResponse
2. Puertos: ISpecRepository, IReviewCycleRepository, IPromptRepository, ISkillCatalogRepository, IAuditLogRepository, ICachePort
3. Servicio de aplicacion MetricsService: GetMetrics, GetTraceabilityTree, ExportMetrics
4. Cache en Redis con TTL 5 minutos e invalidacion por evento SpecStatusChanged
5. Background job cada 15 minutos para metricas pesadas
6. Endpoints Minimal API

## Entrada
- GetMetrics: projectId, period (week/sprint/month/custom), startDate?, endDate?, level?, team?
- GetTraceabilityTree: projectId, rootSpecId? (si null, todas las L1)
- ExportMetrics: projectId, format (csv/json), period?, metrics[]?

## Salida esperada
- MetricsDashboardResponse: specsByStatus, dataClassificationCoverage, dataProtectionCoverage, avgCreationTime, avgReviewCycleTime, avgReviewCycles, skillUsage[], skillRework[], calculatedAt
- TraceabilityTreeResponse: projectId, trees (TraceabilityNode[] recursivo)
- ExportResponse: fileName, contentType, data (bytes)

## Reglas
- Todas las reglas provienen EXCLUSIVAMENTE de la spec.
- Dependencias: SpecRepository, ReviewCycleRepository, PromptRepository, SkillCatalogRepository, AuditLogRepository, CachePort (Redis).
- Errores: PROJECT_NOT_FOUND (404), INVALID_PERIOD (400), EXPORT_TOO_LARGE (413), UNAUTHORIZED (403).
- Cache Redis con TTL 5 min. Cache hit < 200ms. Cache miss < 3s.
- Invalidar cache al cambiar estado de spec.
- Export limite: 10,000 registros max.
- Alerta de skill: si reworkPercentage > 20% por 2 sprints consecutivos → alert = true.
- Todos los roles pueden ver metricas.

## Contexto de gobernanza adjunto
- ADR-001: Ports & Adapters
- ADR-003: Trazabilidad

## Restricciones DLP (ADR-006)
- No incluyas datos reales (PII) en el codigo, tests o fixtures.
- Metricas agregadas son Interno — no contienen PII.
- Export CSV/JSON es Confidencial — metricas del proyecto son propiedad intelectual.

## Restricciones de salida
- Dividir en PRs: 1) Value objects + puertos, 2) MetricsService + cache, 3) Trazabilidad + export, 4) Endpoints
- Tests basados en GWT-01 a GWT-06 de la spec
- Incluir tests de cache hit y cache miss
```

---

## Contexto a adjuntar

| Documento | Seccion |
|-----------|---------|
| SPEC-L2-metrics-dashboard.md | Completo |
| ADR-001 | Ports & Adapters |
| ADR-003 | Trazabilidad |
