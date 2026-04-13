# ADR-P002: Generacion asincrona via Azure Service Bus

| Campo | Valor |
|-------|-------|
| **Estado** | ACCEPTED |
| **Fecha** | 2026-04-09 |
| **Derivado de** | ADR-002 (Idempotencia), RNF-PERF-07 (asincronia) |
| **Decisor** | Equipo de Arquitectura |

---

## Contexto

La generacion de specs con LLM puede tomar entre 30s y 120s. Una llamada HTTP sincrona con ese timeout bloquea al usuario, consume recursos del servidor y es vulnerable a desconexiones del navegador. Necesitamos un patron asincrono que sea robusto, idempotente y observable.

---

## Decision

Toda generacion de specs con IA se ejecuta de forma asincrona usando Azure Service Bus como cola de trabajos.

### Flujo

```
1. Cliente → API: POST /specs/generate/{level}
2. API valida input, crea job, encola mensaje en Service Bus
3. API retorna 202 Accepted con jobId
4. Worker (proceso separado) dequeue el mensaje
5. Worker ejecuta: DLP pre-prompt → LLM → DLP post-response → persistir spec
6. Worker notifica al frontend via WebSocket (o polling)
```

### Garantias

| Garantia | Mecanismo |
|----------|-----------|
| **Idempotencia** | JobId como idempotency key. Si el job ya existe con status COMPLETED, no re-ejecutar |
| **At-least-once delivery** | Azure Service Bus con PeekLock. Si worker falla, mensaje vuelve a la cola |
| **Dead-letter** | Tras 3 reintentos fallidos, mensaje va a dead-letter queue para investigacion |
| **Timeout** | Max 120s para L1, 60s para L2/L3. Si excede, registrar FAILED |
| **Observabilidad** | CorrelationId propagado desde request original hasta job |

---

## Alternativas evaluadas

### Alternativa 1: HTTP sincrono con long polling
- **Pro**: Simple, sin infraestructura adicional
- **Contra**: Timeout de browser (60s), bloquea thread del servidor, no idempotente
- **Descartada**: No escala, vulnerable a desconexiones

### Alternativa 2: Background task in-process (Hosted Service)
- **Pro**: Sin Service Bus, todo en el mismo proceso
- **Contra**: Si el proceso se reinicia, el job se pierde. No hay dead-letter ni reintentos
- **Descartada**: No es resiliente para operaciones criticas

### Alternativa 3: RabbitMQ
- **Pro**: Open source, sin costo de cloud
- **Contra**: Requiere infraestructura propia, no tiene dead-letter nativo tan robusto
- **Evaluada como fallback**: Si Service Bus no esta disponible, usar RabbitMQ

---

## Consecuencias

### Positivas
- Usuario no queda bloqueado durante generacion
- Jobs sobreviven reinicios del backend
- Dead-letter permite diagnosticar fallos sin perder mensajes
- Idempotencia evita duplicados en caso de reenvio

### Negativas
- Complejidad adicional: Service Bus + Worker + WebSocket
- Costo de Azure Service Bus (minimo en tier basico)
- Requiere monitoreo de la cola (profundidad, latencia, dead-letter)
