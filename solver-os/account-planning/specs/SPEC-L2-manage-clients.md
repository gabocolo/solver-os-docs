# Especificacion de Sistema (Nivel 2)

| Campo | Valor |
|-------|-------|
| **Spec ID** | SPEC-L2-manage-clients |
| **Nivel** | L2 — Sistema |
| **Version** | 1.0.0 |
| **Parent L1** | SPEC-L1-account-planning v2.0.0 |
| **Estado** | DRAFT |
| **Creado por** | @juan.gabriel.colorado |
| **Fecha de creacion** | 2026-04-09 |
| **Aprobado por** | Pendiente |
| **Fecha de aprobacion** | Pendiente |
| **PR** | Pendiente |

---

## Descripcion general

Modulo de gestion de clientes (prospectos) del sistema de Account Planning. Provee operaciones CRUD basicas sobre clientes y consulta del historial completo de investigaciones asociadas a cada cliente. Es el punto de entrada del flujo: el comercial registra un cliente antes de investigarlo.

---

## Sistema

**Nombre:** Client Management Module

## Caso de uso

**Nombre:** ManageClients

**Descripcion:** Permite crear, editar, consultar y listar clientes con su informacion basica (nombre, sector, pais, idioma, LinkedIn, descripcion). Cada cliente mantiene un historial de investigaciones realizadas. La actualizacion de informacion es bajo demanda — el comercial decide cuando refrescar los datos.

---

## Contexto

- Los clientes son prospectos o cuentas que el equipo comercial de Tech&Solve quiere investigar para preparar reuniones
- Hoy esta informacion esta en SharePoint en carpetas por cliente, sin estructura estandarizada
- Este modulo centraliza la informacion y la asocia a investigaciones posteriores
- No hay integracion con CRM externo en MVP — todo se gestiona dentro del sistema

---

## Contrato tecnico

### Input — CreateClient

| Campo | Tipo | Requerido | Descripcion |
|-------|------|-----------|------------|
| name | string | Si | Nombre de la empresa/cliente |
| industry | string | Si | Sector o industria (ej: Seguros, Banca, Retail) |
| country | string | Si | Pais del cliente |
| language | string | No | Idioma principal del cliente (default: "ES") |
| linkedinUrl | string | No | URL del perfil de LinkedIn de la empresa |
| description | string | No | Descripcion general o notas del comercial sobre el cliente |

### Input — UpdateClient

| Campo | Tipo | Requerido | Descripcion |
|-------|------|-----------|------------|
| clientId | string (UUID) | Si | ID del cliente a actualizar |
| name | string | No | Nombre actualizado |
| industry | string | No | Sector actualizado |
| country | string | No | Pais actualizado |
| language | string | No | Idioma actualizado |
| linkedinUrl | string | No | LinkedIn actualizado |
| description | string | No | Descripcion actualizada |

### Input — GetClient

| Campo | Tipo | Requerido | Descripcion |
|-------|------|-----------|------------|
| clientId | string (UUID) | Si | ID del cliente a consultar |

### Input — ListClients

| Campo | Tipo | Requerido | Descripcion |
|-------|------|-----------|------------|
| search | string | No | Texto libre para buscar por nombre o industria |
| page | integer | No | Pagina (default: 1) |
| pageSize | integer | No | Tamano de pagina (default: 20, max: 100) |

### Output — ClientResponse

| Campo | Tipo | Descripcion |
|-------|------|------------|
| clientId | string (UUID) | ID unico del cliente |
| name | string | Nombre de la empresa |
| industry | string | Sector/industria |
| country | string | Pais |
| language | string | Idioma principal |
| linkedinUrl | string | URL del LinkedIn |
| description | string | Descripcion/notas |
| investigationHistory | InvestigationSummary[] | Historial de investigaciones |
| investigationHistory[].investigationId | string (UUID) | ID de la investigacion |
| investigationHistory[].type | string | QUICK o DEEP |
| investigationHistory[].status | string | Estado de la investigacion |
| investigationHistory[].startedAt | string (ISO 8601) | Cuando inicio |
| investigationHistory[].completedAt | string (ISO 8601) | Cuando termino (null si en progreso) |
| createdAt | string (ISO 8601) | Fecha de creacion |
| updatedAt | string (ISO 8601) | Ultima actualizacion |

---

## Comportamiento esperado

### CreateClient
1. El sistema valida que no exista otro cliente con el mismo nombre y pais
2. Asigna un UUID como clientId
3. Persiste el cliente con timestamps
4. Retorna el cliente creado (sin historial de investigaciones)

### UpdateClient
1. El sistema valida que el cliente existe
2. Actualiza solo los campos enviados (patch semantico)
3. Actualiza el timestamp updatedAt
4. Retorna el cliente actualizado con su historial

### GetClient
1. El sistema busca el cliente por clientId
2. Incluye el historial de investigaciones ordenado por fecha (mas reciente primero)
3. Retorna el cliente completo

### ListClients
1. Si hay filtro search, busca por nombre o industria (case-insensitive, contains)
2. Retorna lista paginada ordenada por ultima actualizacion (mas reciente primero)
3. No incluye historial de investigaciones en el listado (solo en GetClient)

---

## Tipos de error

| Codigo de error | HTTP Status | Descripcion | Condicion de disparo |
|----------------|-------------|------------|---------------------|
| CLIENT_NOT_FOUND | 404 | El cliente no existe | clientId no encontrado en GetClient o UpdateClient |
| DUPLICATE_CLIENT | 409 | Ya existe un cliente con el mismo nombre y pais | name + country duplicado en CreateClient |
| INVALID_DATA | 400 | Datos de entrada invalidos | name vacio, industry vacia, country vacio |
| INVALID_LANGUAGE | 400 | Idioma no soportado | language no es ES, EN o PT |

---

## Dependencias permitidas

> **REGLA (ADR-004):** Solo estas dependencias pueden ser usadas en la implementacion.

| Dependencia | Proposito | Tipo de interfaz |
|------------|----------|-----------------|
| ClientRepository | Persistir y consultar clientes | Puerto (IClientRepository) |
| InvestigationRepository | Consultar historial de investigaciones por cliente | Puerto (IInvestigationRepository) |
| EventBus | Publicar eventos ClientCreated, ClientUpdated | Puerto (IEventPublisher) |

---

## Requisitos de ejecucion

| Requisito | Valor | Notas |
|-----------|-------|-------|
| **Latencia maxima** | 500 ms (p95) | Operaciones CRUD simples |
| **Idempotencia** | No requerida | CRUD estandar |
| **Atomicidad** | Transaccion atomica por operacion | |
| **Observabilidad** | Ver abajo | |

### Observabilidad requerida

- **Logs:** Cliente creado (INFO), cliente actualizado (INFO), cliente no encontrado (WARN), duplicado (WARN)
- **Trazas:** span por operacion CRUD
- **Metricas:** client_count_total, clients_created_total, client_search_duration_ms
- **Alertas:** error rate > 5% en 5 min

---

## Restricciones de arquitectura

| ADR | Titulo | Impacto en esta spec |
|-----|--------|---------------------|
| ADR-001 | Ports & Adapters | ManageClients depende de puertos (IClientRepository, IEventPublisher), no de implementaciones |
| ADR-004 | Dependency Management | Solo las 3 dependencias listadas |
| ADR-005 | Testing Strategy | Tests GWT antes de implementar |

---

## Restricciones y consideraciones

| Tipo | Descripcion |
|------|------------|
| **Limitaciones tecnicas** | MVP sin paginacion compleja — offset/limit basico |
| **Integraciones necesarias** | Ninguna externa en MVP |
| **Consideraciones de seguridad** | OAuth GitHub requerido. No almacenar datos sensibles del prospecto (PII de empleados) |
| **Consideraciones de rendimiento** | < 50 clientes en MVP. Sin necesidad de cache |

---

## Criterios de aceptacion

- [ ] Se puede crear un cliente con nombre, sector, pais e idioma
- [ ] Se puede actualizar un cliente existente (patch parcial)
- [ ] Se puede consultar un cliente con su historial de investigaciones
- [ ] Se puede buscar clientes por nombre o industria
- [ ] No se permite crear dos clientes con el mismo nombre y pais
- [ ] El listado esta paginado y ordenado por ultima actualizacion

---

## Escenarios de prueba (GWT)

> **REGLA (ADR-005):** Estos escenarios deben traducirse en tests ANTES de generar la implementacion.

| ID | Given (Dado que) | When (Cuando) | Then (Entonces) | Prioridad |
|----|-----------------|--------------|-----------------|-----------|
| TS-001 | No existe un cliente "Bancolombia" en Colombia | Se crea el cliente con nombre, sector y pais | Cliente creado con UUID, timestamps y sin historial | Alta |
| TS-002 | Existe un cliente "Bancolombia" con sector "Banca" | Se actualiza el sector a "Banca y Seguros" | Cliente actualizado, updatedAt cambia, demas campos intactos | Alta |
| TS-003 | Existe un cliente con 3 investigaciones completadas | Se consulta el cliente por ID | Retorna el cliente con historial de 3 investigaciones ordenadas por fecha desc | Alta |
| TS-004 | Existen 5 clientes, 2 del sector "Seguros" | Se busca con search="seguros" | Retorna los 2 clientes del sector seguros | Media |
| TS-005 | Ya existe "Bancolombia" en Colombia | Se intenta crear "Bancolombia" en Colombia | Error DUPLICATE_CLIENT | Alta |
| TS-006 | No existe un cliente con el ID dado | Se consulta por un ID inexistente | Error CLIENT_NOT_FOUND | Alta |

---

## Referencias

- SPEC-L1-account-planning v2.0.0 — Dominio
- context/scope.md — Seccion "Gestion de clientes"

---

## Especificaciones relacionadas

| Spec ID | Nivel | Version | Relacion |
|---------|-------|---------|----------|
| SPEC-L1-account-planning | L1 | 2.0.0 | Parent — dominio |

---

## Historial de cambios

| Version | Fecha | Autor | Descripcion del cambio |
|---------|-------|-------|----------------------|
| 1.0.0 | 2026-04-09 | Juan Gabriel Colorado | Version inicial: CRUD de clientes con historial de investigaciones |
