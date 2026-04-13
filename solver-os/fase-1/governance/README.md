# Gobernanza

La gobernanza define los limites dentro de los cuales la IA puede operar. Sin estos limites, la IA genera caos mas rapido.

## Componentes

### ADRs (Architecture Decision Records)
Ubicacion: `adrs/`

Decisiones tecnicas **transversales** que todo modulo debe cumplir. Cada ADR:
- Esta enumerado (`ADR-NNN`)
- Evalua minimo 2 alternativas con pros/cons
- Tiene estado: `[PROPOSED | ACCEPTED | DEPRECATED | SUPERSEDED]`
- Requiere PR y revision para cambios

Los ADRs definen la **regla transversal**. Las **decisiones tecnicas locales** se derivan de un ADR y son especificas de un caso (ej: "el modulo booking implementa los puertos X, Y, Z").

### Quality Gates
Ubicacion: `quality-gates/`

3 puntos de control obligatorios en el flujo:
1. **Gate 1**: Spec versionada y aprobada → antes de generar codigo
2. **Gate 2**: Tests + SAST + SCA pasan → antes de revision humana
3. **Gate 3**: Revision senior obligatoria → antes del merge

### Engineering Statute (Estatuto de Ingenieria)
Ubicacion: `engineering-statute/`

Estandares transversales divididos en:
- **Codigo**: convenciones, estructura, PRs, documentacion
- **Seguridad**: SAST/SCA, dependencias, secrets, prompt injection
- **Observabilidad**: logs, trazas, metricas, alertas

## Quien mantiene la gobernanza

| Artefacto | Responsable | Frecuencia de revision |
|-----------|------------|----------------------|
| ADRs | Arquitecto / Lider tecnico | Cada sprint o cuando hay nueva evidencia |
| Quality Gates | Arquitecto + DevOps | Trimestral o cuando hay cambio de pipeline |
| Engineering Statute | Equipo tecnico | Trimestral o cuando hay nuevo estandar |

## Como alimenta la gobernanza a los demas artefactos

- Los **ADRs** son referenciados en las especificaciones L2 (seccion "Architecture Constraints")
- Los **estandares** se adjuntan como contexto de gobernanza en los prompts
- Los **quality gates** validan que todo el flujo se cumpla antes de merge
