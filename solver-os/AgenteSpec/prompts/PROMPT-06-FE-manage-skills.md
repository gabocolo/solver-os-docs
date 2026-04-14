# Prompt 06-FE — Implementar Modulo Skill Catalog (Frontend Angular)

| Campo | Valor |
|-------|-------|
| **Spec** | SPEC-L2-manage-skills v1.0.0 |
| **Skill** | SK-001 — Generacion de codigo de caso de uso |
| **Modelo recomendado** | Sonnet |
| **Pareo backend** | PROMPT-06 (Implementar Skill Catalog Service) |

---

## Prompt

```
## Rol
Actua como un desarrollador senior Angular con experiencia en Angular 21, TypeScript strict, Angular Material y Tailwind CSS.

## Invocacion de especificacion
Aplica la especificacion SPEC-L2-manage-skills version 1.0.0 + Wireframe (seccion Skills del sidebar).
[ADJUNTAR: contenido completo de SPEC-L2-manage-skills.md]
[ADJUNTAR: seccion Skills de 05-UI-WIREFRAMES.md]

## Tarea
Implementa el feature module Skills segun la spec. Esto incluye:
1. SkillListComponent: tabla de skills con filtros por tipo (Build/Workflow), estado (active/deprecated), metricas de uso
2. SkillCreateComponent: reactive form para crear skill (name, type, description, version, criteria, owner)
3. SkillDetailComponent: vista detalle con metricas (uso, % retrabajo, tiempo revision promedio), historial de versiones
4. SkillDeprecateComponent: dialog de confirmacion para deprecar skill (requiere justificacion)
5. SkillMetricsComponent: cards con metricas del skill individual (invocaciones, retrabajo, hallazgos)
6. SkillService: HttpClient calls a endpoints de PROMPT-06 backend

## Entrada
- API contract de SPEC-L2-manage-skills: endpoints CreateSkill, UpdateSkill, DeprecateSkill, GetSkillMetrics, ListSkills
- Interfaces tipadas: SkillResponse, SkillListResponse, SkillMetricsResponse, CreateSkillRequest, DeprecateSkillRequest

## Salida esperada
Feature module con CRUD, metrics, deprecation flow y tests unitarios.

## Reglas
- Todas las reglas provienen EXCLUSIVAMENTE de la spec.
- Standalone components (no NgModules).
- Angular Signals para estado reactivo.
- Reactive forms con validacion (name requerido y unico, version semver, type requerido).
- Lazy loading del feature module en app routing.
- Solo ARCHITECT y LEAD pueden crear y deprecar skills — ocultar botones si el rol no aplica (route guard + UI condicional).
- Version se incrementa automaticamente (semver) — mostrar version actual y sugerir siguiente minor.
- Skills deprecated se muestran en gris con badge "Deprecated" (mat-chip).
- Dialog de deprecacion (mat-dialog) requiere campo justificacion (min 20 caracteres) y opcionalmente replacedBy (selector de skill activo).
- Metricas con skeleton loader (mat-progress-bar) mientras cargan.
- Alerta visual si reworkPercentage > 20% por 2 sprints (badge rojo + tooltip con mensaje).
- Tabla con mat-sort y mat-paginator (page size: 10, 25, 50).
- ARIA labels y keyboard navigation (accesibilidad AA).
- No PII real en codigo/tests — datos sinteticos.

## Contexto de gobernanza adjunto
- ADR-001: Ports & Adapters — frontend consume APIs definidos en backend
- ADR-003: Trazabilidad — skills trazables en cadena spec → skill → prompt

## Restricciones DLP (ADR-006)
- Datos sinteticos en fixtures y mocks.
- No hardcodear tokens ni secretos.
- Environment variables para configuracion sensible.
- Skill.criteria es Interno — puede contener informacion tecnica interna, no exponer en UI publica.
- SkillMetrics son Interno — datos agregados sin PII.

## Restricciones de salida
- Diff < 400 lineas por PR.
- Si el diff es mayor, divide en PRs: 1) SkillList + filtros, 2) SkillCreate + form, 3) SkillDetail + metrics, 4) Deprecation flow.
- Tests unitarios con Jasmine/Karma para SkillService y componentes.
- Incluir test de restriccion de rol (solo ARCHITECT/LEAD pueden crear/deprecar).
- Cada componente en su propio archivo.
```

---

## Contexto a adjuntar

| Documento | Seccion |
|-----------|---------|
| SPEC-L2-manage-skills.md | Completo |
| 05-UI-WIREFRAMES.md | Seccion Skills |
| ADR-001 | Ports & Adapters |
| ADR-003 | Trazabilidad |
| ADR-006 | Data Classification & DLP |
| SKILLS-CATALOG.md | Para referencia de skills existentes |
