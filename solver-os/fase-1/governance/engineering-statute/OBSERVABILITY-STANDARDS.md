# Estandares de Observabilidad

Lineamientos para logs, trazas y metricas. La observabilidad es un requisito no funcional priorizado por defecto en el framework.

---

## Los 3 pilares

### 1. Logs (registro de eventos)

**Formato:** Estructurado (JSON)

**Campos obligatorios:**

| Campo | Descripcion | Ejemplo |
|-------|------------|---------|
| `timestamp` | Fecha y hora ISO 8601 | `2026-04-07T14:30:00.000Z` |
| `level` | Nivel de severidad | `INFO`, `WARN`, `ERROR`, `DEBUG` |
| `message` | Descripcion del evento | `Booking created successfully` |
| `correlationId` | ID de correlacion para trazar flujos | `uuid-v4` |
| `service` | Nombre del servicio | `booking-api` |
| `traceId` | ID de traza distribuida (si aplica) | `abc123` |

**Niveles de logging:**

| Nivel | Cuando usar | Ejemplo |
|-------|------------|---------|
| `ERROR` | Fallos que requieren atencion | Excepcion no recuperable, servicio externo no disponible |
| `WARN` | Situaciones inesperadas pero manejables | Reintento de operacion, cache miss inesperado |
| `INFO` | Flujo normal de la aplicacion | Operacion completada, request recibido |
| `DEBUG` | Informacion detallada para desarrollo | Valores de variables, pasos intermedios |

**Que NO loggear (segun ADR-006):**
- Passwords, tokens, API keys
- Datos personales (PII): emails, numeros de documento, telefonos, direcciones
- Informacion financiera personal: numeros de tarjeta, cuentas bancarias
- Datos de salud o datos sensibles segun Ley 1581
- Stack traces completos en produccion (solo en DEBUG)

**Filtrado automatico de PII en logs:**

Todo sistema debe implementar un filtro de PII en la capa de logging que:
- Detecte patrones de email, documento de identidad, telefono y tarjeta de credito
- Aplique **masking automatico** antes de persistir el log (ej: `email: j***@example.com`)
- Registre una metrica `pii_filtered_total` por cada dato filtrado (para monitorear fugas potenciales)
- Nunca lance excepcion si el filtro falla — loggear sin el campo sensible, no bloquear el flujo

Patrones minimos a filtrar:

| Tipo de dato | Patron de deteccion | Formato de masking |
|-------------|--------------------|--------------------|
| Email | `[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}` | `j***@***.com` |
| Cedula/Documento | Secuencia numerica de 6-12 digitos en contexto de documento | `****1234` (ultimos 4) |
| Telefono | Patron de telefono con prefijo de pais | `+57***4567` |
| Tarjeta de credito | 13-19 digitos con formato de tarjeta | `****-****-****-1234` |
| Token/API key | Strings alfanumericos largos (>20 chars) en contexto de auth | `[REDACTED]` |

---

### 2. Trazas distribuidas (seguimiento de flujos)

**Propagacion:**
- Cada request entrante debe tener o generar un `traceId`
- El `traceId` se propaga a todos los servicios downstream
- Cada operacion significativa crea un **span** dentro de la traza

**Convencion de nombres de spans:**

```
{servicio}.{operacion}
```

Ejemplos: `booking-api.create-booking`, `availability-service.check-availability`

**Atributos obligatorios por span:**

| Atributo | Descripcion |
|----------|------------|
| `service.name` | Nombre del servicio |
| `operation.name` | Nombre de la operacion |
| `http.method` | Metodo HTTP (si aplica) |
| `http.status_code` | Codigo de respuesta (si aplica) |
| `error` | `true` si el span represento un error |
| `spec.id` | ID de la especificacion relacionada (si es trazable) |

---

### 3. Metricas (indicadores cuantitativos)

**Metricas obligatorias por servicio:**

| Metrica | Tipo | Descripcion |
|---------|------|------------|
| `request_duration_ms` | Histogram | Latencia de cada request |
| `request_total` | Counter | Total de requests recibidos |
| `error_total` | Counter | Total de errores por tipo |
| `dependency_duration_ms` | Histogram | Latencia de llamadas a dependencias |
| `active_connections` | Gauge | Conexiones activas (si aplica) |

**Convencion de nombres:**

```
{servicio}_{metrica}_{unidad}
```

Ejemplo: `booking_api_request_duration_ms`

**Labels/tags obligatorios:**

| Label | Descripcion | Valores |
|-------|------------|---------|
| `method` | Metodo HTTP | `GET`, `POST`, `PUT`, `DELETE` |
| `endpoint` | Endpoint | `/bookings`, `/bookings/{id}` |
| `status` | Codigo de respuesta agrupado | `2xx`, `4xx`, `5xx` |
| `service` | Nombre del servicio | `booking-api` |

---

## Monitoreo y alertas

### Alertas obligatorias

| Alerta | Condicion | Severidad |
|--------|----------|-----------|
| Error rate alto | `error_total / request_total > {{umbral, ej: 5%}}` en 5 min | Critica |
| Latencia alta | `p95(request_duration_ms) > {{umbral, ej: 2000ms}}` en 5 min | Alta |
| Servicio caido | Sin requests en {{umbral, ej: 2 min}} | Critica |
| Dependencia degradada | `p95(dependency_duration_ms) > {{umbral}}` en 5 min | Alta |
| PII detectada en logs | `pii_filtered_total > 0` en ventana de 1 min | Alta |

### Dashboards requeridos

Cada servicio debe tener un dashboard que muestre:
- Throughput (requests/segundo)
- Latencia (p50, p95, p99)
- Error rate
- Estado de dependencias
- Metricas de negocio relevantes (ej: bookings/minuto)

---

## Requisitos en especificaciones

En las especificaciones L2, la seccion "Execution Requirements" debe incluir:

```
Observabilidad:
  - Logs: eventos de creacion, errores, validaciones fallidas
  - Trazas: span por operacion principal y llamadas a dependencias
  - Metricas: latencia, throughput, error rate
  - Alertas: error rate > {{umbral}}, latencia p95 > {{umbral}}ms
```

---

## Configuracion

{{Agregar stack de observabilidad del proyecto aqui, ej:}}
{{- Logs: ELK Stack / CloudWatch / Datadog}}
{{- Trazas: Jaeger / Zipkin / AWS X-Ray / OpenTelemetry}}
{{- Metricas: Prometheus + Grafana / Datadog / CloudWatch}}
{{- Alertas: PagerDuty / OpsGenie / Grafana Alerting}}

---

*Referencia: ADR-006 (Data Classification & DLP), Framework AI-Driven Engineering v1.0 — Tech&Solve*
