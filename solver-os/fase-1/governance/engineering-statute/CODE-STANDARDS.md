# Estandares de Codigo

Lineamientos de desarrollo aplicables a todo codigo generado (por humanos o por IA).

---

## Convenciones de nombrado

| Elemento | Convencion | Ejemplo |
|----------|-----------|---------|
| Archivos | kebab-case | `booking-service.ts`, `order-repository.py` |
| Clases | PascalCase | `BookingService`, `OrderRepository` |
| Funciones/metodos | camelCase o snake_case (segun lenguaje) | `createBooking()`, `create_booking()` |
| Variables | camelCase o snake_case (segun lenguaje) | `bookingId`, `booking_id` |
| Constantes | UPPER_SNAKE_CASE | `MAX_RETRY_COUNT`, `DEFAULT_TIMEOUT_MS` |
| Interfaces/puertos | Prefijo con `I` o sufijo con `Port` | `IBookingRepository`, `BookingRepositoryPort` |

> **Nota:** Adaptar al lenguaje del proyecto. Lo importante es la **consistencia** dentro del repositorio.

{{Agregar convenciones especificas del lenguaje del proyecto aqui}}

---

## Estructura del proyecto

Seguir la separacion de capas segun ADR-001 (Ports & Adapters):

```
src/
├── domain/           Logica de negocio pura, entidades, reglas
│   ├── entities/     Modelos de dominio
│   ├── ports/        Interfaces (puertos)
│   └── services/     Servicios de dominio
├── application/      Casos de uso, orquestacion
│   └── use-cases/    Un archivo por caso de uso
├── infrastructure/   Adaptadores, integraciones externas
│   ├── adapters/     Implementaciones de puertos
│   ├── persistence/  Repositorios concretos
│   └── external/     Clientes de APIs externas
└── api/              Capa de presentacion (controllers, handlers)
    ├── controllers/
    └── dtos/         Data Transfer Objects
```

> **Regla:** El dominio NUNCA importa de infrastructure. Solo de ports.

---

## Tamano de PRs

- **Maximo recomendado:** 400 lineas de cambio
- Si el PR supera 400 lineas, dividirlo en cambios mas pequenos
- Cada PR debe ser trazable a una especificacion (L2 o L3)
- Preferir 10 PRs de 40 lineas sobre 1 PR de 400 lineas

---

## Manejo de errores

- Usar tipos de error definidos en la especificacion L2
- No capturar excepciones genericas sin re-lanzar o manejar
- Cada error debe tener un codigo, un mensaje y un contexto suficiente para debugging
- No exponer detalles internos al cliente (stack traces, queries, paths)

---

## Logging

- Usar logging estructurado (JSON preferido)
- Campos obligatorios: `timestamp`, `level`, `message`, `correlationId`, `service`
- Niveles: `ERROR` para fallos, `WARN` para situaciones inesperadas, `INFO` para flujo normal, `DEBUG` para desarrollo
- No loggear datos sensibles (passwords, tokens, PII)
- Ver [OBSERVABILITY-STANDARDS.md](../engineering-statute/OBSERVABILITY-STANDARDS.md) para mas detalle

---

## Documentacion en codigo

- No agregar comentarios obvios que repitan lo que el codigo ya dice
- Si agregar comentarios cuando la logica no es autoevidente (decisiones de negocio complejas, workarounds)
- Cada modulo debe tener un README breve si su proposito no es evidente por nombre
- Los contratos de API deben estar documentados con OpenAPI/Swagger

---

## Revision de codigo generado por IA

Cuando la IA genera codigo, el reviewer debe verificar:
- [ ] El codigo esta en la capa correcta
- [ ] Las dependencias usadas coinciden con la spec L2
- [ ] No hay logica de negocio inventada (no presente en la spec)
- [ ] No hay dependencias fantasma (alucinaciones de la IA)
- [ ] Los patrones siguen los ADRs vigentes
- [ ] El error handling usa los tipos de la spec

---

{{Agregar estandares especificos del lenguaje/framework del proyecto aqui}}

---

*Referencia: ADR-001 (Ports & Adapters), Framework AI-Driven Engineering v1.0*
