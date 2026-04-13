# ADR-001: Modulos de dominio dependen de puertos, no de adaptadores

| Campo | Valor |
|-------|-------|
| **ADR ID** | ADR-001 |
| **Estado** | ACCEPTED |
| **Fecha** | 2026-04-07 |
| **Autor** | Equipo de Arquitectura |
| **PR** | Pendiente |
| **Reemplaza** | N/A |
| **Reemplazado por** | N/A |

## Contexto

En sistemas con multiples integraciones externas (bases de datos, APIs de terceros, servicios de mensajeria), el dominio de negocio tiende a acoplarse a implementaciones concretas. Esto genera fragilidad: un cambio en la base de datos o en un proveedor externo impacta directamente la logica de negocio.

Con la generacion de codigo asistida por IA, el riesgo se amplifica: la IA puede generar codigo que acopla el dominio directamente a infraestructura si no hay restricciones claras.

## Decision

Todo modulo de dominio debe depender exclusivamente de **puertos** (interfaces/abstracciones), nunca de **adaptadores** (implementaciones concretas). Los adaptadores de infraestructura viven fuera de la capa de dominio.

Esto sigue los principios de arquitectura hexagonal (Ports & Adapters).

## Alternativas evaluadas

| Opcion | Descripcion | Pros | Contras |
|--------|------------|------|---------|
| **Ports & Adapters** (elegida) | Dominio define interfaces, infraestructura las implementa | Desacoplamiento total, testeable, la IA genera dentro de limites claros | Mayor complejidad inicial, mas archivos |
| **Arquitectura en capas clasica** | Capas con dependencia directa hacia abajo | Simple de entender, menos archivos | Dominio acoplado a infraestructura, dificil de testear en aislamiento |
| **Dependencia directa con abstracciones parciales** | Solo abstraer servicios externos, no BD | Menos overhead | Inconsistente, el dominio aun depende de la capa de datos |

## Consecuencias

### Positivas
- El dominio es testeable sin infraestructura real
- La IA genera codigo dentro de limites bien definidos (solo puede usar puertos declarados)
- Cambiar una implementacion (ej: migrar de PostgreSQL a DynamoDB) no impacta el dominio
- Facilita el trabajo en paralelo entre equipos

### Negativas
- Mayor cantidad de archivos (interfaces + implementaciones)
- Curva de aprendizaje para desarrolladores no familiarizados con el patron
- Requiere disciplina en la revision de PRs para mantener la separacion

## Cumplimiento

- En especificaciones L2: la seccion "Dependencias permitidas" solo lista puertos, no implementaciones
- En code review (Gate 3): verificar que el dominio no importa modulos de infraestructura
- Linting: configurar reglas de import que prohiban dependencias de infraestructura en la capa de dominio

## ADRs relacionados

- ADR-004: Dependency Management — complementa la gestion de dependencias autorizadas
