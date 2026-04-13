# Fase 1: Fundamentos del Framework AI-Driven Engineering

**Periodo:** Dias 1-30  
**Objetivo:** Establecer las bases de gobernanza, especificaciones y primeros aceleradores para la adopcion del framework Solver OS en Tech&Solve.

---

## Estructura de carpetas

```
fase-1/
├── specs/                    Especificaciones de 3 niveles (dominio, sistema, cambio)
│   ├── templates/            Templates reutilizables L1, L2, L3
│   └── examples/             Ejemplos con dominio "Gestion de Pedidos"
├── governance/               Artefactos de gobernanza
│   ├── adrs/                 Architecture Decision Records (transversales)
│   ├── quality-gates/        Definicion de los 3 gates obligatorios
│   └── engineering-statute/  Estandares de codigo, seguridad y observabilidad
├── prompts/                  Templates y ejemplos de prompts estructurados
│   └── examples/             Ejemplos de invocacion de specs
├── accelerators/             Aceleradores y framework de agentizacion
└── skills/                   Catalogo de skills y templates de definicion
    └── templates/            Templates para definir nuevos skills
```

---

## Mapa de trazabilidad

```
ADRs (transversales) ──────────────────────────────────┐
                                                        ↓
L1 (dominio) → L2 (sistema) → L3 (cambio) → Prompt → Generacion → Tests → PR → Merge → Release
                    ↑                           ↑
Engineering Statute ┘                           │
(code/security/obs)          Skills (estandarizan la ejecucion)
```

Cada artefacto es trazable:
- **L1 → L2 → L3**: jerarquia de especificaciones (negocio → tecnico → cambio)
- **ADRs → Specs**: las especificaciones L2 referencian ADRs relevantes
- **Specs → Prompts**: el prompt invoca un spec versionado, nunca contiene reglas directamente
- **Engineering Statute → Todo**: estandares transversales que aplican a toda generacion
- **Skills → Prompts**: los skills estandarizan como se construyen los prompts por tipo de tarea

---

## Glosario

| Termino | Definicion |
|---------|-----------|
| **Spec (Especificacion)** | Documento versionado que define que construir y bajo que reglas. Existen 3 niveles |
| **L1 (Dominio)** | Especificacion de negocio: objetivo, entidades, reglas, politicas |
| **L2 (Sistema)** | Especificacion tecnica: contrato, dependencias, errores, requisitos de ejecucion |
| **L3 (Cambio)** | Especificacion de cambio incremental sobre un L2 existente |
| **ADR** | Architecture Decision Record: decision tecnica transversal documentada |
| **Quality Gate** | Punto de control obligatorio antes de avanzar en el flujo |
| **Skill** | Patron estandarizado de ejecucion para un tipo de tarea (build o workflow) |
| **Acelerador** | Base reutilizable y adaptable a multiples dominios/industrias |
| **Agentizacion** | Proceso de automatizar tareas repetitivas mediante agentes de IA |
| **GWT** | Given-When-Then: formato de criterios de aceptacion para tests |
| **Semver** | Versionado semantico: MAJOR.MINOR.PATCH |
| **SAST** | Static Application Security Testing |
| **SCA** | Software Composition Analysis |

---

## Convenciones de nombrado

| Artefacto | Convencion | Ejemplo |
|-----------|-----------|---------|
| Spec (desde template) | `SPEC-L{n}-{dominio}-v{X.Y.Z}.md` | `SPEC-L1-order-mgmt-v1.0.0.md` |
| ADR | `ADR-{NNN}-{titulo-kebab}.md` | `ADR-003-traceability.md` |
| Skill ID | `SK-{NNN}` | `SK-001` |
| Regla de negocio | `BR-{NNN}` | `BR-001` |
| Escenario de test | `TS-{NNN}` | `TS-001` |
| Versiones | Semver `X.Y.Z` | `1.0.0` |
| Placeholders | `{{nombre}}` | `{{domain-name}}` |
| Opciones | `[OPCION_A | OPCION_B]` | `[DRAFT | APPROVED]` |

---

## Orden de lectura recomendado

1. **Governance** → entender las reglas del juego (ADRs, quality gates, estandares)
2. **Specs** → entender el modelo de especificaciones (L1 → L2 → L3)
3. **Prompts** → entender como invocar specs para generar codigo
4. **Skills** → entender como estandarizar tipos de tarea
5. **Accelerators** → entender como escalar y agentizar

---

## Regla fundamental

> **Primero especificar, luego invocar, despues generar y validar.**  
> Sin gobernanza, la IA acelera el caos.

---

*Framework AI-Driven Engineering v1.0 - Tech&Solve - Fase 1: Fundamentos*
