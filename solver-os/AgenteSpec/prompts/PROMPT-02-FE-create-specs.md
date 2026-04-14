# Prompt 02-FE — Implementar Modulo Spec Editor (Frontend Angular)

| Campo | Valor |
|-------|-------|
| **Spec** | SPEC-L2-create-specs v1.0.0 |
| **Skill** | SK-001 — Generacion de codigo de caso de uso |
| **Modelo recomendado** | Sonnet (tarea acotada por spec) |
| **Pareo** | PROMPT-02 (backend) |

---

## Prompt

```
## Rol
Actua como un desarrollador senior Angular con experiencia en Angular 21, TypeScript strict, Angular Material y Tailwind CSS.

## Invocacion de especificacion
Aplica la especificacion SPEC-L2-create-specs version 1.0.0 + Wireframes Pantalla 2 (Editor de Specs) y Pantalla 3 (Generacion con IA).
[ADJUNTAR: contenido completo de SPEC-L2-create-specs.md]
[ADJUNTAR: secciones Pantalla 2 y Pantalla 3 de 05-UI-WIREFRAMES.md]
[ADJUNTAR: PROMPT-02 backend (API contract de referencia)]

## Tarea
Implementa el feature module Specs segun la spec. Esto incluye:
1. SpecListComponent: tabla de specs con filtros por nivel (L1/L2/L3), estado (DRAFT/IN_REVIEW/APPROVED/CHANGES_REQUESTED) y proyecto
2. SpecCreateWizardComponent: stepper (mat-stepper) con pasos: seleccionar nivel -> seleccionar parent (si L2/L3, solo specs APPROVED) -> descripcion en lenguaje natural -> confirmar y enviar
3. SpecEditorComponent: Monaco Editor (ngx-monaco-editor) para edicion de markdown, panel lateral con metadata (version, nivel, estado, parent, clasificacion de datos), botones de regenerar seccion individual
4. SpecGenerationProgressComponent: barra de progreso (mat-progress-bar) para generacion asincrona, polling de estado via GET /api/specs/jobs/{jobId}, notificacion WebSocket cuando la generacion completa
5. SpecDetailComponent: vista de spec completa con navegacion por secciones, metadata, historial de versiones
6. SpecService: HttpClient calls a todos los endpoints definidos en PROMPT-02 backend, WebSocket para notificaciones de generacion completada

## Entrada
API contract de SPEC-L2-create-specs: endpoints, tipos de request/response, codigos de error.
[ADJUNTAR: seccion de endpoints y tipos de SPEC-L2-create-specs.md]

## Salida esperada
Feature module con wizard, Monaco editor, progress tracking, service, routing, tests unitarios.

## Reglas
- Todas las reglas, dependencias y errores provienen EXCLUSIVAMENTE de la spec.
- NO inventes endpoints, servicios ni dependencias fuera de la spec.
- Standalone components (no NgModules).
- Angular Signals para estado reactivo.
- Reactive forms (no template-driven).
- Monaco Editor con syntax highlighting para markdown.
- mat-stepper para el wizard de creacion.
- WebSocket para real-time updates de generacion (reusar WebSocket service de PROMPT-00-FE).
- Parent spec dropdown solo muestra specs con estado APPROVED.
- Manejar errores del backend:
  - PARENT_NOT_APPROVED (422) -> snackbar "La spec padre debe estar aprobada"
  - PARENT_NOT_FOUND (404) -> snackbar "Spec padre no encontrada"
  - PARENT_WRONG_LEVEL (422) -> snackbar "Nivel de spec padre incorrecto"
  - DUPLICATE_SPEC (409) -> snackbar "Ya existe una spec con ese titulo"
  - LLM_UNAVAILABLE (503) -> dialog de reintento "Servicio IA no disponible"
  - LLM_TIMEOUT (504) -> dialog de reintento "Tiempo de espera agotado"
  - SECTION_NOT_FOUND (400) -> snackbar "Seccion no encontrada"
  - UNAUTHORIZED (403) -> snackbar "Sin permisos"
- Datos sinteticos en tests y fixtures — no PII real.
- ARIA labels y keyboard navigation (accesibilidad AA).

## Contexto de gobernanza adjunto
- ADR-001: Ports & Adapters — frontend consume puertos/APIs definidos en backend
- ADR-003: Trazabilidad — correlationId en cada request HTTP via interceptor
- ADR-P002: Generacion asincrona — frontend debe manejar polling + WebSocket para jobs
- CODE-STANDARDS.md: secciones relevantes a TypeScript

## Restricciones DLP (ADR-006)
- Datos sinteticos en fixtures y mocks.
- No hardcodear tokens ni secretos.
- Environment variables para configuracion sensible.
- Spec content es Confidencial — no cachear en localStorage sin cifrado.
- Input del usuario en wizard es Confidencial — no loguear en consola.

## Restricciones de salida
- Diff pequeno y revisable (< 400 lineas por PR).
- Si el diff es mayor, dividir en PRs: 1) SpecList + routing, 2) SpecCreateWizard + stepper, 3) SpecEditor + Monaco, 4) SpecGenerationProgress + WebSocket + polling, 5) SpecDetail + SpecService.
- Tests unitarios con Jasmine/Karma para componentes y service.
- Cada componente en su propio archivo.
- Codigo en la carpeta correcta: features/specs/components/, features/specs/services/.
```

---

## Contexto a adjuntar

| Documento | Seccion |
|-----------|---------|
| SPEC-L2-create-specs.md | Completo |
| 05-UI-WIREFRAMES.md | Pantalla 2 — Editor de Specs, Pantalla 3 — Generacion con IA |
| PROMPT-02 (backend) | API contract (endpoints, tipos, errores) |
| ADR-001 | Ports & Adapters |
| ADR-003 | Trazabilidad |
| ADR-P002 | Generacion asincrona |
| CODE-STANDARDS.md | TypeScript |
