# Prompt 05-FE — Implementar Modulo Metrics Dashboard (Frontend Angular)

| Campo | Valor |
|-------|-------|
| **Spec** | SPEC-L2-metrics-dashboard v1.0.0 |
| **Skill** | SK-001 — Generacion de codigo de caso de uso |
| **Modelo recomendado** | Sonnet |
| **Pareo backend** | PROMPT-05 (Implementar Metrics Service) |

---

## Prompt

```
## Rol
Actua como un desarrollador senior Angular con experiencia en Angular 21, TypeScript strict, Angular Material y Tailwind CSS.

## Invocacion de especificacion
Aplica la especificacion SPEC-L2-metrics-dashboard version 1.0.0 + Wireframe Pantalla 6 (Dashboard de metricas).
[ADJUNTAR: contenido completo de SPEC-L2-metrics-dashboard.md]
[ADJUNTAR: seccion Pantalla 6 de 05-UI-WIREFRAMES.md]

## Tarea
Implementa el feature module Metrics segun la spec. Esto incluye:
1. MetricsDashboardComponent: layout principal con grid de cards y graficos
2. SpecStatusChartComponent: grafico de dona/pie con specs por estado (DRAFT, IN_REVIEW, APPROVED, IMPLEMENTED) usando ngx-charts o Chart.js
3. DataClassificationCoverageComponent: barra de progreso con % de specs L1 que tienen clasificacion de datos completa
4. VelocityMetricsComponent: cards con tiempo promedio de creacion de spec, tiempo promedio de aprobacion, specs por sprint
5. SkillUsageChartComponent: grafico de barras con uso de skills y % retrabajo por skill
6. TraceabilityTreeComponent: arbol interactivo L1 → L2 → L3 → prompts → commits con expand/collapse
7. MetricsExportComponent: botones exportar CSV y JSON
8. MetricsService: HttpClient calls a endpoints de PROMPT-05 backend, cache local (5 min) alineado con Redis del backend

## Entrada
- API contract de SPEC-L2-metrics-dashboard: endpoints GetMetrics, GetTraceabilityTree, ExportMetrics
- Interfaces tipadas: MetricsDashboardResponse, SpecStatusMetric, SkillUsageMetric, SkillReworkMetric, TraceabilityNode, ExportResponse

## Salida esperada
Feature module con dashboard, charts, tree, export y tests unitarios.

## Reglas
- Todas las reglas provienen EXCLUSIVAMENTE de la spec.
- Standalone components (no NgModules).
- Angular Signals para estado reactivo.
- Lazy loading del feature module en app routing.
- Skeleton loaders (mat-progress-bar o custom) mientras cargan datos.
- Cache local en servicio con TTL 5 minutos, alineado con cache Redis del backend.
- Graficos responsivos — se adaptan al tamano del contenedor.
- Arbol de trazabilidad usa mat-tree (Angular Material) con expand/collapse.
- Export genera archivo descargable via Blob + URL.createObjectURL.
- Periodo seleccionable: week, sprint, month, custom (date range picker).
- Alerta visual en skills con reworkPercentage > 20% (badge rojo).
- Todos los roles pueden ver metricas (sin restriccion de rol en frontend).
- ARIA labels y keyboard navigation (accesibilidad AA).
- No PII real en codigo/tests — datos sinteticos.

## Contexto de gobernanza adjunto
- ADR-001: Ports & Adapters — frontend consume APIs definidos en backend
- ADR-003: Trazabilidad — el arbol de trazabilidad visualiza la cadena spec → commit → release

## Restricciones DLP (ADR-006)
- Datos sinteticos en fixtures y mocks.
- No hardcodear tokens ni secretos.
- Environment variables para configuracion sensible.
- Metricas agregadas son Interno — no contienen PII.
- Export CSV/JSON es Confidencial — no cachear en localStorage del navegador.

## Restricciones de salida
- Diff < 400 lineas por PR.
- Si el diff es mayor, divide en PRs: 1) Dashboard layout + cards, 2) Charts (spec status + skill usage), 3) Traceability tree, 4) Export + cache.
- Tests unitarios con Jasmine/Karma para MetricsService y componentes.
- Incluir tests de cache hit y cache expiry.
- Cada componente en su propio archivo.
```

---

## Contexto a adjuntar

| Documento | Seccion |
|-----------|---------|
| SPEC-L2-metrics-dashboard.md | Completo |
| 05-UI-WIREFRAMES.md | Pantalla 6 — Dashboard de metricas |
| ADR-001 | Ports & Adapters |
| ADR-003 | Trazabilidad |
| ADR-006 | Data Classification & DLP |
