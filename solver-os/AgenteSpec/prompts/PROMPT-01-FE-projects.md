# Prompt 01-FE — Implementar Modulo Projects (Frontend Angular)

| Campo | Valor |
|-------|-------|
| **Spec** | SPEC-L2-manage-projects v1.0.0 |
| **Skill** | SK-001 — Generacion de codigo de caso de uso |
| **Modelo recomendado** | Sonnet (tarea acotada por spec) |
| **Pareo** | PROMPT-01 (backend) |

---

## Prompt

```
## Rol
Actua como un desarrollador senior Angular con experiencia en Angular 21, TypeScript strict, Angular Material y Tailwind CSS.

## Invocacion de especificacion
Aplica la especificacion SPEC-L2-manage-projects version 1.0.0 + Wireframe Pantalla 1 (Dashboard de proyecto).
[ADJUNTAR: contenido completo de SPEC-L2-manage-projects.md]
[ADJUNTAR: seccion Pantalla 1 de 05-UI-WIREFRAMES.md]
[ADJUNTAR: PROMPT-01 backend (API contract de referencia)]

## Tarea
Implementa el feature module Projects segun la spec. Esto incluye:
1. ProjectListComponent: tabla con Angular Material (mat-table), filtros por nombre y estado, paginacion (mat-paginator)
2. ProjectCreateComponent: reactive form con validaciones (name max 200 caracteres, unico)
3. ProjectDashboardComponent: cards de metricas (total specs, approved, in review, data classification coverage), arbol de trazabilidad (L1->L2->L3)
4. ADRListComponent: tabla de ADRs del proyecto con columnas: number, title, status, createdAt
5. ADRCreateComponent: formulario de creacion de ADR + boton "Generar con IA" (llama POST /api/adrs/generate, muestra loading spinner)
6. QualityGatesConfigComponent: configuracion de quality gates del proyecto (checklist editable)
7. ProjectService: HttpClient calls a todos los endpoints definidos en PROMPT-01 backend

## Entrada
API contract de SPEC-L2-manage-projects: endpoints, tipos de request/response, codigos de error.
[ADJUNTAR: seccion de endpoints y tipos de SPEC-L2-manage-projects.md]

## Salida esperada
Feature module con components, service, routing, tests unitarios.

## Reglas
- Todas las reglas, dependencias y errores provienen EXCLUSIVAMENTE de la spec.
- NO inventes endpoints, servicios ni dependencias fuera de la spec.
- Standalone components (no NgModules).
- Angular Signals para estado reactivo.
- Reactive forms (no template-driven).
- HttpClient con interceptors definidos en PROMPT-00-FE (no axios).
- Manejar errores del backend:
  - PROJECT_NAME_DUPLICATE (409) -> snackbar con mensaje de error
  - PROJECT_NOT_FOUND (404) -> redirect a lista de proyectos
  - ADR_NUMBER_DUPLICATE (409) -> snackbar con mensaje de error
  - ADR_NOT_FOUND (404) -> redirect a lista de ADRs
  - UNAUTHORIZED (403) -> snackbar "Sin permisos"
  - LLM_UNAVAILABLE (503) -> snackbar "Servicio IA no disponible, intente mas tarde"
- Solo roles ARCHITECT y LEAD pueden crear proyectos y gestionar ADRs.
- Datos sinteticos en tests y fixtures — no PII real.
- ARIA labels y keyboard navigation (accesibilidad AA).

## Contexto de gobernanza adjunto
- ADR-001: Ports & Adapters — frontend consume puertos/APIs definidos en backend
- ADR-003: Trazabilidad — correlationId en cada request HTTP via interceptor
- CODE-STANDARDS.md: secciones relevantes a TypeScript

## Restricciones DLP (ADR-006)
- Datos sinteticos en fixtures y mocks.
- No hardcodear tokens ni secretos.
- Environment variables para configuracion sensible.
- Project.gitRepoUrl es Interno — no mostrar en vistas publicas.

## Restricciones de salida
- Diff pequeno y revisable (< 400 lineas por PR).
- Si el diff es mayor, dividir en PRs: 1) ProjectList + ProjectCreate + routing, 2) ProjectDashboard + metricas, 3) ADRList + ADRCreate + generacion IA, 4) QualityGatesConfig + ProjectService.
- Tests unitarios con Jasmine/Karma para componentes y service.
- Cada componente en su propio archivo.
- Codigo en la carpeta correcta: features/projects/components/, features/projects/services/.
```

---

## Contexto a adjuntar

| Documento | Seccion |
|-----------|---------|
| SPEC-L2-manage-projects.md | Completo |
| 05-UI-WIREFRAMES.md | Pantalla 1 — Dashboard de proyecto |
| PROMPT-01 (backend) | API contract (endpoints, tipos, errores) |
| ADR-001 | Ports & Adapters |
| ADR-003 | Trazabilidad |
| CODE-STANDARDS.md | TypeScript |
