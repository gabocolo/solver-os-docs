# Ejemplo: Prompt para generar tests GWT

Este es un ejemplo de prompt que invoca la especificacion SPEC-L2-create-order v1.0.0 para generar los tests antes de la implementacion (ADR-005: TDD).

---

## Prompt

```
## Rol
Actua como un QA engineer con experiencia en TDD y pruebas unitarias.

## Invocacion de especificacion
Aplica la especificacion SPEC-L2-create-order version 1.0.0.
[Adjuntar contenido completo de SPEC-L2-EXAMPLE.md]

## Tarea
Genera los tests unitarios para los escenarios de prueba definidos en la spec
(TS-001 a TS-008), usando el formato Given-When-Then.

Cada test debe:
- Mockear las dependencias autorizadas (puertos)
- Validar el comportamiento esperado segun la spec
- Verificar los errores correctos en escenarios negativos

## Escenarios a cubrir
De la seccion "Escenarios de prueba (GWT)" de la spec:
- TS-001: Pedido exitoso sin descuento
- TS-002: Pedido exitoso con descuento
- TS-003: Error por stock insuficiente
- TS-004: Error por cliente no encontrado
- TS-005: Error por producto no encontrado
- TS-006: Error por cantidad invalida
- TS-007: Error por descuento expirado
- TS-008: Error por request duplicado (idempotencia)

## Reglas
- Solo mockear las dependencias listadas en "Dependencias permitidas" de la spec.
- Los errores deben corresponder a los "Tipos de error" de la spec.
- Los tests deben ser independientes entre si.
- NO inventes escenarios que no esten en la spec.
- Los mocks deben respetar las interfaces (puertos) definidas.

## Contexto de gobernanza adjunto
- ADR-004: Dependency Management (solo dependencias autorizadas)
- ADR-005: Testing Strategy (tests antes de implementacion)

## Restricciones de salida
- Tests ubicados en tests/unit/use-cases/
- Un test por escenario GWT
- Nomenclatura: should_[resultado]_when_[condicion]
- Diff < 400 lineas
```

---

## Que esperar de la IA

Con este prompt, la IA deberia generar:
1. Un archivo de tests con 8 escenarios
2. Mocks para los 6 puertos autorizados
3. Assertions que validen outputs y errores segun la spec
4. Tests independientes y ejecutables
5. Nomenclatura clara que mapea a los IDs de escenarios (TS-001 a TS-008)

---

## Nota importante

Estos tests se generan **ANTES** de la implementacion. El objetivo es que la implementacion (generada despues con otro prompt) pase estos tests.
