# Prompts estructurados

El prompt es el mecanismo de comunicacion con la IA para generar codigo. En este framework, el prompt **no contiene reglas de negocio** — el prompt **invoca** una especificacion versionada.

---

## Principio clave

> La especificacion es la que manda, el prompt es el que invoca.

El prompt es quirurgico: invoca un spec, define la tarea y establece los limites. Todo lo demas (reglas, dependencias, criterios de aceptacion, salidas esperadas) proviene del spec.

---

## Contexto minimo

**NO pasar todo el repositorio como contexto.** Solo el minimo necesario:

| Tipo | Que incluir | Ejemplo |
|------|------------|---------|
| **Gobernanza** | ADRs relevantes al cambio, estandares aplicables | ADR-001, CODE-STANDARDS.md |
| **Ejecutable** | Spec versionado, contrato API, escenarios de prueba | SPEC-L2-create-order v1.0.0 |

---

## Anti-patrones en prompts

| Anti-patron | Por que es malo |
|------------|----------------|
| Prompt gigante que pide un modulo completo | Codigo incontrolable, imposible de revisar |
| Incluir reglas de negocio en el prompt | Se desincroniza con la spec, no es trazable |
| Contexto ambiguo | La IA pierde correlacion entre tokens, alucina |
| No referenciar version del spec | No hay trazabilidad |
| Pedir a la IA que "use lo que crea necesario" | Alucinacion de dependencias garantizada |

---

## Contenido

- `STRUCTURED-PROMPT-TEMPLATE.md` — Template para construir prompts
- `examples/` — Ejemplos de prompts que invocan specs
