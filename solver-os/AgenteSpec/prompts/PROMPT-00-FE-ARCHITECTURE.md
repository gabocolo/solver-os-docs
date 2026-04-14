# Prompt 00-FE — Implementar Arquitectura Base Frontend (Angular 21)

| Campo | Valor |
|-------|-------|
| **Spec** | SPEC-L1-spec-agent-portal v1.0.0 |
| **Skill** | SK-001 — Generacion de codigo de caso de uso |
| **Modelo recomendado** | Sonnet (tarea acotada por spec) |

---

## Prompt

```
## Rol
Actua como un desarrollador senior Angular con experiencia en Angular 21, TypeScript strict, Angular Material y Tailwind CSS.

## Invocacion de especificacion
Aplica la especificacion SPEC-L1-spec-agent-portal version 1.0.0 + arquitectura de 01-SOLUTION-ARCHITECTURE.md.
[ADJUNTAR: contenido completo de SPEC-L1-spec-agent-portal.md]
[ADJUNTAR: seccion Frontend de 01-SOLUTION-ARCHITECTURE.md]

## Tarea
Genera la estructura base del proyecto Angular 21 con:
1. Angular 21 project scaffold (standalone components, signals)
2. App routing con lazy loading por feature (projects, specs, reviews, prompts, metrics, skills, governance)
3. Core module: HTTP interceptors (auth JWT, error handling, correlationId), WebSocket service para notificaciones
4. Shared module: Angular Material theme config, sidebar navigation layout (matching wireframe from 05-UI-WIREFRAMES.md), header con project selector y user menu
5. Auth module: OAuth 2.0 login, JWT token management, route guards por rol (ARCHITECT, LEAD, DEVELOPER)
6. Tailwind CSS config con design tokens del proyecto
7. Monaco Editor integration base (ngx-monaco-editor for markdown)
8. Environment configs (dev, staging, prod) con API base URL

## Entrada
Navegacion principal del wireframe 05-UI-WIREFRAMES.md.
[ADJUNTAR: seccion de navegacion principal de 05-UI-WIREFRAMES.md]

## Salida esperada
Project scaffold con routing, core services, shared layout, auth module.

## Reglas
- Solo genera la estructura base, NO features individuales (esos van en PROMPT-01-FE a PROMPT-06-FE).
- Standalone components (no NgModules).
- Angular Signals para estado reactivo.
- HttpClient con interceptors (no axios).
- Reactive forms (no template-driven).
- Lazy loading obligatorio por feature.
- No PII real en codigo/tests — datos sinteticos.
- ARIA labels y keyboard navigation (accesibilidad AA).

## Contexto de gobernanza adjunto
- ADR-001: Ports & Adapters — frontend consume puertos/APIs definidos en backend
- ADR-003: Trazabilidad — correlationId en cada request HTTP via interceptor
- CODE-STANDARDS.md: secciones relevantes a TypeScript

## Restricciones DLP (ADR-006)
- Datos sinteticos en fixtures y mocks.
- No hardcodear tokens ni secretos.
- Environment variables para configuracion sensible.

## Restricciones de salida
- Diff < 400 lineas por PR.
- Si el diff es mayor, divide en PRs: 1) scaffold + routing, 2) core + shared, 3) auth + Monaco.
- Tests unitarios con Jasmine/Karma para servicios core.
- Cada componente en su propio archivo.
```

---

## Contexto a adjuntar

| Documento | Seccion |
|-----------|---------|
| SPEC-L1-spec-agent-portal.md | Completo |
| 01-SOLUTION-ARCHITECTURE.md | Seccion Frontend |
| 05-UI-WIREFRAMES.md | Navegacion principal |
| ADR-001 | Ports & Adapters |
| CODE-STANDARDS.md | TypeScript |
