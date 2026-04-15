# Patrones DLP — Spec Agent Portal

## Patrones pre-prompt (sanitizar antes de enviar a LLM)

| Patron | Regex | Clasificacion | Accion |
|--------|-------|--------------|--------|
| Email | `[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}` | Sensible (PII) | Reemplazar por `[EMAIL_REDACTED]` |
| Cedula colombiana | `\b\d{6,10}\b` (en contexto de documento) | Sensible (PII) | Reemplazar por `[DOC_REDACTED]` |
| Telefono | `(\+57)?\s?\d{3}\s?\d{3}\s?\d{4}` | Sensible (PII) | Reemplazar por `[PHONE_REDACTED]` |
| Tarjeta de credito | `\b\d{4}[\s-]?\d{4}[\s-]?\d{4}[\s-]?\d{4}\b` | Sensible (PII) | Bloquear envio |
| API Key | `(sk-[a-zA-Z0-9]{20,})|(ghp_[a-zA-Z0-9]{36})` | Sensible | Bloquear envio |
| JWT Token | `eyJ[a-zA-Z0-9_-]+\.eyJ[a-zA-Z0-9_-]+\.[a-zA-Z0-9_-]+` | Sensible | Bloquear envio |
| Connection String | `(Server|Host)=[^;]+;.*Password=[^;]+` | Sensible | Bloquear envio |

## Patrones post-response (filtrar respuesta del LLM antes de mostrar)

| Patron | Clasificacion | Accion |
|--------|--------------|--------|
| Email | Sensible (PII) | Enmascarar: `u***@example.com` |
| Numeros de documento | Sensible (PII) | Enmascarar: `***456` |
| Direcciones IP privadas | Interno | Alertar pero no bloquear |

## Patrones en logs (filtro Serilog)

| Campo | Clasificacion | Tratamiento |
|-------|--------------|-------------|
| User.email | Sensible | Masking: `u***@domain.com` |
| User.userId | Interno | No exponer en APIs publicas |
| Spec.content | Confidencial | Sanitizar antes de loggear (solo metadata) |
| ADR.content | Confidencial | Sanitizar antes de enviar a LLM |

## Implementacion

Estos patrones se implementan mediante los siguientes prompts:

| Prompt | Skill | Que genera |
|--------|-------|-----------|
| [PROMPT-08](../../prompts/PROMPT-08-data-masking.md) | SK-006 | Filtro Serilog PII, serializador seguro API, anonimizador para no-prod |
| [PROMPT-09](../../prompts/PROMPT-09-dlp-guard-llm.md) | SK-007 | Middleware DLP bidireccional pre/post-prompt para Claude API |

> Referencia: ADR-P005 (DLP Filter Architecture), ADR-006 (Data Classification)
