# Spec Agent Portal — Backend (.NET 10)

Este archivo define las reglas, convenciones y contexto obligatorio para cualquier generacion de codigo asistida por IA en el repositorio backend del Spec Agent Portal.

> **Regla fundamental**: Primero especificar, luego invocar, despues generar y validar.

---

## 1. Stack tecnologico

| Componente | Tecnologia |
|------------|------------|
| Runtime | .NET 10, C# |
| Framework web | ASP.NET Core Minimal APIs |
| ORM | Entity Framework Core |
| Base de datos | PostgreSQL 16 |
| Cache | Redis 7 |
| IA/LLM | Semantic Kernel |
| Mensajeria | Azure Service Bus |

---

## 2. Arquitectura: Hexagonal (Ports & Adapters) — ADR-001

La arquitectura sigue el patron hexagonal. Toda dependencia externa se consume a traves de puertos (interfaces), nunca directamente.

```
src/
  Domain/
    Entities/          -- Entidades de dominio y agregados
    ValueObjects/      -- Value objects inmutables
    Ports/             -- Interfaces (puertos) que definen contratos
    Events/            -- Eventos de dominio
  Application/
    Services/          -- Servicios de aplicacion (casos de uso)
    UseCases/          -- Handlers de casos de uso
    DTOs/              -- Objetos de transferencia de datos
    Validators/        -- Validadores de entrada
  Infrastructure/
    Adapters/          -- Implementaciones de puertos (adaptadores)
    Persistence/       -- Configuraciones de EF Core, migrations
    ExternalServices/  -- Clientes HTTP, integraciones externas
    Messaging/         -- Azure Service Bus producers/consumers
    Cache/             -- Implementacion Redis
  Api/
    Endpoints/         -- Minimal API endpoints
    Middleware/        -- Middleware (correlationId, error handling, DLP)
    Auth/              -- Autenticacion y autorizacion
    Filters/           -- Filtros de request/response
```

### Reglas de arquitectura

- El dominio NO depende de infraestructura ni de la capa Api
- Los adaptadores implementan los puertos definidos en Domain/Ports/
- Las mutaciones de estado pasan por la raiz del agregado
- Los eventos de dominio se publican DESPUES de la persistencia exitosa
- Un caso de uso opera sobre un solo agregado principal

---

## 3. ADRs relevantes

| ADR | Nombre | Impacto en este repo |
|-----|--------|---------------------|
| ADR-001 | Ports & Adapters | Toda dependencia externa via puertos/interfaces |
| ADR-002 | Idempotencia | Idempotency key obligatoria en operaciones de escritura |
| ADR-003 | Trazabilidad | correlationId en todas las operaciones, propagado en headers |
| ADR-004 | Dependencias | Solo dependencias autorizadas en la spec — NO inventar nuevas |
| ADR-005 | Testing | Tests GWT definidos antes de implementar, cobertura minima |
| ADR-006 | DLP | Clasificacion de datos, masking en logs, datos sinteticos siempre |

---

## 4. Convenciones de codigo

### Naming

- **Clases, metodos, propiedades**: PascalCase (`BookingService`, `CreateBooking`, `UserId`)
- **Parametros, variables locales**: camelCase (`bookingRequest`, `correlationId`)
- **Interfaces (puertos)**: prefijo `I` (`IBookingRepository`, `IAvailabilityPort`)
- **Constantes**: PascalCase (`MaxRetryCount`)

### Patrones obligatorios

- **async/await obligatorio** en toda operacion I/O — nunca `.Result` ni `.Wait()`
- **correlationId** en todas las operaciones — propagado via middleware y headers HTTP
- **CancellationToken** en todos los metodos async
- **Result pattern** para manejar errores de negocio (no excepciones para flujo de control)
- **Records** para DTOs y value objects cuando sea posible

### Ejemplo de endpoint Minimal API

```csharp
app.MapPost("/api/bookings", async (
    CreateBookingRequest request,
    ICreateBookingUseCase useCase,
    CancellationToken cancellationToken) =>
{
    var result = await useCase.ExecuteAsync(request, cancellationToken);
    return result.Match(
        success => Results.Created($"/api/bookings/{success.Id}", success),
        error => Results.Problem(error.ToProblemDetails()));
});
```

---

## 5. Dependencias autorizadas

**Solo se pueden usar las dependencias listadas en la especificacion del caso de uso.**

- NO inventar servicios, librerias o paquetes NuGet fuera de la spec
- NO agregar dependencias sin aprobacion previa
- Si una tarea requiere una dependencia nueva, debe solicitarse como cambio en la spec

---

## 6. Reglas DLP y proteccion de datos — ADR-006

- **Nunca PII real** en codigo, tests, fixtures ni seeds
- **Datos sinteticos siempre** — usar datos generados para pruebas y desarrollo
- **Masking en logs obligatorio** — campos clasificados como Sensible (PII) deben enmascararse
  - Ejemplo: `userId` → `u***123` en logs
- **No exponer IDs internos en respuestas de error**
- **Clasificacion de datos** definida en la spec L1 — respetarla siempre
- **Filtro DLP en middleware** — sanitizar respuestas y logs antes de salida

### Ejemplo de masking en log

```csharp
// CORRECTO
_logger.LogInformation("Booking created for user {MaskedUserId}", MaskPii(userId));

// INCORRECTO — nunca hacer esto
_logger.LogInformation("Booking created for user {UserId}", userId);
```

---

## 7. Testing

### Formato obligatorio: Given-When-Then (GWT)

- Framework: **xUnit**
- Mocking: **NSubstitute**
- Los tests se definen ANTES de la implementacion (TDD)
- Escenarios de prueba vienen de la especificacion

### Estructura de tests

```csharp
[Fact]
public async Task CreateBooking_WhenRoomIsAvailable_ShouldReturnConfirmation()
{
    // Given
    var request = new CreateBookingRequest(UserId: "synthetic-user-001", ...);
    _availabilityPort.CheckAvailability(...).Returns(AvailabilityResult.Available(...));

    // When
    var result = await _useCase.ExecuteAsync(request, CancellationToken.None);

    // Then
    result.IsSuccess.Should().BeTrue();
    result.Value.BookingId.Should().NotBeEmpty();
}
```

### Reglas de testing

- **Datos sinteticos** — nunca PII real en tests
- Cubrir escenarios de exito Y de error definidos en la spec
- Tests de contrato para validar contratos API
- Nombres descriptivos que reflejen el escenario GWT

---

## 8. Observabilidad

| Pilar | Herramienta | Configuracion |
|-------|------------|---------------|
| Logs | Serilog (structured logging) | JSON, correlationId en todos los logs, PII masking |
| Trazas | OpenTelemetry | Distribuido, propagacion de contexto |
| Visualizacion | Seq | Dashboard centralizado |

### Reglas de logging

- Logs estructurados en formato JSON
- correlationId presente en TODOS los log entries
- Niveles: Information para flujo normal, Warning para degradacion, Error para fallos
- **Nunca loguear PII en texto plano** — usar filtros de masking (ADR-006)

---

## 9. Ubicacion de especificaciones

Las especificaciones que gobiernan este repositorio se encuentran en:

```
solver-os/AgenteSpec/specs/
```

### Regla de invocacion

El prompt NO contiene las reglas de negocio. El prompt INVOCA la especificacion:

```
Invocacion: Aplica la especificacion [nombre] v[X.Y.Z]
```

Las reglas, dependencias autorizadas, criterios de aceptacion y salidas esperadas provienen EXCLUSIVAMENTE del spec versionado. No inventar servicios, reglas ni dependencias fuera del spec.

---

## 10. Quality gates antes de merge

- [ ] Spec versionada y aprobada (Gate 1)
- [ ] Tests GWT pasan
- [ ] SAST + SCA sin hallazgos criticos
- [ ] DLP Scan — sin PII en codigo, logs con masking
- [ ] PR menor a 400 lineas
- [ ] Revision de senior obligatoria (Gate 3)
- [ ] correlationId presente en nuevas operaciones
- [ ] Datos sinteticos en todos los tests y fixtures
