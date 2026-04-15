# /observability — Logs, Metricas y Trazas

Configuracion y definiciones de observabilidad del proyecto (3 pilares + alertamiento).

## Estructura

| Carpeta | Pilar | Proposito |
|---------|-------|-----------|
| logs/ | Logging | Configuracion de logging estructurado, filtros PII |
| metrics/ | Metricas | Definiciones de metricas custom, dashboards, KPIs |
| tracing/ | Trazas | Configuracion de tracing distribuido, spans |
| alerts/ | Alertamiento | Reglas de alerta, umbrales, canales de notificacion |

## 3 Pilares

### Logs
- Logging estructurado (JSON)
- correlationId en cada operacion (ADR-003)
- Filtro de PII obligatorio (ADR-006) — campos Sensible se enmascaran
- Niveles: INFO (exitosas), WARN (validaciones fallidas), ERROR (errores inesperados)

### Trazas
- OpenTelemetry como estandar
- Span por operacion principal + spans por cada llamada a dependencia
- Propagacion de contexto entre servicios

### Metricas
- Metricas de negocio: operaciones por minuto, errores por tipo
- Metricas tecnicas: latencia (p50, p95, p99), throughput, error rate
- Metricas del framework: % retrabajo, trazabilidad, cobertura de specs

## Alertas

| Condicion | Severidad | Canal |
|-----------|----------|-------|
| Error rate > 5% en 5 min | CRITICAL | PagerDuty / Slack |
| Latencia p95 > {{umbral}}ms | WARNING | Slack |
| PII detectada en logs | CRITICAL | Seguridad + DPO |

> La observabilidad se define en la spec L2 (seccion "Observabilidad requerida").
