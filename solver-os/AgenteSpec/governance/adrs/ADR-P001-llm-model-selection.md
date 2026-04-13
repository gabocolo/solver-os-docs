# ADR-P001: Seleccion de modelos LLM por tipo de tarea

| Campo | Valor |
|-------|-------|
| **Estado** | ACCEPTED |
| **Fecha** | 2026-04-09 |
| **Derivado de** | Framework AI-Driven Engineering v1.1, Seccion 11 (Costos) |
| **Decisor** | Equipo de Arquitectura |

---

## Contexto

El Spec Agent Portal usa LLMs para generar especificaciones (L1, L2, L3), generar ADRs asistidos y validar contenido. Cada tipo de tarea tiene distinta complejidad y requiere diferente nivel de razonamiento. Usar el modelo mas potente para todo es costoso e innecesario; usar el modelo mas barato para todo sacrifica calidad.

---

## Decision

Asignar modelos LLM segun el tipo de tarea, optimizando el balance costo/calidad:

| Tarea | Modelo | Justificacion |
|-------|--------|---------------|
| Generar spec L1 (dominio completo) | **Claude Opus** (`claude-opus-4-6`) | Razonamiento profundo: entidades, reglas, clasificacion de datos, multiples casos de uso |
| Generar spec L2 (sistema tecnico) | **Claude Sonnet** (`claude-sonnet-4-6`) | Tarea acotada por L1 parent: contrato, dependencias, errores |
| Generar spec L3 (cambio incremental) | **Claude Sonnet** (`claude-sonnet-4-6`) | Cambio puntual sobre L2 existente |
| Regenerar seccion individual | **Claude Sonnet** (`claude-sonnet-4-6`) | Contexto acotado a una seccion |
| Generar ADR asistido | **Claude Sonnet** (`claude-sonnet-4-6`) | Documento estructurado con alternativas |
| Validacion de estructura (SK-004) | **Claude Haiku** (`claude-haiku-4-5-20251001`) | Tarea mecanica: verificar campos, formato, reglas |
| DLP post-response (deteccion PII) | **Regex + reglas** (sin LLM) | No requiere LLM — patrones deterministicos |

---

## Alternativas evaluadas

### Alternativa 1: Opus para todo
- **Pro**: Maxima calidad en toda generacion
- **Contra**: Costo ~10x mayor que Sonnet, latencia ~2x mayor
- **Descartada**: El costo no se justifica para tareas acotadas (L2, L3)

### Alternativa 2: Sonnet para todo
- **Pro**: Balance uniforme, menor costo
- **Contra**: L1 requiere razonamiento profundo que Sonnet puede no resolver bien (entidades complejas, reglas cruzadas)
- **Descartada**: Calidad insuficiente para L1 en dominios complejos

### Alternativa 3: Un solo modelo configurable por proyecto
- **Pro**: Flexibilidad total
- **Contra**: Introduce decision manual innecesaria, riesgo de usar modelo incorrecto
- **Descartada**: La seleccion automatica por tipo de tarea es mas segura

---

## Consecuencias

### Positivas
- Optimizacion de costos: Opus solo cuando la complejidad lo requiere
- Latencia predecible por tipo de tarea
- Sin decision manual del usuario — el sistema selecciona automaticamente

### Negativas
- Si Opus no esta disponible, L1 se degrada a Sonnet (menor calidad)
- Requiere monitorear metricas de calidad por modelo para ajustar
- Haiku para validacion puede no detectar problemas semanticos sutiles

### Metricas de seguimiento
- `llm_tokens_consumed_total` por modelo
- `llm_generation_quality_score` (futuro: calificacion manual de revisores)
- Costo mensual por modelo
