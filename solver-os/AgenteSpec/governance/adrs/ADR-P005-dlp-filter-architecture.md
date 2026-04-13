# ADR-P005: Arquitectura del filtro DLP bidireccional

| Campo | Valor |
|-------|-------|
| **Estado** | ACCEPTED |
| **Fecha** | 2026-04-09 |
| **Derivado de** | ADR-006 (Clasificacion de datos y DLP), SK-007 |
| **Decisor** | Equipo de Arquitectura |

---

## Contexto

El Spec Agent Portal envia datos del usuario a un LLM externo (Claude API) para generar specs. Los datos pueden contener informacion confidencial (reglas de negocio, descripciones de dominio) y potencialmente PII si el usuario la incluye en la descripcion. La respuesta del LLM puede generar o inferir PII. Necesitamos un filtro DLP bidireccional que opere en ambas direcciones.

---

## Decision

Implementar un `IDLPFilter` como puerto del dominio con dos operaciones: `SanitizeInput` (pre-prompt) y `ValidateOutput` (post-response). El adaptador implementa deteccion basada en regex + patrones configurables.

### Arquitectura

```
                    IDLPFilter (Puerto)
                    ├── SanitizeInput(text) → sanitizedText
                    └── ValidateOutput(text) → (text, findings[])

                    DLPFilterAdapter (Adaptador)
                    ├── Patrones de email: /[\w.-]+@[\w.-]+\.\w+/
                    ├── Patrones de documento: /\d{6,12}/ (contexto)
                    ├── Patrones de telefono: /\+?\d{7,15}/
                    ├── Patrones de tarjeta: /\d{4}[\s-]?\d{4}[\s-]?\d{4}[\s-]?\d{4}/
                    ├── Patrones de secreto: /(?:key|token|password|secret)[\s=:]+\S+/i
                    └── Patrones custom (configurables por proyecto)
```

### Fase Pre-prompt (SanitizeInput)

Se ejecuta ANTES de enviar al LLM:
1. Recibe el texto completo que se enviara como contexto
2. Aplica patrones de deteccion de PII
3. Reemplaza hallazgos con placeholders: `[REDACTED-EMAIL]`, `[REDACTED-PHONE]`
4. Registra hallazgo en metricas: `dlp_pre_prompt_findings_total`
5. Retorna texto sanitizado

### Fase Post-response (ValidateOutput)

Se ejecuta DESPUES de recibir respuesta del LLM:
1. Recibe la respuesta completa del LLM
2. Aplica mismos patrones de deteccion
3. Si detecta PII generada: reemplaza con dato sintetico (ej: `user@example.com`)
4. Registra hallazgo: `dlp_post_response_findings_total`
5. Si hallazgos > umbral configurable: loguear warning
6. Retorna respuesta sanitizada + lista de findings

### Configuracion por proyecto

```json
{
  "patterns": {
    "email": { "enabled": true, "replacement": "[REDACTED-EMAIL]" },
    "phone": { "enabled": true, "replacement": "[REDACTED-PHONE]" },
    "document": { "enabled": true, "replacement": "[REDACTED-DOC]" },
    "creditCard": { "enabled": true, "replacement": "[REDACTED-CC]" },
    "custom": [
      { "name": "internal-id", "pattern": "EMP-\\d{5}", "replacement": "[REDACTED-ID]" }
    ]
  },
  "thresholds": {
    "maxFindingsWarning": 3,
    "blockOnCritical": true
  }
}
```

---

## Alternativas evaluadas

### Alternativa 1: Usar el LLM para detectar PII
- **Pro**: Deteccion semantica, entiende contexto
- **Contra**: Costo adicional (llamada extra), latencia, el LLM ya vio los datos
- **Descartada**: Ineficiente y circular — el dato ya llego al LLM

### Alternativa 2: Servicio DLP externo (Azure Purview, Google DLP)
- **Pro**: Deteccion enterprise, ML-based
- **Contra**: Costo adicional, latencia de red, dependencia externa
- **Evaluada para futuro**: Puede reemplazar regex en version enterprise

### Alternativa 3: No filtrar, confiar en instrucciones del prompt
- **Pro**: Zero overhead
- **Contra**: Los usuarios pueden pegar PII sin darse cuenta. El LLM puede generar PII
- **Descartada**: Viola ADR-006 y el principio de Privacy by Design

---

## Consecuencias

### Positivas
- Proteccion bidireccional sin depender de servicios externos
- Latencia minima (< 50ms para regex)
- Configuracion por proyecto permite personalizar patrones
- Metricas de hallazgos alimentan el dashboard de DLP

### Negativas
- Regex no detecta PII semantica (ej: "el paciente Juan del piso 3")
- Falsos positivos posibles (numeros que parecen documentos)
- Requiere mantenimiento de patrones ante nuevos tipos de PII
- No protege contra prompt injection (concern separado)
