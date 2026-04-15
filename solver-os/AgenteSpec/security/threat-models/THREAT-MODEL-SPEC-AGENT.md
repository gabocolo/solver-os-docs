# Modelo de Amenazas — Spec Agent Portal

## Analisis STRIDE

| Categoria | Amenaza | Impacto | Mitigacion | ADR |
|-----------|---------|---------|-----------|-----|
| **Spoofing** | Usuario se hace pasar por ARCHITECT para aprobar specs | Specs no revisadas llegan a produccion | OAuth 2.0 + JWT con roles, validacion de rol en cada endpoint | — |
| **Tampering** | Modificacion de spec despues de aprobada | Codigo generado no corresponde a spec aprobada | Spec APPROVED es inmutable, cambios requieren nueva version (L3) | ADR-003 |
| **Repudiation** | Usuario niega haber aprobado una spec | Sin trazabilidad de acciones criticas | AuditLog con userId, action, timestamp, correlationId | ADR-003 |
| **Info Disclosure** | PII en prompts enviados a LLM externo | Datos personales expuestos a terceros | DLP pre-prompt: sanitizar antes de enviar a Claude API | ADR-P005 |
| **Info Disclosure** | PII en respuestas de LLM mostradas al usuario | Datos personales en UI | DLP post-response: filtrar antes de entregar al frontend | ADR-P005 |
| **Info Disclosure** | PII en logs de produccion | Fuga silenciosa de datos | Masking automatico en Serilog para campos Sensible | ADR-006 |
| **DoS** | Abuso de generacion de specs con IA (consumo excesivo de tokens) | Costos elevados, degradacion del servicio | Rate limiting por usuario, circuit breaker (3 fallos → pausa 60s) | ADR-P002 |
| **DoS** | Cola de Service Bus saturada | Generacion de specs bloqueada | Dead-letter queue, reintentos con backoff exponencial (max 3) | ADR-P002 |
| **Elevation** | SENIOR_DEV intenta aprobar spec sin rol ARCHITECT/LEAD | Bypass de Gate 1 | Validacion de rol en endpoint de aprobacion, auditoria | — |
| **Tampering** | Prompt injection via descripcion de spec | LLM genera contenido malicioso o fuera de spec | DLP pre-prompt + system prompt con restricciones explicitas | ADR-P005 |

## Superficie de ataque

1. **Frontend → Backend**: autenticacion JWT, CORS restrictivo, rate limiting
2. **Backend → LLM**: DLP bidireccional, no enviar PII, system prompt hardened
3. **Backend → Azure DevOps**: PAT cifrado en vault, permisos minimos
4. **Backend → PostgreSQL**: conexion cifrada, queries parametrizadas, EF Core
