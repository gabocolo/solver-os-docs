# Prompt 04-FE — Implementar Modulo Prompt Builder (Frontend Angular)

| Campo | Valor |
|-------|-------|
| **Spec** | SPEC-L2-generate-prompts v1.0.0 |
| **Skill** | SK-001 — Generacion de codigo de caso de uso |
| **Modelo recomendado** | Sonnet |
| **Pareo backend** | PROMPT-04 (Implementar Prompt Builder) |

---

## Prompt

```
## Rol
Actua como un desarrollador senior Angular con experiencia en Angular 21, TypeScript strict, Angular Material y Tailwind CSS.

## Invocacion de especificacion
Aplica la especificacion SPEC-L2-generate-prompts version 1.0.0 + Wireframe Pantalla 5 (Prompt Builder).
[ADJUNTAR: contenido completo de SPEC-L2-generate-prompts.md]
[ADJUNTAR: seccion Pantalla 5 de 05-UI-WIREFRAMES.md]

## Tarea
Implementa el feature module Prompts segun la spec. Esto incluye:
1. PromptBuilderComponent: pagina principal con 3 paneles — selector (spec + skill), preview del prompt generado, acciones
2. SpecSelectorComponent: dropdown de specs APPROVED con busqueda, muestra nivel y version
3. SkillSelectorComponent: dropdown de skills activos del catalogo
4. PromptPreviewComponent: Monaco Editor read-only con el prompt generado (7 secciones: rol, invocacion, tarea, entrada/salida, ADRs, DLP, custom)
5. PromptActionsComponent: botones "Copiar al portapapeles", "Editar instrucciones custom", "Guardar", "Historial de prompts"
6. PromptHistoryComponent: tabla de prompts generados con filtros por spec/skill/fecha
7. PromptService: HttpClient calls a endpoints de PROMPT-04 backend

## Entrada
- API contract de SPEC-L2-generate-prompts: endpoints GeneratePrompt, UpdatePrompt, GetPromptHistory
- Interfaces tipadas: GeneratePromptRequest, StructuredPromptResponse, PromptContent, PromptHistoryResponse

## Salida esperada
Feature module con builder, preview, copy, history y tests unitarios.

## Reglas
- Todas las reglas provienen EXCLUSIVAMENTE de la spec.
- Standalone components (no NgModules).
- Angular Signals para estado reactivo.
- Reactive forms para instrucciones custom.
- Lazy loading del feature module en app routing.
- Copiar usa Clipboard API del navegador con feedback (snackbar "Copiado").
- Monaco Editor en modo read-only para preview del prompt.
- Custom instructions en textarea con limite de 2000 caracteres.
- El prompt se genera en backend (respuesta < 2s) — mostrar spinner durante la llamada.
- Historial paginado con mat-paginator (page size: 10, 25, 50).
- Solo specs con estado APPROVED aparecen en el selector.
- Solo skills con estado ACTIVE aparecen en el selector.
- ARIA labels y keyboard navigation (accesibilidad AA).
- No PII real en codigo/tests — datos sinteticos.

## Contexto de gobernanza adjunto
- ADR-001: Ports & Adapters — frontend consume APIs definidos en backend
- ADR-003: Trazabilidad — cada prompt generado trazable a spec + skill + usuario

## Restricciones DLP (ADR-006)
- Datos sinteticos en fixtures y mocks.
- No hardcodear tokens ni secretos.
- Environment variables para configuracion sensible.
- PromptContent es Confidencial — no loguear en consola del navegador.

## Restricciones de salida
- Diff < 400 lineas por PR.
- Si el diff es mayor, divide en PRs: 1) PromptBuilder + selectors, 2) Preview + Monaco, 3) Actions + History.
- Tests unitarios con Jasmine/Karma para PromptService y componentes.
- Cada componente en su propio archivo.
```

---

## Contexto a adjuntar

| Documento | Seccion |
|-----------|---------|
| SPEC-L2-generate-prompts.md | Completo |
| 05-UI-WIREFRAMES.md | Pantalla 5 — Prompt Builder |
| ADR-001 | Ports & Adapters |
| ADR-003 | Trazabilidad |
| ADR-006 | Data Classification & DLP |
