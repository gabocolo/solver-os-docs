# Prompt 04 — Implementar Prompt Builder

| Campo | Valor |
|-------|-------|
| **Spec** | SPEC-L2-generate-prompts v1.0.0 |
| **Skill** | SK-001 — Generacion de codigo de caso de uso |
| **Modelo recomendado** | Sonnet |

---

## Prompt

```
## Rol
Actua como un desarrollador senior con experiencia en .NET 10, C# y arquitectura hexagonal.

## Invocacion de especificacion
Aplica la especificacion SPEC-L2-generate-prompts version 1.0.0.
[ADJUNTAR: contenido completo de SPEC-L2-generate-prompts.md]

## Tarea
Implementa el caso de uso GenerateStructuredPrompt segun la spec. Esto incluye:
1. Entidad de dominio Prompt con PromptContent value object
2. Puertos: ISpecRepository, ISkillCatalogRepository, IADRRepository, IPromptRepository, IDLPFilter, IAuditLogRepository
3. Servicio de aplicacion PromptBuilderService: GeneratePrompt, UpdatePrompt
4. Logica de construccion local del prompt (sin LLM — es ensamblaje de secciones)
5. Inclusion automatica de reglas DLP segun clasificacion de datos
6. Endpoints Minimal API

## Entrada
- GeneratePrompt: specId (UUID, APPROVED), skillId (UUID, ACTIVE), customInstructions (opcional)
- UpdatePrompt: promptId (UUID), fullText (string editado)

## Salida esperada
- StructuredPromptResponse: promptId, specId, specVersion, skillId, skillName, content (PromptContent), generatedBy, generatedAt
- PromptContent: role, invocation, task, input, output, governance[], dlpRules[], customInstructions, fullText

## Reglas
- Todas las reglas provienen EXCLUSIVAMENTE de la spec.
- Dependencias permitidas: SpecRepository, SkillCatalogRepository, ADRRepository, PromptRepository, DLPFilter, AuditLogRepository.
- Errores: SPEC_NOT_FOUND (404), SPEC_NOT_APPROVED (422), SKILL_NOT_FOUND (404), SKILL_DEPRECATED (422), PROMPT_NOT_FOUND (404), UNAUTHORIZED (403).
- La generacion de prompt es LOCAL (< 2s), NO usa LLM.
- Si la spec tiene campos Sensible → incluir automaticamente regla DLP de masking.
- Siempre incluir: "No incluyas datos reales (PII) en codigo, tests o fixtures".
- Si la spec referencia ADR-006 → incluir: "Cumple ADR-006".
- Todo prompt queda registrado para trazabilidad: specId + specVersion + skillId + userId.

## Contexto de gobernanza adjunto
- ADR-001: Ports & Adapters
- ADR-003: Trazabilidad — cada prompt trazable a spec + skill
- ADR-006: Reglas DLP automaticas en prompt

## Restricciones DLP (ADR-006)
- No incluyas datos reales (PII) en el codigo, tests o fixtures.
- Prompt.content es Confidencial — contiene referencia a spec y reglas.
- Incluir DLP rules automaticamente segun clasificacion de la spec parent.

## Restricciones de salida
- PR unico (funcionalidad acotada, < 400 lineas)
- Tests basados en GWT-01 a GWT-06 de la spec
- Codigo en: Domain/ (entidad + puerto), Application/ (servicio), API/ (endpoints)
```

---

## Contexto a adjuntar

| Documento | Seccion |
|-----------|---------|
| SPEC-L2-generate-prompts.md | Completo |
| ADR-001 | Ports & Adapters |
| ADR-003 | Trazabilidad |
| ADR-006 | Data Classification & DLP |
