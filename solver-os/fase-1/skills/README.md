# Skills

Un **skill** estandariza COMO ejecutar una clase de tarea. La especificacion define QUE construir; el skill define el patron de ejecucion.

> El skill no reemplaza la especificacion, la vuelve **operable**.

---

## Tipos de skills

| Tipo | Descripcion | Ejemplos |
|------|------------|---------|
| **Build** | Generan artefactos de codigo | Generacion de casos de uso, tests, contratos API |
| **Workflow** | Automatizan flujos de trabajo | Revision automatizada de specs, pipelines de validacion |

---

## Ciclo de vida de un skill

```
Diseno → Versionado → Medicion → Evolucion
```

1. **Diseno y versionado**: definir owner tecnico, versionar, definir criterios de uso
2. **Mapeo en fases**: ubicar en que fase del framework opera cada skill
3. **Medicion**: metricas de efectividad (retrabajo, tiempo de revision, hallazgos)
4. **Evolucion**: si no mejora calidad o costo → redisenar o eliminar (basado en metricas)

---

## Invocacion de un skill

```
Invoca el skill [nombre] con:
- Contexto del cambio: [referencia al cambio]
- Especificacion: [spec-id] version [X.Y.Z]
```

El skill estandariza el prompt, pero el contexto y la spec siempre los proporciona el desarrollador.

---

## Regla de catalogo

- Si un skill no mejora la calidad o no reduce el costo → redisena o elimina
- La decision se basa en **metricas**, no en intuicion
- Cada skill tiene un owner responsable de su evolucion

---

## Contenido

- `SKILLS-CATALOG.md` — Catalogo inicial con 5 skills base
- `templates/SKILL-DEFINITION-TEMPLATE.md` — Template para definir nuevos skills
