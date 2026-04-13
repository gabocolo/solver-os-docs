# Ejemplo: Prompt para generar caso de uso

Este es un ejemplo de prompt que invoca la especificacion SPEC-L2-create-order v1.0.0 para generar la implementacion del caso de uso.

---

## Prompt

```
## Rol
Actua como un desarrollador senior con experiencia en arquitectura hexagonal.

## Invocacion de especificacion
Aplica la especificacion SPEC-L2-create-order version 1.0.0.
[Adjuntar contenido completo de SPEC-L2-EXAMPLE.md]

## Tarea
Implementa el caso de uso CreateOrder que:
1. Valide la existencia del cliente (CustomerPort)
2. Valide stock disponible para cada item (InventoryPort)
3. Obtenga precios actuales de productos (ProductCatalogPort)
4. Si hay discountCode, valide y calcule descuento (DiscountPort)
5. Persista el pedido en una transaccion atomica (OrderRepositoryPort)
6. Publique evento OrderCreated (EventPublisherPort)
7. Retorne OrderConfirmation segun spec

## Entrada
CreateOrderRequest segun la seccion "Input" de la spec:
- customerId (UUID, requerido)
- items[] (productId + quantity, requerido)
- discountCode (string, opcional)
- idempotencyKey (UUID, requerido)

## Salida esperada
OrderConfirmation segun la seccion "Output" de la spec:
- orderId, status, items con precios, subtotal, discountAmount, total, createdAt

## Reglas
- Todas las reglas de negocio provienen de la spec (BR-001 a BR-005 del L1).
- Solo usar las 6 dependencias listadas en "Dependencias permitidas".
- Los errores deben corresponder a los tipos de la spec (INSUFFICIENT_STOCK, etc.).
- Implementar idempotencia con idempotencyKey (ADR-002).
- El dominio depende de puertos, no de adaptadores (ADR-001).
- NO inventes servicios, reglas ni dependencias fuera de la spec.

## Contexto de gobernanza adjunto
- ADR-001: Ports & Adapters
- ADR-002: Idempotency
- ADR-004: Dependency Management
- CODE-STANDARDS.md (estructura de capas)

## Restricciones de salida
- Codigo ubicado en src/application/use-cases/
- Interfaces (puertos) en src/domain/ports/
- Diff < 400 lineas
- Incluir los tests para TS-001, TS-003, TS-004 y TS-008
```

---

## Que esperar de la IA

Con este prompt, la IA deberia generar:
1. La clase/funcion `CreateOrder` en la capa de aplicacion
2. Las interfaces de los puertos referenciados (si no existen)
3. Los tests para los escenarios indicados
4. Codigo que respeta la separacion de capas (ADR-001)
5. Manejo de idempotencia (ADR-002)
6. Solo las dependencias autorizadas (ADR-004)
