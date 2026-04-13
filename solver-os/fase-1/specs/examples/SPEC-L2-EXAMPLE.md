# Especificacion de Sistema (Nivel 2)

| Campo | Valor |
|-------|-------|
| **Spec ID** | SPEC-L2-create-order |
| **Nivel** | L2 — Sistema |
| **Version** | 1.0.0 |
| **Parent L1** | SPEC-L1-order-mgmt v1.0.0 |
| **Estado** | APPROVED |
| **Creado por** | Equipo de Arquitectura |
| **Fecha de creacion** | 2026-04-07 |
| **Aprobado por** | Lider Tecnico |
| **Fecha de aprobacion** | 2026-04-08 |
| **PR** | #002 |

---

## Sistema

**Nombre:** Order API

## Caso de uso

**Nombre:** CreateOrder

**Descripcion:** Crea un nuevo pedido validando stock, calculando totales y persistiendo la transaccion. Opcionalmente aplica un descuento si se proporciona un codigo valido.

**Agregado principal:** Order (raiz del agregado Order, definido en SPEC-L1-order-mgmt)

---

## Contrato tecnico

### Input

| Campo | Tipo | Requerido | Descripcion |
|-------|------|-----------|------------|
| customerId | string (UUID) | Si | ID del cliente que realiza el pedido |
| items | OrderItemInput[] | Si | Lista de productos y cantidades |
| items[].productId | string (UUID) | Si | ID del producto |
| items[].quantity | integer (> 0) | Si | Cantidad solicitada |
| discountCode | string | No | Codigo de descuento opcional |
| idempotencyKey | string (UUID) | Si | Key de idempotencia para prevenir duplicados |

### Output

| Campo | Tipo | Descripcion |
|-------|------|------------|
| orderId | string (UUID) | ID unico del pedido creado |
| status | string | Estado del pedido: "CONFIRMED" |
| items | OrderItemOutput[] | Items con precios capturados |
| items[].productId | string (UUID) | ID del producto |
| items[].quantity | integer | Cantidad |
| items[].unitPrice | decimal | Precio unitario al momento |
| items[].subtotal | decimal | quantity * unitPrice |
| subtotal | decimal | Suma de subtotales |
| discountAmount | decimal | Monto de descuento aplicado (0 si no hay) |
| total | decimal | subtotal - discountAmount |
| createdAt | string (ISO 8601) | Fecha de creacion |

---

## Eventos de dominio

### OrderCreated

| Campo | Valor |
|-------|-------|
| **Nombre** | OrderCreated |
| **Evento L1** | OrderCreated (SPEC-L1-order-mgmt) |
| **Canal** | EventBus (EventPublisherPort) |
| **Tipo de entrega** | At-least-once |

**Payload:**

| Campo | Tipo | Descripcion |
|-------|------|------------|
| orderId | string (UUID) | ID unico del pedido creado |
| customerId | string (UUID) | ID del cliente |
| items | OrderItemEvent[] | Items con productId, quantity, unitPrice |
| total | decimal | Total final del pedido |
| createdAt | string (ISO 8601) | Fecha de creacion |

> Este evento se publica DESPUES de la persistencia exitosa del pedido (transaccion confirmada).

---

## Tipos de error

| Codigo de error | HTTP Status | Descripcion | Condicion de disparo |
|----------------|-------------|------------|---------------------|
| INSUFFICIENT_STOCK | 409 Conflict | Stock insuficiente para uno o mas productos | product.stockQuantity < item.quantity |
| CUSTOMER_NOT_FOUND | 404 Not Found | Cliente no existe | customerId no encontrado |
| PRODUCT_NOT_FOUND | 404 Not Found | Producto no existe | productId no encontrado en catalogo |
| INVALID_QUANTITY | 400 Bad Request | Cantidad invalida | item.quantity <= 0 |
| INVALID_DISCOUNT | 400 Bad Request | Codigo de descuento invalido o expirado | discount no existe o validUntil < now |
| DUPLICATE_REQUEST | 409 Conflict | Request duplicado detectado | idempotencyKey ya procesado |

---

## Dependencias permitidas

> **REGLA (ADR-004):** Solo estas dependencias pueden ser usadas.

| Dependencia | Proposito | Tipo de interfaz |
|------------|----------|-----------------|
| InventoryService | Validar stock y reservar unidades | Puerto (InventoryPort) |
| ProductCatalogService | Obtener precios actuales de productos | Puerto (ProductCatalogPort) |
| DiscountService | Validar y calcular descuento | Puerto (DiscountPort) |
| CustomerService | Validar existencia del cliente | Puerto (CustomerPort) |
| OrderRepository | Persistir el pedido | Puerto (OrderRepositoryPort) |
| EventBus | Publicar evento OrderCreated | Puerto (EventPublisherPort) |

---

## Requisitos de ejecucion

| Requisito | Valor | Notas |
|-----------|-------|-------|
| **Latencia maxima** | 500 ms (p95) | Incluye validaciones y persistencia |
| **Idempotencia** | Requerida | Key: idempotencyKey (UUID del cliente) |
| **Atomicidad** | Transaccion atomica | Reserva de stock + persistencia en una transaccion |
| **Observabilidad** | Ver abajo | |

### Observabilidad requerida

- **Logs:** Pedido creado (INFO), validacion fallida (WARN), error inesperado (ERROR)
- **Trazas:** span por operacion principal (create-order), spans por cada llamada a dependencia
- **Metricas:** order_creation_duration_ms, order_creation_total, order_creation_error_total
- **Alertas:** error rate > 5% en 5 min, latencia p95 > 500ms en 5 min

---

## Restricciones de arquitectura

| ADR | Titulo | Impacto en esta spec |
|-----|--------|---------------------|
| ADR-001 | Ports & Adapters | CreateOrder depende de puertos (interfaces), no de implementaciones concretas |
| ADR-002 | Idempotency | Requiere idempotencyKey y deteccion de duplicados |
| ADR-003 | Traceability | Commits deben referenciar SPEC-L2-create-order v1.0.0 |
| ADR-004 | Dependency Management | Solo las 6 dependencias listadas estan autorizadas |
| ADR-005 | Testing Strategy | Tests GWT deben crearse antes de implementar |

---

## Escenarios de prueba (GWT)

| ID | Given (Dado que) | When (Cuando) | Then (Entonces) | Prioridad |
|----|-----------------|--------------|-----------------|-----------|
| TS-001 | Un cliente valido con items que tienen stock suficiente | Se crea el pedido sin descuento | El pedido se crea con status CONFIRMED, total = suma de subtotales | Alta |
| TS-002 | Un cliente valido con items y un codigo de descuento valido | Se crea el pedido con descuento | El pedido se crea con discountAmount > 0 y total ajustado | Alta |
| TS-003 | Un producto sin stock suficiente | Se intenta crear el pedido | Se retorna error INSUFFICIENT_STOCK | Alta |
| TS-004 | Un customerId que no existe | Se intenta crear el pedido | Se retorna error CUSTOMER_NOT_FOUND | Alta |
| TS-005 | Un productId que no existe | Se intenta crear el pedido | Se retorna error PRODUCT_NOT_FOUND | Media |
| TS-006 | Un item con quantity = 0 | Se intenta crear el pedido | Se retorna error INVALID_QUANTITY | Media |
| TS-007 | Un codigo de descuento expirado | Se crea el pedido con ese codigo | Se retorna error INVALID_DISCOUNT | Media |
| TS-008 | La misma idempotencyKey de un request previo | Se envia el mismo request | Se retorna error DUPLICATE_REQUEST (o el resultado original) | Alta |

---

## Especificaciones relacionadas

| Spec ID | Nivel | Version | Relacion |
|---------|-------|---------|----------|
| SPEC-L1-order-mgmt | L1 | 1.0.0 | Parent — dominio de gestion de pedidos |
| SPEC-L3-add-discount-code | L3 | 1.1.0 | Child — agregar campo de descuento (ejemplo) |

---

## Historial de cambios

| Version | Fecha | Autor | Descripcion del cambio |
|---------|-------|-------|----------------------|
| 1.0.0 | 2026-04-07 | Equipo de Arquitectura | Version inicial del caso de uso CreateOrder |
