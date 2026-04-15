# Definiciones de Metricas — Spec Agent Portal

## Metricas de negocio

| Nombre | Tipo | Labels | Descripcion | Alerta |
|--------|------|--------|------------|--------|
| spec_creation_total | counter | project, level | Total de specs creadas | — |
| spec_creation_duration_ms | histogram | project, level, model | Tiempo de generacion de spec con IA | p95 > 120s (L1), p95 > 60s (L2/L3) |
| spec_approval_total | counter | project, level | Total de specs aprobadas | — |
| review_cycle_duration_ms | histogram | project | Tiempo desde IN_REVIEW hasta decision | p95 > 24h |
| prompt_generation_total | counter | project, skill | Total de prompts generados | — |
| prompt_generation_duration_ms | histogram | project | Tiempo de generacion de prompt | p95 > 2s |

## Metricas de LLM

| Nombre | Tipo | Labels | Descripcion | Alerta |
|--------|------|--------|------------|--------|
| llm_tokens_used_total | counter | model, operation | Total de tokens consumidos | > 100K por hora |
| llm_request_duration_ms | histogram | model, operation | Latencia de llamadas al LLM | p95 > 30s |
| llm_error_total | counter | model, error_type | Errores de LLM | > 3 consecutivos (circuit breaker) |
| llm_circuit_breaker_state | gauge | model | Estado del circuit breaker (0=closed, 1=open) | state = 1 |

## Metricas de DLP

| Nombre | Tipo | Labels | Descripcion | Alerta |
|--------|------|--------|------------|--------|
| dlp_pre_prompt_violations | counter | pattern_type | PII detectada antes de enviar a LLM | cualquier incremento |
| dlp_post_response_violations | counter | pattern_type | PII detectada en respuesta de LLM | cualquier incremento |
| dlp_scan_pr_findings | counter | finding_type | PII/secretos detectados en PRs | > 0 (bloquea merge) |

## Metricas tecnicas

| Nombre | Tipo | Labels | Descripcion | Alerta |
|--------|------|--------|------------|--------|
| http_request_duration_ms | histogram | method, path, status | Latencia de endpoints | p95 > 500ms |
| http_request_total | counter | method, path, status | Total de requests | error rate > 5% |
| service_bus_messages_processed | counter | queue, status | Mensajes procesados de Service Bus | dead_letter > 0 |
| cache_hit_ratio | gauge | cache_name | Ratio de aciertos de cache Redis | < 70% |
