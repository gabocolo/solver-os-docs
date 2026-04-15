# Prompt 09 — Implementar DLP Guard LLM (SK-007)

| Campo | Valor |
|-------|-------|
| **Spec** | SPEC-L2-create-specs v1.0.0 + SPEC-L1-spec-agent-portal v1.0.0 |
| **Skill** | SK-007 — DLP Guard para prompts y respuestas LLM |
| **Modelo recomendado** | Sonnet (tarea acotada por spec) |

---

## Prompt

```
## Rol
Actua como un desarrollador senior con experiencia en .NET 10, C#, Semantic Kernel y prevencion de fuga de datos (DLP).

## Invocacion de especificacion
Aplica el skill SK-007 (DLP Guard LLM) basado en:
- SPEC-L2-create-specs v1.0.0 (caso de uso que integra LLM)
- SPEC-L1-spec-agent-portal v1.0.0 (clasificacion de datos)
- ADR-P005 (DLP Filter Architecture)
[ADJUNTAR: contenido de SPEC-L2-create-specs, seccion clasificacion de datos de L1, ADR-P005]

## Tarea
Implementa el middleware DLP bidireccional para el Spec Agent Portal. El portal envia datos de usuario a Claude API (via Semantic Kernel) en dos casos de uso:
- CU-02: CreateSpec — usuario describe dominio/cambio en lenguaje natural → LLM genera spec
- CU-01: CreateADRWithAI — usuario describe decision tecnica → LLM genera ADR

El middleware debe:

1. **DLP Pre-Prompt Filter** (IDLPFilter — puerto en Domain):
   - Recibir el input del usuario antes de construir el prompt
   - Detectar PII usando patrones de DLP-PATTERNS.md:
     * Email: [a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}
     * Cedula colombiana: \b\d{6,10}\b (en contexto de documento)
     * Telefono: (\+57)?\s?\d{3}\s?\d{3}\s?\d{4}
     * Tarjeta de credito: \b\d{4}[\s-]?\d{4}[\s-]?\d{4}[\s-]?\d{4}\b
     * API Keys: (sk-[a-zA-Z0-9]{20,})|(ghp_[a-zA-Z0-9]{36})
     * JWT Token: eyJ[a-zA-Z0-9_-]+\.eyJ[a-zA-Z0-9_-]+\.[a-zA-Z0-9_-]+
     * Connection String: (Server|Host)=[^;]+;.*Password=[^;]+
   - Reemplazar PII detectada con placeholders: [EMAIL_1], [DOC_1], [PHONE_1]
   - Mantener mapa de reemplazo en memoria (TTL: duracion del request)
   - Registrar metrica: dlp_pre_prompt_violations (counter por pattern_type)
   - Log: tipo de PII detectada SIN el dato real (ej: "PII detected: EMAIL at position 45")

2. **DLP Post-Response Filter**:
   - Recibir respuesta del LLM antes de guardar en DB o mostrar al usuario
   - Detectar PII que el LLM pueda haber generado o replicado
   - Aplicar masking: email → u***@domain.com, documento → ***456
   - Registrar metrica: dlp_post_response_violations (counter por pattern_type)
   - Log: tipo de PII detectada en respuesta

3. **Adaptador de infraestructura** (DLPFilterAdapter):
   - Implementa IDLPFilter (puerto del dominio)
   - Configurable: patrones regex cargados desde configuracion (no hardcoded)
   - Fail-safe: si el filtro falla, loggear error y dejar pasar SIN PII
   - El mapa de reemplazo NUNCA se persiste ni se loguea

## Entrada
- Input del usuario (string de lenguaje natural para describir dominio/cambio)
- Patrones DLP de security/dlp/DLP-PATTERNS.md
- Campos PII del dominio segun SPEC-L1

## Salida esperada
- IDLPFilter.cs (puerto en Domain/Ports/)
- DLPFilterAdapter.cs (adaptador en Infrastructure/DLP/)
- DLPPatternConfig.cs (configuracion de patrones en Infrastructure/DLP/)
- Tests GWT:
  * Given input con email real → When se prepara prompt → Then email reemplazado por [EMAIL_1]
  * Given respuesta LLM con numero de documento → When se procesa → Then documento enmascarado
  * Given input sin PII → When se prepara prompt → Then input pasa sin modificacion
  * Given el filtro detecta PII → When registra metrica → Then dlp_pre_prompt_violations incrementa
  * Given el filtro falla (excepcion) → When procesa input → Then loguea error y deja pasar sin PII
  * Given tarjeta de credito en input → When se prepara prompt → Then se BLOQUEA el envio (no placeholder)
  * Given API key en input → When se prepara prompt → Then se BLOQUEA el envio

## Reglas
- El middleware opera en **capa de infraestructura** (adaptador), NO en dominio
- Puerto IDLPFilter en Domain/Ports/ — adaptador DLPFilterAdapter en Infrastructure/DLP/
- Fail-safe: NUNCA lanzar excepciones que bloqueen el flujo principal
- Mapa de reemplazo solo en memoria, NUNCA en logs ni persistencia
- Tarjetas de credito y API keys/secrets → BLOQUEAR envio (no solo enmascarar)
- Latencia del middleware < 50ms p95
- Usar datos sinteticos en los tests, nunca datos reales
- Registrar metricas: dlp_pre_prompt_violations, dlp_post_response_violations

## Contexto de gobernanza adjunto
- ADR-001: Ports & Adapters — IDLPFilter es un puerto, DLPFilterAdapter es un adaptador
- ADR-006: Data Classification — clasificacion obligatoria, campos PII definidos
- ADR-P005: DLP Filter Architecture — arquitectura DLP bidireccional, regex-based, configurable

## Restricciones DLP (ADR-006)
- No incluyas datos reales (PII) en el codigo, tests o fixtures — usa datos sinteticos
- Tests: usar emails como test@synthetic.test, documentos como 123456789 (sintetico)

## Restricciones de salida
- Diff pequeno y revisable (< 400 lineas por PR)
- Si el diff es mayor, dividir: 1) IDLPFilter + DLPFilterAdapter + config, 2) tests
- Codigo en Domain/Ports/ (interfaz) y Infrastructure/DLP/ (implementacion)
```

---

## Arquitectura del middleware

```
Usuario → [Input] → DLP Pre-Prompt Filter → [Input sanitizado] → Prompt Builder → Claude API
                                                                                      |
Usuario ← [Respuesta filtrada] ← DLP Post-Response Filter ← [Respuesta raw] ←-------┘
```

---

## Contexto a adjuntar

| Documento | Seccion |
|-----------|---------|
| SPEC-L2-create-specs.md | Completo (caso de uso con LLM) |
| SPEC-L1-spec-agent-portal.md | Clasificacion de datos (ADR-006) |
| ADR-P005 | DLP Filter Architecture |
| ADR-006 | Data Classification and DLP |
| ADR-001 | Ports & Adapters |
| DLP-PATTERNS.md | Patrones regex pre/post-prompt |
