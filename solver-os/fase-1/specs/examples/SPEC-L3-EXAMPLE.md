# Especificacion de Cambio (Nivel 3)

| Campo | Valor |
|-------|-------|
| **Spec ID** | SPEC-L3-add-discount-code |
| **Nivel** | L3 — Cambio |
| **Version** | 1.1.0 |
| **Parent L2** | SPEC-L2-create-order v1.0.0 |
| **Estado** | APPROVED |
| **Creado por** | Equipo de Desarrollo |
| **Fecha de creacion** | 2026-04-10 |
| **Aprobado por** | Lider Tecnico |
| **Fecha de aprobacion** | 2026-04-11 |
| **PR** | #003 |

---

## Resumen del cambio

Agregar soporte para un codigo de descuento opcional en la creacion de pedidos, con validacion contra el servicio de descuentos.

---

## Motivacion

El equipo de marketing requiere que los clientes puedan aplicar codigos promocionales al crear pedidos. Actualmente el sistema no soporta descuentos. Este cambio agrega el campo `discountCode` como opcional y la logica de validacion y calculo.

---

## Impacto en contrato

### Campos agregados

| Campo | Tipo | Requerido | Descripcion |
|-------|------|-----------|------------|
| discountCode | string | No | Codigo de descuento promocional opcional |

### Campos modificados

| Campo | Cambio | Antes | Despues |
|-------|--------|-------|---------|
| total | Calculo actualizado | subtotal | subtotal - discountAmount |

### Campos eliminados

Sin cambios.

---

## Impacto en comportamiento

- Si `discountCode` esta presente en el request:
  1. Validar contra DiscountService que el codigo existe y no esta expirado
  2. Calcular el monto de descuento segun el tipo (porcentaje o monto fijo)
  3. Aplicar descuento al total: `total = subtotal - discountAmount`
- Si `discountCode` no esta presente:
  - El flujo continua sin cambios, `discountAmount = 0`
- Si el codigo es invalido o expirado:
  - Retornar error `INVALID_DISCOUNT` (no crear el pedido)

---

## Compatibilidad hacia atras

| Aspecto | Valor |
|---------|-------|
| **Tipo de cambio** | Non-breaking |
| **Requiere migracion** | No |
| **Notas de migracion** | El campo discountCode es opcional. Requests existentes sin este campo siguen funcionando igual |

---

## Dependencias afectadas

### Nuevas dependencias

| Dependencia | Proposito | Tipo |
|------------|----------|------|
| DiscountService | Validar codigo y calcular monto de descuento | Puerto (DiscountPort) |

### Cambios en dependencias existentes

Sin cambios en dependencias existentes.

---

## Escenarios de prueba afectados

### Tests existentes que requieren actualizacion

| ID del test original | Cambio necesario |
|---------------------|-----------------|
| TS-001 | Verificar que sin discountCode, discountAmount = 0 y total = subtotal |

### Nuevos escenarios de prueba (GWT)

| ID | Given (Dado que) | When (Cuando) | Then (Entonces) | Prioridad |
|----|-----------------|--------------|-----------------|-----------|
| TS-009 | Un pedido valido con discountCode de tipo porcentaje (10%) y subtotal de $100 | Se crea el pedido con ese discountCode | discountAmount = $10, total = $90 | Alta |
| TS-010 | Un pedido valido con discountCode de tipo monto fijo ($15) y subtotal de $100 | Se crea el pedido con ese discountCode | discountAmount = $15, total = $85 | Alta |
| TS-011 | Un discountCode que no existe | Se intenta crear el pedido con ese codigo | Se retorna error INVALID_DISCOUNT | Alta |
| TS-012 | Un discountCode con validUntil en el pasado | Se intenta crear el pedido con ese codigo | Se retorna error INVALID_DISCOUNT | Media |
| TS-013 | Un pedido valido sin discountCode | Se crea el pedido | discountAmount = 0, total = subtotal (comportamiento sin cambios) | Alta |

---

## Trazabilidad

```
L1: SPEC-L1-order-mgmt v1.0.0 (dominio: gestion de pedidos)
  └── L2: SPEC-L2-create-order v1.0.0 (caso de uso: crear pedido)
        └── L3: SPEC-L3-add-discount-code v1.1.0 (este cambio: agregar descuento)
              └── Commits esperados:
                    - feat: add DiscountPort interface (spec: SPEC-L3-add-discount-code v1.1.0)
                    - feat: add discount validation to CreateOrder use case
                    - test: add GWT tests for discount code scenarios (TS-009 to TS-013)
```

---

## Historial de cambios

| Version | Fecha | Autor | Descripcion del cambio |
|---------|-------|-------|----------------------|
| 1.1.0 | 2026-04-10 | Equipo de Desarrollo | Agregar soporte para codigo de descuento opcional |
