# ADR-002: Idempotencia obligatoria en operaciones criticas

| Campo | Valor |
|-------|-------|
| **ADR ID** | ADR-002 |
| **Estado** | ACCEPTED |
| **Fecha** | 2026-04-07 |
| **Autor** | Equipo de Arquitectura |
| **PR** | Pendiente |
| **Reemplaza** | N/A |
| **Reemplazado por** | N/A |

## Contexto

En sistemas distribuidos, las operaciones de escritura pueden ejecutarse mas de una vez debido a reintentos de red, timeouts o fallos parciales. Sin idempotencia, esto puede causar duplicacion de datos, cobros dobles o estados inconsistentes (ej: overbooking en un sistema de reservas).

Con la generacion de codigo por IA, si no se exige idempotencia explicitamente en la especificacion, la IA puede generar operaciones que no la implementan.

## Decision

Todas las operaciones criticas de escritura (creacion, actualizacion de estado, transacciones financieras) deben ser **idempotentes**. Cada operacion critica debe aceptar un **idempotency key** proporcionado por el cliente.

## Alternativas evaluadas

| Opcion | Descripcion | Pros | Contras |
|--------|------------|------|---------|
| **Idempotency key del cliente** (elegida) | El cliente envia un identificador unico por operacion; el servidor detecta duplicados | Control total del cliente, deteccion fiable de duplicados, estandar de industria | Requiere almacenamiento de keys, logica adicional en el servidor |
| **Idempotency key generada por el servidor** | El servidor genera un ID previo a la operacion (two-step) | No requiere cambio en el cliente | Doble round-trip, complejidad adicional, el cliente puede perder el ID |
| **Sin idempotencia, deduplicacion a nivel de BD** | Constraints unicos en la base de datos | Simple de implementar | No cubre todos los casos, errores poco claros, no funciona con operaciones distribuidas |

## Consecuencias

### Positivas
- Previene duplicacion de datos y estados inconsistentes
- Seguro ante reintentos de red y timeouts
- La especificacion L2 declara explicitamente el campo idempotency key
- La IA genera codigo que incluye la validacion de idempotencia

### Negativas
- Requiere almacenamiento de keys procesadas (con TTL)
- Logica adicional en cada operacion critica
- El cliente debe generar y gestionar las keys

## Cumplimiento

- En especificaciones L2: la seccion "Execution Requirements" debe declarar `idempotency: required` con la definicion del key
- En tests (Gate 2): incluir escenarios de reintento con el mismo idempotency key
- En code review (Gate 3): verificar que operaciones criticas implementan la validacion

## ADRs relacionados

- ADR-003: Traceability — la idempotency key contribuye a la trazabilidad de operaciones
