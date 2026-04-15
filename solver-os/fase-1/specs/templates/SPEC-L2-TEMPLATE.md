# Especificacion de Sistema (Nivel 2)

| Campo | Valor |
|-------|-------|
| **Spec ID** | SPEC-L2-{{system-name}} |
| **Nivel** | L2 — Sistema |
| **Version** | {{X.Y.Z}} |
| **Parent L1** | {{SPEC-L1-xxx}} v{{X.Y.Z}} |
| **Estado** | `DRAFT` | `IN_REVIEW` | `APPROVED` | `IMPLEMENTED` | `DEPRECATED` |
| **Creado por** | @{{autor}} |
| **Fecha de creacion** | {{YYYY-MM-DD}} |
| **Aprobado por** | @{{aprobador}} |
| **Fecha de aprobacion** | {{YYYY-MM-DD}} |
| **PR** | {{link al PR}} |

---

## Descripcion general

_Resumen breve de que hace este sistema o modulo y cual es su proposito dentro de la arquitectura._

---

## Sistema

**Nombre:** {{nombre del sistema o modulo}}

## Caso de uso

**Nombre:** {{nombre del caso de uso, ej: CreateOrder, ProcessPayment}}

**Descripcion:** {{que hace este caso de uso}}

**Agregado principal:** {{agregado raiz que este caso de uso modifica (definido en L1)}}

---

## Contexto

_Estado actual del sistema, decisiones tecnicas previas, dependencias relevantes y supuestos que enmarcan esta especificacion._

---

## Contrato tecnico

### Input

| Campo | Tipo | Requerido | Descripcion |
|-------|------|-----------|------------|
| {{campo1}} | {{string/number/boolean/object}} | Si/No | {{descripcion}} |
| {{campo2}} | {{tipo}} | Si/No | {{descripcion}} |
| {{campo3}} | {{tipo}} | Si/No | {{descripcion}} |

> Referencia OpenAPI: `{{path al archivo openapi.yaml}}` (generar con SK-003 usando OPENAPI-CONTRACT-TEMPLATE.yaml)
> El contrato OpenAPI es el punto de convergencia entre backend y frontend. Debe existir en ambos repositorios.

### Output

| Campo | Tipo | Descripcion |
|-------|------|------------|
| {{campo1}} | {{tipo}} | {{descripcion}} |
| {{campo2}} | {{tipo}} | {{descripcion}} |

---

## Comportamiento esperado

_Describe paso a paso como debe comportarse el sistema al ejecutar este caso de uso._

1. {{El sistema recibe la solicitud y valida autenticacion...}}
2. {{Valida los datos de entrada contra las reglas de dominio...}}
3. {{Invoca la dependencia X para...}}
4. {{Persiste el resultado...}}
5. {{Publica evento / retorna respuesta...}}

---

## Eventos de dominio

_Especificacion tecnica de los eventos publicados por este caso de uso. Cada evento debe estar definido previamente en la spec L1._

### {{NombreDelEvento}}

| Campo | Valor |
|-------|-------|
| **Nombre** | {{NombreDelEvento}} |
| **Evento L1** | {{referencia al evento en spec L1}} |
| **Canal** | {{EventBus / topic / queue}} |
| **Tipo de entrega** | {{At-least-once / At-most-once}} |

**Payload:**

| Campo | Tipo | Descripcion |
|-------|------|------------|
| {{campo1}} | {{tipo}} | {{descripcion}} |
| {{campo2}} | {{tipo}} | {{descripcion}} |

> Los eventos se publican DESPUES de la persistencia exitosa.

---

## Tipos de error

| Codigo de error | HTTP Status | Descripcion | Condicion de disparo |
|----------------|-------------|------------|---------------------|
| {{ERROR_CODE_1}} | {{4xx/5xx}} | {{descripcion}} | {{cuando ocurre}} |
| {{ERROR_CODE_2}} | {{4xx/5xx}} | {{descripcion}} | {{cuando ocurre}} |
| {{ERROR_CODE_3}} | {{4xx/5xx}} | {{descripcion}} | {{cuando ocurre}} |

---

## Dependencias permitidas

> **REGLA (ADR-004):** Solo estas dependencias pueden ser usadas en la implementacion. La IA no debe inventar dependencias adicionales.

| Dependencia | Proposito | Tipo de interfaz |
|------------|----------|-----------------|
| {{DependencyService1}} | {{para que se usa}} | {{Puerto/API REST/gRPC/Event}} |
| {{DependencyService2}} | {{para que se usa}} | {{tipo}} |
| {{DependencyService3}} | {{para que se usa}} | {{tipo}} |

---

## Requisitos de ejecucion

| Requisito | Valor | Notas |
|-----------|-------|-------|
| **Latencia maxima** | {{max_ms}} ms (p95) | {{notas}} |
| **Idempotencia** | [Requerida | No requerida] | Key: {{campo de idempotency key}} |
| **Atomicidad** | [Transaccion atomica | Eventual consistency] | {{notas}} |
| **Observabilidad** | Ver abajo | |

### Observabilidad requerida

- **Logs:** {{eventos a loggear: creacion, errores, validaciones fallidas}}
- **Trazas:** {{spans requeridos: operacion principal, llamadas a dependencias}}
- **Metricas:** {{metricas especificas: latencia, throughput, error rate}}
- **Alertas:** {{condiciones de alerta: error rate > X%, latencia p95 > Yms}}

---

## Restricciones de arquitectura

ADRs aplicables a este sistema:

| ADR | Titulo | Impacto en esta spec |
|-----|--------|---------------------|
| ADR-001 | Ports & Adapters | {{como aplica}} |
| ADR-002 | Idempotency | {{como aplica}} |
| {{ADR-NNN}} | {{titulo}} | {{como aplica}} |

---

## Restricciones y consideraciones

| Tipo | Descripcion |
|------|------------|
| **Limitaciones tecnicas** | {{restricciones de infraestructura, plataforma o stack}} |
| **Integraciones necesarias** | {{sistemas externos con los que se debe integrar}} |
| **Consideraciones de seguridad** | {{autenticacion, autorizacion, validacion de inputs, datos sensibles}} |
| **Consideraciones de rendimiento** | {{volumetria esperada, concurrencia, escalabilidad}} |

---

## Criterios de aceptacion

- [ ] {{Criterio 1: descripcion verificable ligada al contrato}}
- [ ] {{Criterio 2: descripcion verificable ligada a reglas de negocio}}
- [ ] {{Criterio 3: descripcion verificable ligada a requisitos no funcionales}}

---

## Escenarios de prueba (GWT)

> **REGLA (ADR-005):** Estos escenarios deben traducirse en tests ANTES de generar la implementacion.

| ID | Given (Dado que) | When (Cuando) | Then (Entonces) | Prioridad |
|----|-----------------|--------------|-----------------|-----------|
| TS-001 | {{precondicion}} | {{accion}} | {{resultado esperado}} | [Alta | Media | Baja] |
| TS-002 | {{precondicion}} | {{accion}} | {{resultado esperado}} | [Alta | Media | Baja] |
| TS-003 | {{precondicion}} | {{accion}} | {{resultado esperado}} | [Alta | Media | Baja] |
| TS-004 | {{precondicion — caso de error}} | {{accion}} | {{error esperado}} | [Alta | Media | Baja] |

---

## Diseno / Mockups

_Incluye enlaces a disenos, wireframes, diagramas de secuencia o arquitectura si aplica._

---

## Referencias

- {{Enlace a documentacion tecnica relevante}}
- {{Enlace a ADRs referenciados}}
- {{Enlace a OpenAPI / contratos existentes}}

---

## Especificaciones relacionadas

| Spec ID | Nivel | Version | Relacion |
|---------|-------|---------|----------|
| {{SPEC-L1-xxx}} | L1 | {{X.Y.Z}} | Parent — dominio |
| {{SPEC-L3-xxx}} | L3 | {{X.Y.Z}} | Child — cambio incremental |

---

## Historial de cambios

| Version | Fecha | Autor | Descripcion del cambio |
|---------|-------|-------|----------------------|
| {{X.Y.Z}} | {{YYYY-MM-DD}} | {{autor}} | {{descripcion}} |
