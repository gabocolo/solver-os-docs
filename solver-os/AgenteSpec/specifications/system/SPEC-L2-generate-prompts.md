# Especificacion de Sistema (Nivel 2)

| Campo | Valor |
|-------|-------|
| **Spec ID** | SPEC-L2-generate-prompts |
| **Nivel** | L2 — Sistema |
| **Version** | 1.0.0 |
| **Estado** | DRAFT |
| **Parent** | SPEC-L1-spec-agent-portal v1.0.0 |
| **Caso de uso** | CU-04: Generar prompt estructurado |
| **Creado por** | Equipo de Arquitectura |
| **Fecha de creacion** | 2026-04-09 |
| **Aprobado por** | Pendiente |
| **Fecha de aprobacion** | Pendiente |
| **PR** | Pendiente |

---

## Descripcion general

Modulo de generacion de prompts estructurados del Spec Agent Portal. Permite al desarrollador senior seleccionar una spec aprobada y un skill del catalogo para generar un prompt que invoque la spec con el contexto correcto, reglas de gobernanza y restricciones DLP. El prompt se puede previsualizar, editar y copiar al clipboard. Toda generacion queda registrada para trazabilidad.

---

## Sistema

**Nombre:** Agent Orchestration Service (Prompt Builder)

## Caso de uso

**Nombre:** GenerateStructuredPrompt

**Descripcion:** El usuario selecciona una spec en estado APPROVED y un skill del catalogo. El portal construye un prompt estructurado con: rol, invocacion de spec (ID + version), tarea segun skill, entrada/salida desde spec, reglas de gobernanza (ADRs relevantes) y restricciones DLP. El usuario puede editar el prompt antes de copiarlo o enviarlo.

---

## Contrato tecnico

### Input — GeneratePrompt

| Campo | Tipo | Requerido | Descripcion |
|-------|------|-----------|-------------|
| specId | UUID | Si | Spec aprobada sobre la cual generar el prompt |
| skillId | UUID | Si | Skill del catalogo a invocar |
| customInstructions | string | No | Instrucciones adicionales del usuario |

### Output — StructuredPromptResponse

| Campo | Tipo | Descripcion |
|-------|------|-------------|
| promptId | UUID | ID del prompt generado |
| specId | UUID | Spec referenciada |
| specVersion | string | Version de la spec al momento de generar |
| skillId | UUID | Skill invocado |
| skillName | string | Nombre del skill |
| content | PromptContent | Contenido estructurado del prompt |
| generatedBy | UUID | Usuario que genero |
| generatedAt | datetime | Fecha de generacion |

### PromptContent

| Campo | Tipo | Descripcion |
|-------|------|-------------|
| role | string | Rol asignado al agente (ej: "Actua como un desarrollador senior") |
| invocation | string | Referencia a spec: "Aplica la especificacion {specId} v{version}" |
| task | string | Tarea segun el skill seleccionado |
| input | string | Entrada definida en la spec (contrato tecnico) |
| output | string | Salida esperada definida en la spec |
| governance | string[] | ADRs relevantes aplicables |
| dlpRules | string[] | Restricciones DLP: datos sinteticos, masking, no PII en tests |
| customInstructions | string | Instrucciones adicionales del usuario |
| fullText | string | Texto completo del prompt listo para copiar |

### Input — UpdatePrompt

| Campo | Tipo | Requerido | Descripcion |
|-------|------|-----------|-------------|
| promptId | UUID | Si | Prompt a editar |
| fullText | string | Si | Texto editado por el usuario |

---

## Dependencias permitidas

| Dependencia | Tipo | Proposito |
|-------------|------|----------|
| SpecRepository | Puerto (persistencia) | Obtener spec aprobada con contrato y reglas |
| SkillCatalogRepository | Puerto (persistencia) | Obtener skill con criterios de uso y template |
| ADRRepository | Puerto (persistencia) | Obtener ADRs relevantes del proyecto |
| PromptRepository | Puerto (persistencia) | Persistir prompt generado para trazabilidad |
| DLPFilter | Puerto (seguridad) | Generar reglas DLP segun clasificacion de datos de la spec |
| AuditLogRepository | Puerto (auditoria) | Registrar generacion de prompt |

---

## Errores

| Codigo | Descripcion | HTTP Status |
|--------|-------------|-------------|
| SPEC_NOT_FOUND | La spec no existe | 404 Not Found |
| SPEC_NOT_APPROVED | Solo se pueden generar prompts sobre specs APPROVED | 422 Unprocessable |
| SKILL_NOT_FOUND | El skill no existe en el catalogo | 404 Not Found |
| SKILL_DEPRECATED | El skill esta deprecado y no se puede usar para nuevos prompts | 422 Unprocessable |
| PROMPT_NOT_FOUND | El prompt a editar no existe | 404 Not Found |
| UNAUTHORIZED | Usuario no tiene permisos para generar prompts | 403 Forbidden |

---

## Requisitos de ejecucion

| Aspecto | Requisito |
|---------|----------|
| **Latencia** | Generar prompt: < 2s (p95). No requiere LLM — es construccion local |
| **Observabilidad** | `prompt_generated_total`, `prompt_generated_by_skill` (por skillId), `prompt_copied_total` |
| **Autorizacion** | ARCHITECT, LEAD y SENIOR_DEV pueden generar prompts |
| **Trazabilidad** | Todo prompt queda registrado: specId + specVersion + skillId + userId + timestamp |

---

## Proteccion de datos

| Campo / Flujo | Clasificacion | Tratamiento |
|---------------|---------------|-------------|
| Prompt.content | Confidencial | Contiene referencia a spec y reglas — registrar para trazabilidad |
| DLP rules en prompt | Interno | Incluir reglas de datos sinteticos y masking automaticamente |
| Clipboard | N/A | El portal no tiene control sobre el clipboard — la proteccion esta en el prompt mismo |

**Reglas DLP automaticas en el prompt**:
- Si la spec tiene campos Sensible → incluir instruccion "Aplica masking en logs para campos clasificados como Sensible"
- Siempre incluir: "No incluyas datos reales (PII) en el codigo, tests o fixtures — usa datos sinteticos"
- Si la spec referencia ADR-006 → incluir: "Cumple ADR-006: clasificacion de datos y DLP"

---

## Escenarios de test (GWT)

### GWT-01: Generar prompt exitosamente

```
Given una spec L2 en estado APPROVED (SPEC-L2-create-booking v1.0.0)
  And un skill activo SK-001 (Generacion de codigo CU)
  And un usuario con rol SENIOR_DEV
When solicita generar prompt con specId y skillId
Then el sistema:
  1. Carga la spec con contrato, dependencias y clasificacion
  2. Carga el skill con criterios de uso
  3. Carga ADRs relevantes del proyecto
  4. Construye prompt con: rol, invocacion, tarea, input/output, gobernanza, DLP
  5. Persiste el prompt con referencia a spec + skill + usuario
  6. Registra en AuditLog: PROMPT_GENERATED
  7. Retorna el prompt completo con preview
```

### GWT-02: Rechazar prompt sobre spec no aprobada

```
Given una spec en estado DRAFT
When un usuario intenta generar un prompt
Then el sistema retorna 422 con error SPEC_NOT_APPROVED
```

### GWT-03: Rechazar prompt con skill deprecado

```
Given una spec APPROVED
  And un skill en estado DEPRECATED
When un usuario intenta generar prompt con ese skill
Then el sistema retorna 422 con error SKILL_DEPRECATED
```

### GWT-04: Incluir reglas DLP automaticamente

```
Given una spec L2 cuya L1 parent tiene campos clasificados como Sensible (User.email, User.name)
When se genera un prompt
Then el prompt incluye automaticamente:
  - "No incluyas datos reales (PII) en el codigo, tests o fixtures"
  - "Aplica masking en logs para campos clasificados como Sensible en la spec"
  - Referencia a ADR-006
```

### GWT-05: Editar prompt antes de copiar

```
Given un prompt generado con promptId
When el usuario modifica el texto del prompt (agrega instrucciones adicionales)
  And guarda los cambios
Then el sistema actualiza el prompt en la base de datos
  And mantiene la referencia original a spec + skill
```

### GWT-06: Copiar prompt al clipboard

```
Given un prompt generado y previsualizado
When el usuario hace clic en "Copiar al clipboard"
Then el texto completo del prompt se copia al clipboard del navegador
  And se incrementa la metrica prompt_copied_total
```

---

## Referencias

- SPEC-L1-spec-agent-portal v1.0.0 — CU-04
- RF-PROMPT-01 a RF-PROMPT-06
- RNF-PERF-06 (< 2s)
- ADR-003 (Traceability)
- ADR-006 (Data Classification & DLP)
