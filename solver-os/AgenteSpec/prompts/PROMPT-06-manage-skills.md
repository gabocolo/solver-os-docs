# Prompt 06 — Implementar Skill Catalog Service

| Campo | Valor |
|-------|-------|
| **Spec** | SPEC-L2-manage-skills v1.0.0 |
| **Skill** | SK-001 — Generacion de codigo de caso de uso |
| **Modelo recomendado** | Sonnet |

---

## Prompt

```
## Rol
Actua como un desarrollador senior con experiencia en .NET 10, C# y arquitectura hexagonal.

## Invocacion de especificacion
Aplica la especificacion SPEC-L2-manage-skills version 1.0.0.
[ADJUNTAR: contenido completo de SPEC-L2-manage-skills.md]

## Tarea
Implementa el caso de uso ManageSkillCatalog segun la spec. Esto incluye:
1. Entidad de dominio Skill con value objects (SkillType, SkillStatus)
2. Value objects: SkillMetricsResponse, SkillMetricHistoryEntry
3. Puertos: ISkillCatalogRepository, IPromptRepository, IMetricsService, IAuditLogRepository, IEventPublisher
4. Servicio de aplicacion SkillCatalogService: CreateSkill, UpdateSkill, DeprecateSkill, GetSkillMetrics, ListSkills
5. Logica de alerta de retrabajo (> 20% por 2 sprints → alert)
6. Endpoints Minimal API

## Entrada
- CreateSkill: name (unico), type (BUILD/WORKFLOW), version (semver), owner, phase, criteria?, projectId?
- UpdateSkill: skillId, version (mayor que actual), criteria?, owner?
- DeprecateSkill: skillId, reason, replacedBy?
- GetSkillMetrics: skillId, period (sprint/month)
- ListSkills: filtros por type, status, projectId

## Salida esperada
- SkillResponse: skillId, name, type, version, owner, phase, criteria, status, projectId, createdAt, updatedAt
- SkillListResponse: skills[], total, active, deprecated
- SkillMetricsResponse: invocations, uniqueUsers, reworkPercentage, avgReviewTime, securityFindings, alert, alertMessage, history[]

## Reglas
- Todas las reglas provienen EXCLUSIVAMENTE de la spec.
- Dependencias: SkillCatalogRepository, PromptRepository, MetricsService, AuditLogRepository, EventPublisher.
- Errores: SKILL_NOT_FOUND (404), SKILL_NAME_DUPLICATE (409), SKILL_ALREADY_DEPRECATED (422), INVALID_VERSION (422), UNAUTHORIZED (403).
- Solo ARCHITECT puede crear, editar y deprecar skills. Todos pueden consultar.
- Al deprecar: publicar evento SkillDeprecated, prevenir uso en nuevos prompts.
- Alerta: si reworkPercentage > 20% sostenido por 2 sprints → generar alerta al owner.

## Contexto de gobernanza adjunto
- ADR-001: Ports & Adapters
- Framework v1.1, Seccion 7 (Skills)

## Restricciones DLP (ADR-006)
- No incluyas datos reales (PII) en el codigo, tests o fixtures.
- Skill.criteria es Interno — puede contener informacion tecnica interna.
- SkillMetrics son Interno — datos agregados sin PII.

## Restricciones de salida
- PR unico (funcionalidad acotada, < 400 lineas)
- Tests basados en GWT-01 a GWT-06 de la spec
- Incluir test de alerta de retrabajo (GWT-05)
```

---

## Contexto a adjuntar

| Documento | Seccion |
|-----------|---------|
| SPEC-L2-manage-skills.md | Completo |
| ADR-001 | Ports & Adapters |
| SKILLS-CATALOG.md | Para referencia de skills existentes |
