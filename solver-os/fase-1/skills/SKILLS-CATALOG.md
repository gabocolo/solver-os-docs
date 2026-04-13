# Catalogo de Skills

| Campo | Valor |
|-------|-------|
| **Version del catalogo** | 1.1.0 |
| **Ultima actualizacion** | 2026-04-09 |
| **Owner** | Equipo de Arquitectura |

---

## Skills registrados

### SK-001: Generacion de codigo de caso de uso

| Campo | Valor |
|-------|-------|
| **ID** | SK-001 |
| **Nombre** | Generacion de codigo de caso de uso |
| **Tipo** | Build |
| **Version** | 1.0.0 |
| **Owner** | Lider tecnico |
| **Fase del framework** | Fase 3 — Generacion controlada |
| **Estado** | ACTIVE |

**Descripcion:** Genera la implementacion de un caso de uso a partir de una especificacion L2 aprobada, respetando la arquitectura hexagonal (puertos y adaptadores).

**Cuando usar:** Cuando se necesita implementar un nuevo caso de uso que tiene spec L2 aprobada.

**Cuando NO usar:** Para cambios incrementales (usar SPEC-L3 directamente), para generar infraestructura o adaptadores.

**Metricas:**
| Metrica | Valor actual | Objetivo |
|---------|-------------|---------|
| % retrabajo | Por medir | < 15% |
| Tiempo revision promedio | Por medir | < 30 min |
| Hallazgos de seguridad | Por medir | 0 criticos |

---

### SK-002: Generacion de tests GWT

| Campo | Valor |
|-------|-------|
| **ID** | SK-002 |
| **Nombre** | Generacion de tests GWT |
| **Tipo** | Build |
| **Version** | 1.0.0 |
| **Owner** | QA Lead |
| **Fase del framework** | Fase 4 — Validacion (se ejecuta antes de Fase 3) |
| **Estado** | ACTIVE |

**Descripcion:** Genera tests unitarios en formato Given-When-Then a partir de los escenarios de prueba definidos en la especificacion L2. Se ejecuta ANTES de generar la implementacion (TDD).

**Cuando usar:** Siempre que se va a implementar un caso de uso nuevo o un cambio (L3). Los tests se crean primero.

**Cuando NO usar:** Para tests de integracion o tests E2E (requieren otro skill o enfoque).

**Metricas:**
| Metrica | Valor actual | Objetivo |
|---------|-------------|---------|
| % retrabajo | Por medir | < 10% |
| Cobertura de escenarios spec | Por medir | 100% |
| Hallazgos asociados | Por medir | 0 |

---

### SK-003: Generacion de contrato API

| Campo | Valor |
|-------|-------|
| **ID** | SK-003 |
| **Nombre** | Generacion de contrato API (OpenAPI) |
| **Tipo** | Build |
| **Version** | 1.0.0 |
| **Owner** | Lider tecnico |
| **Fase del framework** | Fase 2 — Especificacion (apoyo a creacion de spec L2) |
| **Estado** | ACTIVE |

**Descripcion:** Genera un archivo OpenAPI/Swagger a partir de la seccion de contrato tecnico de una especificacion L2, incluyendo inputs, outputs y tipos de error.

**Cuando usar:** Cuando se tiene una spec L2 aprobada y se necesita generar el contrato formal para documentacion o validacion de contrato.

**Cuando NO usar:** Como reemplazo de la spec L2 (el contrato OpenAPI complementa, no reemplaza).

**Metricas:**
| Metrica | Valor actual | Objetivo |
|---------|-------------|---------|
| % retrabajo | Por medir | < 10% |
| Precision del contrato vs spec | Por medir | 100% |

---

### SK-004: Revision automatizada de spec

| Campo | Valor |
|-------|-------|
| **ID** | SK-004 |
| **Nombre** | Revision automatizada de especificacion |
| **Tipo** | Workflow |
| **Version** | 1.0.0 |
| **Owner** | Arquitecto |
| **Fase del framework** | Fase 2 — Especificacion (Gate 1) |
| **Estado** | ACTIVE |

**Descripcion:** Revisa automaticamente que una especificacion (L1, L2 o L3) cumpla con la estructura requerida: campos obligatorios completos, versionado correcto, referencias a ADRs validas, escenarios de prueba definidos.

**Cuando usar:** Como paso previo a la revision humana de una spec. Reduce el tiempo de revision al detectar problemas estructurales antes.

**Cuando NO usar:** Como reemplazo de la revision humana (este skill detecta estructura, no calidad de contenido).

**Metricas:**
| Metrica | Valor actual | Objetivo |
|---------|-------------|---------|
| Specs rechazadas por estructura | Por medir | < 5% |
| Tiempo de revision reducido | Por medir | -30% |

---

### SK-005: Pipeline de validacion PR

| Campo | Valor |
|-------|-------|
| **ID** | SK-005 |
| **Nombre** | Pipeline de validacion de PR |
| **Tipo** | Workflow |
| **Version** | 1.0.0 |
| **Owner** | DevOps / Lider tecnico |
| **Fase del framework** | Fase 4 — Validacion (Gates 2 y 3) |
| **Estado** | ACTIVE |

**Descripcion:** Configura y ejecuta el pipeline de validacion automatizada para PRs: tests unitarios, tests de contrato, SAST, SCA, **escaneo DLP (deteccion de PII y secretos)**, verificacion de tamano de PR y trazabilidad.

**Cuando usar:** En cada PR que se crea. Debe ejecutarse automaticamente.

**Cuando NO usar:** N/A — siempre se ejecuta.

**Pasos DLP en el pipeline (segun ADR-006):**
1. `detect-secrets scan` — detecta secretos hardcoded en el diff
2. Escaneo de PII en archivos modificados (emails, documentos, datos financieros en strings literales, fixtures o constantes)
3. Validacion de que logs no expongan campos clasificados como Sensible en la spec
4. Si se detecta PII o secreto → **bloquear merge** (hallazgo critico)

**Metricas:**
| Metrica | Valor actual | Objetivo |
|---------|-------------|---------|
| % PRs que pasan Gate 2 al primer intento | Por medir | > 80% |
| Tiempo promedio de pipeline | Por medir | < 10 min |
| Hallazgos criticos bloqueados | Por medir | 100% bloqueados |
| PII detectada en PRs | Por medir | 0 por sprint |
| Secretos detectados en PRs | Por medir | 0 por sprint |

---

### SK-006: Data Masking en codigo generado

| Campo | Valor |
|-------|-------|
| **ID** | SK-006 |
| **Nombre** | Data Masking en codigo generado |
| **Tipo** | Build |
| **Version** | 1.0.0 |
| **Owner** | Lider tecnico |
| **Fase del framework** | Fase 3 — Generacion controlada |
| **Estado** | ACTIVE |

**Descripcion:** Genera la implementacion de filtros de data masking y anonimizacion para campos clasificados como Sensible (PII) en la especificacion. Produce filtros para logs, respuestas API y ambientes no productivos segun ADR-006 y estandares de observabilidad.

**Cuando usar:** Cuando una spec L2 tiene campos clasificados como Sensible (PII) y se necesita protegerlos en logs, respuestas API o ambientes no productivos.

**Cuando NO usar:** Para campos Publico/Interno, para cifrado en reposo (infraestructura), para validacion de input de usuario.

**Definicion completa:** [SK-006-data-masking.md](SK-006-data-masking.md)

**Metricas:**
| Metrica | Valor actual | Objetivo |
|---------|-------------|---------|
| % retrabajo | Por medir | < 10% |
| PII detectada en logs post-implementacion | Por medir | 0 |
| Cobertura de campos PII de la spec | Por medir | 100% |

---

### SK-007: DLP Guard para prompts y respuestas LLM

| Campo | Valor |
|-------|-------|
| **ID** | SK-007 |
| **Nombre** | DLP Guard para prompts y respuestas LLM |
| **Tipo** | Build |
| **Version** | 1.0.0 |
| **Owner** | Lider tecnico / Arquitecto |
| **Fase del framework** | Fase 3 — Generacion controlada |
| **Estado** | ACTIVE |

**Descripcion:** Genera un middleware/proxy DLP bidireccional que sanitiza prompts antes de enviarlos a un LLM (pre-prompt) y filtra respuestas antes de entregarlas al usuario (post-respuesta). Aplica cuando la aplicacion integra LLMs en runtime.

**Cuando usar:** Cuando la aplicacion envia datos de usuario a un LLM externo, procesa input de usuarios en prompts, o persiste/muestra respuestas de LLM.

**Cuando NO usar:** Para proteccion de prompts en desarrollo (cubierto por SECURITY-STANDARDS.md), si la app no integra LLMs en runtime, para prompt injection (concern separado).

**Definicion completa:** [SK-007-dlp-guard-llm.md](SK-007-dlp-guard-llm.md)

**Metricas:**
| Metrica | Valor actual | Objetivo |
|---------|-------------|---------|
| % retrabajo | Por medir | < 15% |
| PII detectada post-respuesta | Por medir | < 1% de respuestas |
| Latencia agregada por middleware | Por medir | < 50ms p95 |

---

## Resumen del catalogo

| ID | Nombre | Tipo | Fase | Owner | Estado |
|----|--------|------|------|-------|--------|
| SK-001 | Generacion de caso de uso | Build | Fase 3 | Lider tecnico | ACTIVE |
| SK-002 | Generacion de tests GWT | Build | Fase 4 (pre-Fase 3) | QA Lead | ACTIVE |
| SK-003 | Generacion de contrato API | Build | Fase 2 | Lider tecnico | ACTIVE |
| SK-004 | Revision automatizada de spec | Workflow | Fase 2 | Arquitecto | ACTIVE |
| SK-005 | Pipeline de validacion PR (+ DLP) | Workflow | Fase 4 | DevOps | ACTIVE |
| SK-006 | Data Masking en codigo generado | Build | Fase 3 | Lider tecnico | ACTIVE |
| SK-007 | DLP Guard para prompts/respuestas LLM | Build | Fase 3 | Lider tecnico / Arquitecto | ACTIVE |

---

## Evolucion del catalogo

- Revisar metricas cada sprint
- Si un skill tiene > 20% retrabajo sostenido → redisenar
- Si un skill no se usa en 2 sprints → evaluar deprecacion
- Nuevos skills se agregan via PR con template de SKILL-DEFINITION-TEMPLATE.md
