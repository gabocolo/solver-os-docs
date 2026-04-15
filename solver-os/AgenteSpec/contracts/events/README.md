# Eventos de Dominio — Spec Agent Portal

| Evento | Descripcion | Publicador | Consumidores |
|--------|------------|-----------|-------------|
| [SpecApproved](SpecApproved.json) | Spec aprobada por reviewer | Review Service | DevOps Sync, Notifications, Metrics |
| [SpecStatusChanged](SpecStatusChanged.json) | Cualquier cambio de estado de una spec | Spec Management Service | Metrics (cache invalidation) |
| [PromptGenerated](PromptGenerated.json) | Prompt generado desde spec aprobada | Prompt Builder | Metrics, AuditLog |

## Reglas

- Los eventos se publican **DESPUES** de la persistencia exitosa
- Todos incluyen `correlationId` para trazabilidad (ADR-003)
- Campos con `x-data-classification` deben tratarse segun ADR-006
- Canal: Azure Service Bus (ADR-P002)
- Entrega: At-least-once (consumidores deben ser idempotentes)
