# Definicion de Skill

| Campo | Valor |
|-------|-------|
| **ID** | SK-007 |
| **Nombre** | DLP Guard para prompts y respuestas LLM |
| **Tipo** | Build |
| **Version** | 1.0.0 |
| **Owner** | Lider tecnico / Arquitecto |
| **Fase del framework** | Fase 3 — Generacion controlada |
| **Estado** | ACTIVE |

---

## Descripcion

Genera la implementacion de un middleware/proxy DLP que intercepta y sanitiza prompts antes de enviarlos a un LLM y filtra las respuestas antes de entregarlas al usuario o sistema. Este skill aplica cuando la aplicacion integra LLMs en runtime (no para uso de IA en desarrollo, que se cubre con estandares de seguridad).

El middleware actua como una capa de proteccion bidireccional:
- **Pre-prompt (entrada)**: detecta y enmascara PII en el input del usuario antes de incluirlo en el prompt al LLM
- **Post-respuesta (salida)**: detecta PII que el LLM pueda haber generado o inferido y la enmascara antes de retornar

---

## Cuando usar

- Cuando la aplicacion envia datos de usuario a un LLM externo (OpenAI, Anthropic, etc.)
- Cuando el sistema procesa input de usuarios que se incluye en prompts (riesgo de PII en prompt)
- Cuando las respuestas del LLM se muestran a otros usuarios o se persisten

---

## Cuando NO usar

- Para proteccion de prompts en desarrollo (eso se cubre con SECURITY-STANDARDS.md y datos sinteticos)
- Si la aplicacion no integra LLMs en runtime
- Para validacion de prompt injection (ese es un concern de seguridad separado, aunque el middleware puede extenderse para cubrirlo)

---

## Input requerido

| Input | Descripcion | Obligatorio |
|-------|------------|-------------|
| Especificacion L2 del caso de uso con LLM | Spec que define la integracion con LLM, campos de entrada del usuario | Si |
| ADR-006 | Decision de clasificacion y DLP | Si |
| Clasificacion de datos de la spec L1 | Que campos del dominio son PII | Si |
| Proveedor de LLM | Que API de LLM se usa (para adaptar el middleware) | Si |
| Stack tecnologico | Lenguaje y framework del proyecto | Si |

---

## Output esperado

1. **Middleware DLP pre-prompt**: componente que:
   - Recibe el input del usuario antes de construir el prompt
   - Detecta PII usando patrones (email, documento, telefono, datos financieros)
   - Reemplaza PII detectada con placeholders (`[EMAIL_1]`, `[DOC_1]`)
   - Mantiene un mapa de reemplazo en memoria (para re-hidratacion si es necesario)
   - Registra metrica `dlp_pii_detected_pre_prompt` y log del tipo de PII (sin el dato real)

2. **Middleware DLP post-respuesta**: componente que:
   - Recibe la respuesta del LLM antes de entregarla al usuario/sistema
   - Detecta PII que el LLM pueda haber generado o replicado
   - Aplica masking segun los patrones de OBSERVABILITY-STANDARDS.md
   - Registra metrica `dlp_pii_detected_post_response`

3. **Tests GWT**:
   - Given input de usuario con email real → When se prepara el prompt → Then el email esta reemplazado por placeholder
   - Given respuesta del LLM con numero de documento → When se procesa la respuesta → Then el documento esta enmascarado
   - Given input sin PII → When se prepara el prompt → Then el input pasa sin modificacion
   - Given el middleware detecta PII → When se registra la metrica → Then `dlp_pii_detected_pre_prompt` se incrementa

---

## Arquitectura del middleware

```
Usuario → [Input] → DLP Pre-Prompt Filter → [Input sanitizado] → Prompt Builder → LLM API
                                                                                      |
Usuario ← [Respuesta filtrada] ← DLP Post-Response Filter ← [Respuesta raw] ←-------┘
```

El middleware se implementa como un **adaptador de infraestructura** (segun ADR-001), no en la capa de dominio.

---

## Ejemplo de invocacion

```
Invoca el skill DLP Guard LLM con:
- Contexto del cambio: Implementar filtro DLP para el modulo de investigacion con IA
- Especificacion: execute-research v1.0.0
- Campos PII del dominio (segun spec L1): nombre_cliente, email, nit, sector
- Proveedor LLM: {{Anthropic Claude / OpenAI / etc.}}
- Stack: {{lenguaje/framework}}

IMPORTANTE: El middleware opera en la capa de infraestructura.
No debe modificar la logica de dominio ni el caso de uso.
Usar datos sinteticos en los tests.
El filtro NO debe lanzar excepciones que bloqueen el flujo principal.
Si el filtro falla, loggear el error y dejar pasar el request SIN PII (fail-safe).
```

---

## Consideraciones de seguridad

- El mapa de reemplazo (PII real ↔ placeholder) debe vivir **solo en memoria**, nunca en logs o persistencia
- Si se necesita re-hidratacion (devolver el dato real al usuario), el mapa tiene TTL corto y se destruye al finalizar el request
- El middleware debe ser **fail-safe**: si falla la deteccion, NO bloquear el flujo. Loggear el error y continuar sin enviar PII
- Considerar que los patrones de deteccion pueden tener falsos positivos — preferir masking excesivo sobre fuga

---

## Metricas

| Metrica | Valor actual | Objetivo |
|---------|-------------|---------|
| % retrabajo | Por medir | < 15% |
| PII detectada pre-prompt | Por medir | Metrica de monitoreo (no hay objetivo de 0 — es normal detectar) |
| PII detectada post-respuesta | Por medir | < 1% de respuestas |
| Falsos positivos reportados | Por medir | < 5% de detecciones |
| Latencia agregada por el middleware | Por medir | < 50ms p95 |

---

## Historial de cambios

| Version | Fecha | Autor | Descripcion |
|---------|-------|-------|------------|
| 1.0.0 | 2026-04-09 | Equipo de Arquitectura | Creacion inicial del skill |
