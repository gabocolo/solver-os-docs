# 02 — Diagramas de Secuencia

| Campo | Valor |
|-------|-------|
| **Proyecto** | Spec Agent Portal |
| **Version** | 1.0.0 |
| **Fecha** | 2026-04-09 |

---

## Flujo 1: Crear proyecto y configurar gobernanza

```
Architect          Frontend           API Gateway        Governance Svc      DB (PostgreSQL)     AuditLog
    |                  |                  |                   |                   |                 |
    |-- Crear proy --->|                  |                   |                   |                 |
    |                  |-- POST /projects -->                 |                   |                 |
    |                  |                  |-- Validar JWT ---->|                   |                 |
    |                  |                  |-- Validar ARCHITECT                    |                 |
    |                  |                  |                   |-- Check duplicate  |                 |
    |                  |                  |                   |<-- OK -------------|                 |
    |                  |                  |                   |-- INSERT project ->|                 |
    |                  |                  |                   |<-- projectId ------|                 |
    |                  |                  |                   |-- INSERT audit ----|---------------->|
    |                  |<-- 201 Created --|<-- ProjectResp ---|                   |                 |
    |<-- Proyecto creado|                 |                   |                   |                 |
    |                  |                  |                   |                   |                 |
    |-- Registrar ADR->|                  |                   |                   |                 |
    |                  |-- POST /adrs --->|                   |                   |                 |
    |                  |                  |-- Forward ------->|                   |                 |
    |                  |                  |                   |-- INSERT adr ---->|                 |
    |                  |                  |                   |-- INSERT audit ---|---------------->|
    |                  |<-- 201 Created --|<-- ADRResponse ---|                   |                 |
    |                  |                  |                   |                   |                 |
    |-- Config gates-->|                  |                   |                   |                 |
    |                  |-- PUT /gates --->|                   |                   |                 |
    |                  |                  |-- Forward ------->|                   |                 |
    |                  |                  |                   |-- UPSERT gates -->|                 |
    |                  |<-- 200 OK ------|<-- GatesConfig ----|                   |                 |
```

---

## Flujo 2: Generar spec L1 con IA (asincrono)

```
Dev Senior         Frontend           API Gateway        Agent Orch Svc      Service Bus       LLM (Claude)      DLP Filter       DB
    |                  |                  |                   |                   |                 |                |              |
    |-- Generar L1 --->|                  |                   |                   |                 |                |              |
    |  (descripcion    |-- POST /specs/   |                   |                   |                 |                |              |
    |   lenguaje nat.) |   generate/l1 -->|                   |                   |                 |                |              |
    |                  |                  |-- Forward ------->|                   |                 |                |              |
    |                  |                  |                   |-- Cargar ADRs ----|----------------|----------------|------------->|
    |                  |                  |                   |<-- ADRs ----------|----------------|----------------|--------------|
    |                  |                  |                   |-- Armar contexto  |                 |                |              |
    |                  |                  |                   |-- Enqueue job --->|                 |                |              |
    |                  |<-- 202 Accepted -|<-- {jobId} -------|                   |                 |                |              |
    |<-- "Generando..."|                  |                   |                   |                 |                |              |
    |                  |                  |                   |                   |                 |                |              |
    |                  |          [ASYNC - Worker procesa job]|                   |                 |                |              |
    |                  |                  |                   |<-- Dequeue job ---|                 |                |              |
    |                  |                  |                   |                   |                 |                |              |
    |                  |                  |                   |-- Pre-prompt -----|-----------------|--------------->|              |
    |                  |                  |                   |<-- Input sanitiz -|-----------------|----------------|              |
    |                  |                  |                   |                   |                 |                |              |
    |                  |                  |                   |-- Generate -------|---------------->|                |              |
    |                  |                  |                   |  (Opus, template  |                 |                |              |
    |                  |                  |                   |   L1 + ADRs)      |                 |                |              |
    |                  |                  |                   |<-- Borrador L1 ---|-----------------|                |              |
    |                  |                  |                   |                   |                 |                |              |
    |                  |                  |                   |-- Post-response --|-----------------|--------------->|              |
    |                  |                  |                   |<-- Validado ------|-----------------|----------------|              |
    |                  |                  |                   |                   |                 |                |              |
    |                  |                  |                   |-- INSERT spec DRAFT ---------------|----------------|------------->|
    |                  |                  |                   |-- INSERT audit ---|----------------|----------------|------------->|
    |                  |                  |                   |                   |                 |                |              |
    |                  |<-- WebSocket: spec ready ------------|                   |                 |                |              |
    |<-- Ver borrador  |                  |                   |                   |                 |                |              |
```

---

## Flujo 3: Derivar spec L2 desde L1

```
Dev Senior         Frontend           Agent Orch Svc      DB                  Service Bus       LLM (Sonnet)     DLP
    |                  |                   |                 |                   |                 |                |
    |-- Derivar L2 --->|                   |                 |                   |                 |                |
    |  (L1 parent +    |-- POST /specs/    |                 |                   |                 |                |
    |   caso de uso)   |   generate/l2 --->|                 |                   |                 |                |
    |                  |                   |-- Validar L1 APPROVED ------------->|                 |                |
    |                  |                   |<-- L1 completa --|                   |                 |                |
    |                  |                   |-- Extraer CU + entidades + reglas   |                 |                |
    |                  |                   |-- Cargar template L2                |                 |                |
    |                  |                   |-- DLP pre-prompt |                   |                 |--------------->|
    |                  |                   |<-- Sanitizado ---|-------------------|-----------------|----------------|
    |                  |                   |-- Enqueue job ---|------------------>|                 |                |
    |                  |<-- 202 Accepted --|                  |                   |                 |                |
    |                  |                   |                   |                   |                 |                |
    |                  |             [ASYNC]|                  |                   |                 |                |
    |                  |                   |-- Generate (Sonnet) --------------->|                 |
    |                  |                   |<-- Borrador L2 --|-------------------|-----------------|                |
    |                  |                   |-- DLP post-resp  |                   |                 |--------------->|
    |                  |                   |-- INSERT spec ---|----------------->|                  |                |
    |                  |                   |   parentSpecId = L1.id              |                  |                |
    |                  |                   |   version = 1.0.0                   |                  |                |
    |                  |<-- WebSocket -----|                  |                   |                 |                |
    |<-- Ver borrador  |                   |                  |                   |                 |                |
```

---

## Flujo 4: Crear spec L3 (cambio incremental)

```
Dev Senior         Frontend           Agent Orch Svc      DB                  LLM (Sonnet)
    |                  |                   |                 |                   |
    |-- Crear L3 ----->|                   |                 |                   |
    |  (L2 parent +    |-- POST /specs/    |                 |                   |
    |   cambio)        |   generate/l3 --->|                 |                   |
    |                  |                   |-- Validar L2 APPROVED ------------->|
    |                  |                   |<-- L2 completa --|                   |
    |                  |                   |-- Armar contexto: L2 + cambio       |
    |                  |                   |-- DLP + Enqueue  |                   |
    |                  |<-- 202 Accepted --|                  |                   |
    |                  |                   |                   |                  |
    |                  |             [ASYNC]|                  |                  |
    |                  |                   |-- Generate -------|---------------->|
    |                  |                   |<-- Borrador L3 ---|-----------------|
    |                  |                   |-- INSERT spec --->|                 |
    |                  |                   |   version bump semver               |
    |                  |<-- WebSocket -----|                   |                 |
    |                  |                   |                   |                 |
    |                  |  [Si estimacion > 400 lineas]        |                 |
    |                  |<-- Warning: "Considere dividir" -----|                 |
```

---

## Flujo 5: Revision y aprobacion (Gate 1)

```
Autor              Revisor            Frontend           Review Svc          Validator (SK-004)   DB              DevOps Sync
    |                  |                  |                   |                   |                 |                |
    |-- Enviar a rev ->|                  |                   |                   |                 |                |
    |                  |                  |-- POST /reviews -->|                  |                 |                |
    |                  |                  |                   |-- Ejecutar SK-004 |                 |                |
    |                  |                  |                   |                   |-- Validar:      |                |
    |                  |                  |                   |                   |   estructura    |                |
    |                  |                  |                   |                   |   semver        |                |
    |                  |                  |                   |                   |   clasificacion |                |
    |                  |                  |                   |                   |   ADRs          |                |
    |                  |                  |                   |                   |   parent        |                |
    |                  |                  |                   |<-- ValidationResult|                |                |
    |                  |                  |                   |                   |                 |                |
    |                  |                  |                   |  [Si passed=true] |                 |                |
    |                  |                  |                   |-- Estado → IN_REVIEW -------------->|                |
    |                  |                  |                   |-- Crear ReviewCycle --------------->|                |
    |                  |                  |                   |-- Notificar revisor                |                |
    |                  |<-- Notificacion --|<-- Email/WS -----|                   |                 |                |
    |                  |                  |                   |                   |                 |                |
    |                  |-- Revisar spec ->|                   |                   |                 |                |
    |                  |-- Agregar coment |-- POST /comments ->|                  |                 |                |
    |                  |                  |                   |-- INSERT comment ->|                |                |
    |                  |                  |                   |                   |                 |                |
    |                  |-- Aprobar ------>|-- POST /decision ->|                  |                 |                |
    |                  |                  |                   |-- Estado → APPROVED -------------->|                |
    |                  |                  |                   |-- Completar cycle |                 |                |
    |                  |                  |                   |-- Publicar SpecApproved            |                |
    |                  |                  |                   |                   |                 |                |
    |                  |                  |                   |                   |                 |         [ASYNC]|
    |                  |                  |                   |                   |                 |-- Sync ------->|
    |                  |                  |                   |                   |                 |   spec → WI    |
    |                  |                  |                   |                   |                 |                |
    |<-- Spec aprobada |                  |                   |                   |                 |                |
```

---

## Flujo 6: Generar prompt estructurado

```
Dev Senior         Frontend           Prompt Builder       DB                  
    |                  |                   |                 |                   
    |-- Generar prompt |                   |                 |                   
    |  (spec + skill)  |-- POST /prompts ->|                 |                   
    |                  |                   |-- Cargar spec APPROVED ----------->|
    |                  |                   |<-- Spec completa |                  
    |                  |                   |-- Cargar skill ACTIVE ------------>|
    |                  |                   |<-- Skill + criteria                |
    |                  |                   |-- Cargar ADRs relevantes --------->|
    |                  |                   |<-- ADRs ---------|                  
    |                  |                   |                  |                  
    |                  |                   |-- Construir prompt (local):        
    |                  |                   |   1. Rol                           
    |                  |                   |   2. Invocacion spec + version     
    |                  |                   |   3. Tarea segun skill             
    |                  |                   |   4. Input/Output desde spec       
    |                  |                   |   5. ADRs relevantes               
    |                  |                   |   6. Reglas DLP automaticas        
    |                  |                   |   7. Custom instructions           
    |                  |                   |                  |                  
    |                  |                   |-- INSERT prompt |                  
    |                  |                   |-- INSERT audit  |                  
    |                  |<-- PromptResponse -|                 |                  
    |<-- Preview prompt|                   |                 |                  
    |                  |                   |                 |                  
    |-- Copiar ------->|                   |                 |                  
    |                  |-- clipboard API   |                 |                  
    |                  |-- metric: copied  |                 |                  
```

---

## Flujo 7: Consultar metricas y trazabilidad

```
Architect          Frontend           Metrics Svc         Redis (cache)       DB
    |                  |                   |                 |                   |
    |-- Dashboard ---->|                   |                 |                   |
    |                  |-- GET /metrics -->|                 |                   |
    |                  |                   |-- Check cache -->|                  |
    |                  |                   |                  |                  |
    |                  |                   |  [Cache HIT]     |                  |
    |                  |                   |<-- Metricas ------|                 |
    |                  |<-- 200 OK (< 200ms)                  |                 |
    |                  |                   |                   |                 |
    |                  |                   |  [Cache MISS]     |                 |
    |                  |                   |-- Query specs ----|---------------->|
    |                  |                   |-- Query reviews --|---------------->|
    |                  |                   |-- Query prompts --|---------------->|
    |                  |                   |<-- Raw data ------|-----------------|
    |                  |                   |-- Calcular agregados               |
    |                  |                   |-- SET cache (TTL 5min) ----------->|
    |                  |<-- 200 OK (< 3s) -|                  |                 |
    |                  |                   |                   |                 |
    |-- Trazabilidad ->|                   |                   |                 |
    |                  |-- GET /trace ----->|                  |                 |
    |                  |                   |-- Query tree -----|---------------->|
    |                  |                   |<-- L1→L2→L3 tree -|                |
    |                  |<-- TreeResponse ---|                  |                 |
    |<-- Arbol visual  |                   |                  |                 |
    |                  |                   |                   |                 |
    |-- Exportar ----->|                   |                   |                 |
    |                  |-- GET /export ---->|                  |                 |
    |                  |                   |-- Generate CSV/JSON                |
    |                  |<-- File download --|                  |                 |
```
