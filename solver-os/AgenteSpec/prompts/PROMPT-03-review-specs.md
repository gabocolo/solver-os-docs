# Prompt 03 — Implementar Review Service

| Campo | Valor |
|-------|-------|
| **Spec** | SPEC-L2-review-specs v1.0.0 |
| **Skill** | SK-001 — Generacion de codigo de caso de uso |
| **Modelo recomendado** | Sonnet |

---

## Prompt

```
## Rol
Actua como un desarrollador senior con experiencia en .NET 10, C# y arquitectura hexagonal.

## Invocacion de especificacion
Aplica la especificacion SPEC-L2-review-specs version 1.0.0.
[ADJUNTAR: contenido completo de SPEC-L2-review-specs.md]

## Tarea
Implementa el caso de uso ReviewAndApproveSpec segun la spec. Esto incluye:
1. Entidad de dominio ReviewCycle con ReviewComment
2. Value objects: ReviewDecision, ReviewStatus, ValidationResult, ValidationFinding
3. Puertos: ISpecRepository, IReviewCycleRepository, ISpecValidator, INotificationService, IEventPublisher, IAuditLogRepository, IDevOpsWorkItemService
4. Servicio de aplicacion ReviewService: SubmitForReview, AddComment, SubmitDecision
5. Implementacion del validador SK-004 (SpecValidator)
6. Endpoints Minimal API

## Entrada
- SubmitForReview: specId (UUID), reviewerId (UUID, diferente al autor, ARCHITECT o LEAD)
- AddComment: reviewId (UUID), section (string), comment (string)
- SubmitDecision: reviewId (UUID), decision (APPROVED | CHANGES_REQUESTED), summary (opcional)

## Salida esperada
- ValidationResult: passed (bool), findings (ValidationFinding[])
- ReviewCycleResponse: reviewId, specId, reviewerId, status, comments[], decision, validationResult, createdAt, completedAt
- SpecStatusResponse: specId, previousStatus, newStatus, approvedBy, approvedAt, reviewCycleCount

## Reglas
- Todas las reglas provienen EXCLUSIVAMENTE de la spec.
- Dependencias permitidas: SpecRepository, ReviewCycleRepository, SpecValidator, NotificationService, EventPublisher, AuditLogRepository, DevOpsWorkItemService.
- Errores: SPEC_NOT_FOUND (404), SPEC_NOT_IN_DRAFT (422), SPEC_NOT_IN_REVIEW (422), SELF_REVIEW_NOT_ALLOWED (422), REVIEWER_NOT_AUTHORIZED (403), VALIDATION_FAILED (422), REVIEW_NOT_FOUND (404), REVIEW_ALREADY_COMPLETED (422), DEVOPS_SYNC_FAILED (200 warning).
- Maquina de estados: DRAFT → IN_REVIEW → APPROVED (o CHANGES_REQUESTED → DRAFT).
- Validacion automatica (SK-004) se ejecuta al transicionar DRAFT → IN_REVIEW.
- Si validation CRITICAL falla → bloquear transicion.
- Cambio de estado atomico: si AuditLog falla, revertir estado.
- Al aprobar: publicar evento SpecApproved (dispara sync DevOps).
- Solo ARCHITECT y LEAD pueden aprobar. Reviewer != Autor.

## Contexto de gobernanza adjunto
- ADR-001: Ports & Adapters
- ADR-003: Trazabilidad — correlationId en cada operacion
- ADR-006: Clasificacion de datos

## Restricciones DLP (ADR-006)
- No incluyas datos reales (PII) en el codigo, tests o fixtures.
- ReviewCycle.comments es Interno — no exponer externamente.
- Notificaciones: no incluir contenido de la spec, solo referencia (specId + titulo).

## Restricciones de salida
- Dividir en PRs: 1) Entidades + puertos, 2) ReviewService + SpecValidator (SK-004), 3) Endpoints + notificaciones
- Cada PR < 400 lineas
- Tests basados en GWT-01 a GWT-08 de la spec
- Implementar los 8 checks de validacion de SK-004 (tabla de validaciones de la spec)
```

---

## Contexto a adjuntar

| Documento | Seccion |
|-----------|---------|
| SPEC-L2-review-specs.md | Completo |
| ADR-001 | Ports & Adapters |
| ADR-003 | Trazabilidad |
| ADR-006 | Data Classification |
