# Prompt 08 — Implementar Data Masking (SK-006)

| Campo | Valor |
|-------|-------|
| **Spec** | SPEC-L1-spec-agent-portal v1.0.0 (Clasificacion de datos) |
| **Skill** | SK-006 — Data Masking en codigo generado |
| **Modelo recomendado** | Sonnet (tarea acotada por spec) |

---

## Prompt

```
## Rol
Actua como un desarrollador senior con experiencia en .NET 10, C#, Serilog y proteccion de datos personales.

## Invocacion de especificacion
Aplica el skill SK-006 (Data Masking) basado en la clasificacion de datos de SPEC-L1-spec-agent-portal version 1.0.0.
[ADJUNTAR: seccion "Clasificacion de datos (ADR-006)" de la SPEC-L1]

## Tarea
Implementa los filtros de data masking para el Spec Agent Portal. Esto incluye:

1. **Filtro PII para Serilog**: enricher/destructuring policy que intercepta logs y aplica masking automatico:
   - User.email → u***@domain.com
   - User.name → J*** C***
   - Specification.content → [SPEC_CONTENT_REDACTED] (solo loggear metadata: specId, level, status)
   - ADR.content → [ADR_CONTENT_REDACTED] (solo loggear metadata: adrId, number, title)
   - Prompt.context → [PROMPT_CONTEXT_REDACTED]
   - QualityGateCheck.findings → [FINDINGS_REDACTED] (solo loggear count)

2. **Serializador seguro para respuestas API de error**:
   - No exponer User.userId en mensajes de error (reemplazar por correlationId)
   - No exponer Project.gitRepoUrl ni Project.devOpsProjectUrl en respuestas publicas
   - No exponer WorkItemSync.devOpsUrl en respuestas al frontend

3. **Anonimizador para ambientes no productivos**:
   - Servicio que genera datos sinteticos para User (email, name) en seeds/fixtures
   - Patron: email → user{n}@synthetic.test, name → "Test User {n}"

## Entrada
Campos PII segun SPEC-L1:
- User.email: Sensible (PII)
- User.name: Sensible (PII)
- Specification.content: Confidencial
- ADR.content: Confidencial
- Prompt.context: Confidencial
- QualityGateCheck.findings: Confidencial
- User.oauthProvider: Interno
- Project.gitRepoUrl: Interno
- Project.devOpsProjectUrl: Interno
- WorkItemSync.devOpsUrl: Interno

## Salida esperada
- PiiMaskingEnricher.cs (Serilog enricher en Infrastructure/Logging/)
- SecureErrorSerializer.cs (en Api/Middleware/)
- SyntheticDataGenerator.cs (en Infrastructure/TestData/)
- Tests GWT para cada componente

## Reglas
- El filtro opera en la **capa de infraestructura** (adaptador), NO en dominio
- Implementar como Serilog IDestructuringPolicy y ILogEventEnricher
- NO lanzar excepciones que bloqueen el flujo principal — fail-safe
- Usar datos sinteticos en los tests, nunca datos reales
- Registrar metricas: pii_masking_applied_total (counter por campo)
- Seguir ADR-001 (Ports & Adapters) y ADR-006 (Data Classification)

## Contexto de gobernanza adjunto
- ADR-001: Ports & Adapters — filtro es un adaptador de infraestructura
- ADR-006: Data Classification — clasificacion obligatoria
- OBSERVABILITY-STANDARDS.md: patrones de masking

## Restricciones DLP (ADR-006)
- No incluyas datos reales (PII) en el codigo, tests o fixtures — usa datos sinteticos
- Los tests deben usar emails como test@synthetic.test, nunca reales

## Restricciones de salida
- Diff pequeno y revisable (< 400 lineas por PR)
- Si el diff es mayor, dividir: 1) Serilog enricher + tests, 2) error serializer + anonymizer
- Codigo en Infrastructure/ y Api/, NUNCA en Domain/
```

---

## Contexto a adjuntar

| Documento | Seccion |
|-----------|---------|
| SPEC-L1-spec-agent-portal.md | Clasificacion de datos (ADR-006) |
| ADR-006 | Data Classification and DLP |
| ADR-P005 | DLP Filter Architecture |
| OBSERVABILITY-STANDARDS.md | Patrones de masking |
| DLP-PATTERNS.md | Patrones regex para PII |
