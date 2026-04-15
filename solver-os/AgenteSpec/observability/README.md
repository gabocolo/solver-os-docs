# /observability — Observabilidad del Spec Agent Portal

## Stack de observabilidad

| Pilar | Tecnologia | Configuracion |
|-------|-----------|---------------|
| Logs | Serilog + Seq | Structured JSON, correlationId, PII masking |
| Metricas | OpenTelemetry + Prometheus | Custom metrics, dashboards |
| Trazas | OpenTelemetry | Spans por operacion, propagacion de contexto |
| Alertas | Seq + Slack/PagerDuty | Reglas por severidad |

## Metricas custom

Ver [METRICS-DEFINITIONS.md](metrics/METRICS-DEFINITIONS.md) para definiciones completas.

## Alertas

| Condicion | Severidad | Canal | Tiempo de respuesta |
|-----------|----------|-------|-------------------|
| Error rate > 5% en 5 min | CRITICAL | PagerDuty | < 15 min |
| Latencia p95 > 500ms en 5 min | WARNING | Slack | < 30 min |
| LLM circuit breaker abierto | WARNING | Slack | < 15 min |
| PII detectada en logs | CRITICAL | Seguridad + DPO | < 4h |
| DLP violation en prompt/response | CRITICAL | Seguridad | < 1h |

## Filtro PII en logs (ADR-006)

Campos que se enmascaran automaticamente en logs:
- `User.email` → `u***@domain.com`
- `User.userId` → no se loguea en texto plano
- `Spec.content` → solo metadata (specId, level, status)
- `ADR.content` → solo metadata (adrId, number, title)
