# Definicion de Skill

| Campo | Valor |
|-------|-------|
| **ID** | SK-003 |
| **Nombre** | Generacion de contrato API (OpenAPI) |
| **Tipo** | Build |
| **Version** | 1.0.0 |
| **Owner** | Lider tecnico / Arquitecto |
| **Fase del framework** | Fase 2 — Especificacion dirigida |
| **Estado** | ACTIVE |

---

## Descripcion

Genera un contrato OpenAPI 3.1.0 en formato YAML a partir de la seccion "Contrato tecnico" de una especificacion de nivel 2 (L2). El skill toma las tablas de Input/Output, los tipos de error, los requisitos de autenticacion y la clasificacion de datos de la spec, y produce un archivo OpenAPI completo, validado y listo para ser usado como referencia en la generacion de codigo.

El contrato generado utiliza como base la plantilla `OPENAPI-CONTRACT-TEMPLATE.yaml` ubicada en `solver-os/fase-1/specs/templates/` y la completa con los datos concretos de la spec L2.

---

## Cuando usar

- Cuando se aprueba una spec L2 nueva que define un contrato tecnico con endpoints API
- Cuando una spec L3 introduce cambios en el contrato existente (nuevo endpoint, nuevo campo, nuevo error)
- Cuando se necesita formalizar un contrato API para validacion de tests de contrato en CI/CD
- Cuando la spec L2 tiene campos clasificados como Sensible (PII) y se requiere documentar las anotaciones `x-data-classification` en el contrato

---

## Cuando NO usar

- Para APIs internas entre microservicios que no requieren contrato formal (comunicacion via eventos, gRPC con proto ya definido)
- Si la spec L2 no esta aprobada y versionada — el contrato NO debe generarse sin spec aprobada (Gate 1)
- Para modificar un contrato sin spec L3 que respalde el cambio — todo cambio debe ser trazable
- Para generar codigo de implementacion — eso es responsabilidad de SK-001

---

## Input requerido

| Input | Descripcion | Obligatorio |
|-------|------------|-------------|
| Especificacion L2 versionada y aprobada | Spec que contiene el contrato tecnico, tablas de Input/Output, tipos de error, dependencias y requisitos | Si |
| Clasificacion de datos de la spec L1 | Clasificacion de cada campo (Publico, Interno, Confidencial, Sensible) segun ADR-006 | Si |
| Plantilla OPENAPI-CONTRACT-TEMPLATE.yaml | Plantilla base ubicada en `solver-os/fase-1/specs/templates/` | Si |
| Requisitos de autenticacion de la spec L2 | Tipo de auth (JWT Bearer, API Key, OAuth2) y proveedor | Si |
| ADR-006 | Decision de clasificacion de datos y DLP | Si (si hay campos PII) |

---

## Output esperado

1. **Archivo OpenAPI 3.1.0 YAML** completo con:
   - `info` con titulo, version y descripcion trazables a la spec L2
   - `servers` con URLs de ambientes (dev, staging, prod)
   - `paths` con un endpoint por caso de uso definido en la spec L2
   - `components/schemas` con schemas de request y response mapeados desde las tablas Input/Output
   - `components/schemas/ErrorResponse` con los codigos de error de la seccion "Tipos de error"
   - `components/securitySchemes` segun los requisitos de autenticacion
   - Extension `x-data-classification` en cada campo con clasificacion de datos segun spec L1

2. **Mapeo documentado** entre la spec L2 y el contrato:
   - Cada campo de la tabla Input → propiedad del schema de Request
   - Cada campo de la tabla Output → propiedad del schema de Response
   - Cada tipo de error → respuesta HTTP con codigo y ejemplo
   - Cada campo PII → anotacion `x-data-classification: sensible`

3. **Datos sinteticos** en todos los ejemplos del contrato — nunca datos reales

---

## Pasos de ejecucion

1. **Leer la spec L2** — identificar las secciones: Contrato tecnico, Input, Output, Tipos de error, Dependencias permitidas, Requisitos
2. **Leer la clasificacion de datos de la spec L1** — identificar campos PII y su nivel de clasificacion
3. **Cargar la plantilla** `OPENAPI-CONTRACT-TEMPLATE.yaml` como base
4. **Mapear Input a Request schema**:
   - Cada fila de la tabla Input se convierte en una propiedad del schema
   - Campos obligatorios van en el array `required`
   - Campos opcionales se documentan sin incluirlos en `required`
   - Agregar `x-data-classification` segun la clasificacion de la spec L1
5. **Mapear Output a Response schema**:
   - Cada fila de la tabla Output se convierte en una propiedad del schema
   - No exponer IDs internos en respuestas de error
6. **Mapear Tipos de error a respuestas HTTP**:
   - Cada error de la spec L2 se asigna a un codigo HTTP apropiado (400, 404, 409, etc.)
   - El `code` del ErrorResponse debe coincidir con el codigo de error de la spec
   - Los mensajes de error NO deben contener datos sensibles
7. **Configurar seguridad**:
   - Definir `securitySchemes` segun el tipo de autenticacion de la spec L2
   - Aplicar seguridad a nivel de operacion o global segun corresponda
8. **Agregar anotaciones DLP**:
   - Campos Sensible (PII): `x-data-classification: sensible`
   - Campos Confidencial: `x-data-classification: confidencial`
   - Campos Interno: `x-data-classification: interno`
   - Campos Publico: `x-data-classification: publico`
9. **Generar ejemplos sinteticos** para cada schema — nunca usar datos reales
10. **Validar** que el YAML generado sea OpenAPI 3.1.0 valido

---

## Reglas

- El contrato SOLO puede incluir endpoints, campos y errores definidos en la spec L2 — no inventar
- Todos los ejemplos deben usar datos sinteticos, nunca datos reales (PII)
- Los mensajes de error NO deben exponer IDs internos del sistema ni datos sensibles
- La version del contrato (`info.version`) debe coincidir con la version de la spec L2
- Si la spec L2 requiere idempotencia, el contrato debe incluir el header `X-Idempotency-Key`
- Campos PII deben tener la extension `x-data-classification: sensible` — es obligatorio si la spec L1 los clasifica como Sensible
- El contrato generado debe ser validable con herramientas estandar de OpenAPI (Swagger Editor, Spectral, etc.)

---

## Ejemplo de invocacion

```
Invoca el skill Generacion de contrato API con:
- Contexto del cambio: Crear contrato OpenAPI para el modulo de booking
- Especificacion: booking-system v1.1.0
- Casos de uso a incluir: CreateBooking
- Input segun spec:
    - userId (string, obligatorio, Sensible/PII)
    - roomId (string, obligatorio, Interno)
    - dates (objeto con checkIn/checkOut, obligatorio, Publico)
    - couponCode (string, opcional, Interno)
- Output segun spec:
    - id (string, Interno)
    - status (string, Publico)
    - totalPrice (number, Publico)
    - createdAt (datetime, Publico)
- Tipos de error segun spec: ROOM_NOT_AVAILABLE, INVALID_DATES, INVALID_COUPON, UNAUTHORIZED
- Autenticacion: JWT Bearer via AuthService
- Requisitos: idempotencia obligatoria

IMPORTANTE: Usar la plantilla OPENAPI-CONTRACT-TEMPLATE.yaml como base.
No inventar campos, errores ni dependencias fuera de la spec L2.
Todos los ejemplos deben usar datos sinteticos.
Campos PII deben incluir x-data-classification: sensible.
```

---

## Metricas

| Metrica | Valor actual | Objetivo |
|---------|-------------|---------|
| % retrabajo | Por medir | < 10% |
| Contratos validados sin errores de sintaxis | Por medir | 100% |
| Cobertura de campos PII con x-data-classification | Por medir | 100% |
| Tiempo de revision promedio del contrato | Por medir | < 15 min |

---

## Historial de cambios

| Version | Fecha | Autor | Descripcion |
|---------|-------|-------|------------|
| 1.0.0 | 2026-04-14 | Equipo de Arquitectura | Creacion inicial del skill |
