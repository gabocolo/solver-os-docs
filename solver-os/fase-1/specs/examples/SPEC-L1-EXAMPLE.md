# Especificacion de Dominio (Nivel 1)

| Campo | Valor |
|-------|-------|
| **Spec ID** | SPEC-L1-order-mgmt |
| **Nivel** | L1 — Dominio |
| **Version** | 1.0.0 |
| **Estado** | APPROVED |
| **Creado por** | Equipo de Arquitectura |
| **Fecha de creacion** | 2026-04-07 |
| **Aprobado por** | Lider Tecnico |
| **Fecha de aprobacion** | 2026-04-08 |
| **PR** | #001 |

---

## Dominio

**Nombre:** Gestion de Pedidos (Order Management)

### Limite del contexto (Bounded Context)

**Responsabilidad:** Creacion, validacion, calculo y gestion del ciclo de vida de pedidos de productos.

**Fuera de alcance:** No gestiona inventario (Inventory es dominio separado), no gestiona clientes (Customer es dominio separado), no procesa pagos.

| Contexto externo | Relacion | Descripcion |
|-----------------|----------|-------------|
| Inventory | Upstream | Ellos son duenos del stock; nosotros consultamos disponibilidad via InventoryPort |
| Product Catalog | Upstream | Ellos son duenos de precios y productos; nosotros consumimos via ProductCatalogPort |
| Customer | Upstream | Ellos son duenos de datos de cliente; nosotros validamos existencia via CustomerPort |
| Notifications | Downstream | Nosotros publicamos eventos (OrderCreated); ellos consumen para notificar |

---

## Objetivo de negocio

Permitir a los clientes crear, consultar y gestionar pedidos de productos. El sistema debe validar disponibilidad de inventario, calcular totales con impuestos y descuentos, y garantizar que no se vendan productos sin stock (no overselling).

---

## Modelo de dominio

### Agregados

| Agregado (raiz) | Entidades internas | Value Objects | Invariantes |
|-----------------|-------------------|---------------|-------------|
| Order | OrderItem | Money, OrderStatus, DiscountApplication | El total siempre refleja items * precio - descuento. No se puede crear si no hay stock suficiente. |
| Product | — | Money | El stock nunca puede ser negativo. |
| Customer | — | — | El email debe ser unico. |

### Detalle de entidades

| Entidad | Tipo | Agregado | Descripcion | Atributos clave |
|---------|------|----------|------------|----------------|
| Order | Raiz de agregado | Order | Pedido realizado por un cliente | orderId, customerId, items, status, total, createdAt |
| OrderItem | Entidad interna | Order | Linea de producto dentro de un pedido, solo accesible via Order | productId, quantity, unitPrice, subtotal |
| Product | Raiz de agregado | Product | Producto disponible para venta (dominio externo, referenciado por ID) | productId, name, price, stockQuantity |
| Customer | Raiz de agregado | Customer | Cliente que realiza el pedido (dominio externo, referenciado por ID) | customerId, name, email, shippingAddress |
| Money | Value Object | Order / Product | Representa un valor monetario, inmutable | amount, currency |
| OrderStatus | Value Object | Order | Estado del pedido, inmutable | value (PENDING, CONFIRMED, SHIPPED, DELIVERED, CANCELLED) |
| DiscountApplication | Value Object | Order | Resultado de aplicar un descuento al pedido | discountCode, type (percentage/fixed), value, amountApplied |

---

## Reglas de negocio

| ID | Regla | Descripcion | Justificacion |
|----|-------|------------|---------------|
| BR-001 | No overselling | No se puede crear un pedido si algun producto no tiene stock suficiente | Evitar compromisos que no se puedan cumplir |
| BR-002 | Cantidad positiva | Cada item del pedido debe tener cantidad mayor a cero | Evitar pedidos vacios o con datos invalidos |
| BR-003 | Precio al momento | El precio del producto se captura al momento de crear el pedido (no cambia si el catalogo se actualiza despues) | Consistencia financiera |
| BR-004 | Descuento unico | Solo se puede aplicar un codigo de descuento por pedido | Simplificar calculo y evitar acumulacion no autorizada |
| BR-005 | Descuento vigente | Un codigo de descuento solo es valido si no ha expirado | Evitar uso de descuentos vencidos |

---

## Eventos de dominio

| Evento | Disparado cuando | Datos relevantes | Consumidores potenciales |
|--------|-----------------|-----------------|------------------------|
| OrderCreated | Se confirma un pedido exitosamente | orderId, customerId, items, total | Notifications, Analytics |
| OrderCancelled | Se cancela un pedido existente | orderId, customerId, reason | Inventory (liberar stock), Notifications |

---

## Politicas de negocio

- Un pedido cancelado no puede ser reactivado; se debe crear uno nuevo
- Los pedidos con estado "shipped" no pueden ser modificados
- El historico de pedidos debe mantenerse por minimo 5 anos (cumplimiento regulatorio)

---

## Validaciones

| Validacion | Condicion | Error si falla |
|-----------|----------|---------------|
| Stock suficiente | Para cada item: product.stockQuantity >= item.quantity | INSUFFICIENT_STOCK |
| Cliente existe | El customerId debe corresponder a un cliente registrado | CUSTOMER_NOT_FOUND |
| Productos validos | Todos los productId deben existir en el catalogo | PRODUCT_NOT_FOUND |
| Cantidad valida | Cada item.quantity > 0 | INVALID_QUANTITY |
| Descuento valido | Si se aplica, el codigo debe existir y no estar expirado | INVALID_DISCOUNT |

---

## Lenguaje ubicuo del dominio (Glosario)

| Termino | Definicion | Nombre en codigo |
|---------|-----------|-----------------|
| Overselling | Vender mas unidades de un producto de las que hay en stock | `OversellException` |
| Order status | Estados del pedido: PENDING, CONFIRMED, SHIPPED, DELIVERED, CANCELLED | `OrderStatus` (enum) |
| Discount code | Codigo alfanumerico que aplica un descuento (porcentaje o monto fijo) | `DiscountCode` |
| Order item | Linea de producto dentro de un pedido | `OrderItem` |
| Money | Valor monetario con monto y moneda | `Money` (value object) |

---

## Especificaciones relacionadas

| Spec ID | Nivel | Version | Descripcion |
|---------|-------|---------|------------|
| SPEC-L2-create-order | L2 | 1.0.0 | Caso de uso: crear pedido |
| SPEC-L2-get-order | L2 | 1.0.0 | Caso de uso: consultar pedido (futuro) |

---

## Historial de cambios

| Version | Fecha | Autor | Descripcion del cambio |
|---------|-------|-------|----------------------|
| 1.0.0 | 2026-04-07 | Equipo de Arquitectura | Version inicial del dominio |
