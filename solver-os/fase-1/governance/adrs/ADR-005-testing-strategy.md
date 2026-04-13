# ADR-005: Tests GWT definidos antes de la implementacion

| Campo | Valor |
|-------|-------|
| **ADR ID** | ADR-005 |
| **Estado** | ACCEPTED |
| **Fecha** | 2026-04-07 |
| **Autor** | Equipo de Arquitectura |
| **PR** | Pendiente |
| **Reemplaza** | N/A |
| **Reemplazado por** | N/A |

## Contexto

En el ciclo de iteracion del framework (TDD con IA), la calidad del codigo generado depende directamente de que tan bien definidos estan los criterios de aceptacion. Si los tests se definen despues de la implementacion, se corre el riesgo de que los tests se adapten al codigo generado en lugar de validar el comportamiento esperado.

La especificacion L2 incluye escenarios de prueba en formato GWT (Given-When-Then). Estos escenarios deben traducirse en tests antes de generar la implementacion.

## Decision

Los tests deben ser definidos **antes** de la implementacion, siguiendo el formato **GWT (Given-When-Then)** derivado de los escenarios de la especificacion L2. El ciclo de iteracion es:

```
Tests (GWT) → Implementacion minima → Refactor asistido → Revision → Quality gates
```

Los tipos de prueba obligatorios son:
- **Tests unitarios**: basados en GWT de la spec
- **Tests de contrato**: validan la API segun el contrato de la spec L2
- **SAST**: analisis estatico de seguridad (automatizado en pipeline)
- **SCA**: analisis de composicion de software (automatizado en pipeline)

## Alternativas evaluadas

| Opcion | Descripcion | Pros | Contras |
|--------|------------|------|---------|
| **TDD desde spec (GWT primero)** (elegida) | Tests derivados de la spec antes de implementar | Tests validan comportamiento esperado (no el codigo), alta cobertura, la IA genera contra criterios claros | Requiere specs completas con escenarios, mayor esfuerzo inicial |
| **Tests despues de implementacion** | Generar codigo primero, tests despues | Mas rapido para comenzar, se puede iterar sobre lo generado | Tests se adaptan al codigo (no al comportamiento), menor cobertura, sesgo de confirmacion |
| **Tests generados por IA sin spec** | Pedir a la IA que genere tests basados en el codigo | Minimo esfuerzo, la IA infiere el comportamiento | Tests alucinados, no validan reglas de negocio reales, sin trazabilidad |

## Consecuencias

### Positivas
- Los tests validan el comportamiento esperado, no el codigo generado
- Alta cobertura desde el inicio
- Los escenarios GWT de la spec son trazables a los tests
- La IA genera implementacion que debe pasar tests ya definidos
- Facilita la revision: si los tests pasan y estan bien definidos, el codigo es correcto

### Negativas
- Requiere que las especificaciones L2 incluyan escenarios completos
- Mayor tiempo de preparacion antes de la generacion
- Curva de aprendizaje en la escritura de buenos escenarios GWT

## Cumplimiento

- En especificaciones L2: la seccion "Test Scenarios" es obligatoria con formato GWT
- En Gate 2: los tests unitarios y de contrato deben pasar, SAST y SCA deben ejecutarse
- En Gate 3: el reviewer verifica que los tests cubren los escenarios de la spec
- Metrica: cobertura de tests vs escenarios definidos en la spec

## ADRs relacionados

- ADR-003: Traceability — los tests son parte de la cadena de trazabilidad (spec → tests → codigo)
- ADR-004: Dependency Management — los mocks en tests solo deben cubrir dependencias autorizadas
