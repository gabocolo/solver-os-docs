# /tests — Pruebas Unitarias, Integracion y Contrato

Los tests se definen **ANTES** de generar el codigo (TDD con IA).

## Estructura

| Carpeta | Tipo | Proposito |
|---------|------|-----------|
| unit/ | Unitarias | Tests GWT (Given-When-Then) basados en escenarios de spec L2 |
| integration/ | Integracion | Tests que validan interaccion entre componentes |
| contract/ | Contrato | Tests que validan contratos OpenAPI entre back y front |
| e2e/ | End-to-End | Tests de flujo completo (UI → API → DB) |
| fixtures/ | Datos de prueba | Datos sinteticos para tests (NUNCA datos reales/PII) |

## Formato GWT

Cada escenario de prueba sigue el formato Given-When-Then:

| ID | Given (Dado que) | When (Cuando) | Then (Entonces) | Prioridad |
|----|-----------------|--------------|-----------------|-----------|
| TS-001 | {{precondicion}} | {{accion}} | {{resultado}} | Alta/Media/Baja |

## Reglas

- Tests se crean **ANTES** de la implementacion (SK-002)
- Datos sinteticos siempre — nunca PII real en fixtures
- Tests de contrato validan que el backend cumple el OpenAPI
- Cobertura minima: 100% de escenarios definidos en la spec L2
- Se ejecutan automaticamente con SK-005 (pipeline de validacion PR)
