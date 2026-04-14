# Prompt 03-FE — Implementar Modulo Review (Frontend Angular)

| Campo | Valor |
|-------|-------|
| **Spec** | SPEC-L2-review-specs v1.0.0 |
| **Skill** | SK-001 — Generacion de codigo de caso de uso |
| **Modelo recomendado** | Sonnet (tarea acotada por spec) |
| **Pareo** | PROMPT-03 (backend) |

---

## Prompt

```
## Rol
Actua como un desarrollador senior Angular con experiencia en Angular 21, TypeScript strict, Angular Material y Tailwind CSS.

## Invocacion de especificacion
Aplica la especificacion SPEC-L2-review-specs version 1.0.0 + Wireframe Pantalla 4 (Panel de revision).
[ADJUNTAR: contenido completo de SPEC-L2-review-specs.md]
[ADJUNTAR: seccion Pantalla 4 de 05-UI-WIREFRAMES.md]
[ADJUNTAR: PROMPT-03 backend (API contract de referencia)]

## Tarea
Implementa el feature module Reviews segun la spec. Esto incluye:
1. ReviewListComponent: tabla de specs en estado IN_REVIEW con filtros por proyecto, nivel (L1/L2/L3) y reviewer asignado
2. ReviewPanelComponent: panel completo de revision con: spec content (Monaco Editor read-only), resultados de validacion (checklist), seccion de comentarios, botones de aprobar/rechazar
3. ReviewValidationComponent: muestra resultados de validacion automatica SK-004 con iconos check/cross para cada criterio: estructura completa, semver valido, clasificacion de datos presente, referencias a ADR validas, parent spec aprobada, glosario definido, agregados identificados, escenarios GWT presentes
4. ReviewCommentsComponent: lista de comentarios por seccion de la spec, formulario para agregar comentario con referencia a seccion/linea especifica, soporte markdown en comentarios
5. ReviewDecisionComponent: dialog de confirmacion (mat-dialog) para decision APPROVED o CHANGES_REQUESTED, campo de resumen obligatorio, firma digital (reviewer name + timestamp)
6. ReviewService: HttpClient calls a todos los endpoints definidos en PROMPT-03 backend, WebSocket para notificacion de nueva review asignada

## Entrada
API contract de SPEC-L2-review-specs: endpoints, tipos de request/response, codigos de error.
[ADJUNTAR: seccion de endpoints y tipos de SPEC-L2-review-specs.md]

## Salida esperada
Feature module con review panel, comments, decision flow, service, routing, tests unitarios.

## Reglas
- Todas las reglas, dependencias y errores provienen EXCLUSIVAMENTE de la spec.
- NO inventes endpoints, servicios ni dependencias fuera de la spec.
- Standalone components (no NgModules).
- Angular Signals para estado reactivo.
- Reactive forms (no template-driven).
- Monaco Editor en modo read-only para visualizar el contenido de la spec.
- Solo el reviewer asignado puede aprobar o rechazar.
- Resultados de validacion son read-only (generados por backend SK-004).
- Comentarios soportan markdown (usar pipe para renderizar).
- Decision requiere confirmacion via mat-dialog antes de enviar.
- WebSocket para recibir notificaciones de nueva review asignada (reusar WebSocket service de PROMPT-00-FE).
- Manejar errores del backend:
  - SPEC_NOT_FOUND (404) -> redirect a lista de reviews
  - SPEC_NOT_IN_REVIEW (422) -> snackbar "La spec no esta en revision"
  - SELF_REVIEW_NOT_ALLOWED (422) -> snackbar "No puedes revisar tu propia spec"
  - REVIEWER_NOT_AUTHORIZED (403) -> snackbar "No tienes permisos para revisar"
  - VALIDATION_FAILED (422) -> mostrar findings en ReviewValidationComponent
  - REVIEW_NOT_FOUND (404) -> redirect a lista de reviews
  - REVIEW_ALREADY_COMPLETED (422) -> snackbar "Esta revision ya fue completada"
- Datos sinteticos en tests y fixtures — no PII real.
- ARIA labels y keyboard navigation (accesibilidad AA).

## Contexto de gobernanza adjunto
- ADR-001: Ports & Adapters — frontend consume puertos/APIs definidos en backend
- ADR-003: Trazabilidad — correlationId en cada request HTTP via interceptor
- ADR-006: Clasificacion de datos — mostrar clasificacion en panel de revision
- CODE-STANDARDS.md: secciones relevantes a TypeScript

## Restricciones DLP (ADR-006)
- Datos sinteticos en fixtures y mocks.
- No hardcodear tokens ni secretos.
- Environment variables para configuracion sensible.
- ReviewCycle.comments es Interno — no exponer en vistas publicas.
- Spec content es Confidencial — no cachear en localStorage sin cifrado.

## Restricciones de salida
- Diff pequeno y revisable (< 400 lineas por PR).
- Si el diff es mayor, dividir en PRs: 1) ReviewList + routing, 2) ReviewPanel + Monaco read-only, 3) ReviewValidation + checklist SK-004, 4) ReviewComments + markdown, 5) ReviewDecision + dialog + ReviewService.
- Tests unitarios con Jasmine/Karma para componentes y service.
- Cada componente en su propio archivo.
- Codigo en la carpeta correcta: features/reviews/components/, features/reviews/services/.
```

---

## Contexto a adjuntar

| Documento | Seccion |
|-----------|---------|
| SPEC-L2-review-specs.md | Completo |
| 05-UI-WIREFRAMES.md | Pantalla 4 — Panel de revision |
| PROMPT-03 (backend) | API contract (endpoints, tipos, errores) |
| ADR-001 | Ports & Adapters |
| ADR-003 | Trazabilidad |
| ADR-006 | Data Classification |
| CODE-STANDARDS.md | TypeScript |
