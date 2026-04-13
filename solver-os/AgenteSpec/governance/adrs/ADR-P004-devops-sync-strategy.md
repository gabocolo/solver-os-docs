# ADR-P004: Estrategia de sincronizacion con Azure DevOps

| Campo | Valor |
|-------|-------|
| **Estado** | ACCEPTED |
| **Fecha** | 2026-04-09 |
| **Derivado de** | ADR-003 (Trazabilidad) |
| **Decisor** | Equipo de Arquitectura |

---

## Contexto

El framework requiere trazabilidad completa: spec → commit → release. Para cerrar la brecha entre specs aprobadas y trabajo en el backlog del equipo, necesitamos sincronizar specs aprobadas con Azure DevOps como work items. La sincronizacion debe ser automatica, tolerante a fallos y no debe bloquear el flujo de aprobacion.

---

## Decision

Sincronizacion asincrona event-driven: cuando una spec se aprueba, un evento `SpecApproved` dispara la creacion automatica de un work item en Azure DevOps.

### Mapeo de tipos

| Nivel Spec | Tipo Work Item | Relacion |
|-----------|---------------|----------|
| L1 (Dominio) | Feature | Raiz (sin parent) |
| L2 (Sistema) | User Story | Parent → Feature de la L1 |
| L3 (Cambio) | Task | Parent → User Story de la L2 |

### Principios

1. **No bloquea aprobacion**: si Azure DevOps falla, la spec queda APPROVED. El sync se reintenta.
2. **Idempotente**: si el work item ya existe para esa spec, no crear duplicado.
3. **Asincrono**: via evento en Service Bus. El reviewer no espera la sincronizacion.
4. **Tolerante a fallos**: 3 reintentos automaticos con backoff (5s, 15s, 45s). Si falla, queda FAILED para reintento manual.
5. **Trazable**: todo sync queda registrado en `work_item_syncs` con estado, URL y errores.

### API usada

Azure DevOps REST API v7.1:
- `POST /_apis/wit/workitems/${type}` — crear work item
- `PATCH /_apis/wit/workitems/{id}` — agregar relacion parent-child

---

## Alternativas evaluadas

### Alternativa 1: Sincronizacion sincrona en el flujo de aprobacion
- **Pro**: Simple, inmediato
- **Contra**: Si DevOps falla, bloquea la aprobacion. Acoplamiento fuerte
- **Descartada**: La aprobacion de spec no debe depender de un sistema externo

### Alternativa 2: Sincronizacion manual (boton "Crear Work Item")
- **Pro**: Control total del usuario
- **Contra**: Se olvida, no es automatico, rompe la trazabilidad
- **Descartada**: La trazabilidad debe ser automatica

### Alternativa 3: Webhook desde Azure DevOps hacia el portal
- **Pro**: Bidireccional
- **Contra**: Requiere endpoint publico, complejidad de setup, no resuelve el sentido portal → DevOps
- **Evaluada para futuro**: Util para sincronizar estado de work items de vuelta al portal

---

## Consecuencias

### Positivas
- Trazabilidad automatica: spec aprobada → work item creado
- Equipo ve work items en sus Boards sin accion manual
- Acceptance criteria extraidos de GWT de la spec
- Relaciones parent-child mantienen la jerarquia L1 → L2 → L3

### Negativas
- Dependencia de Azure DevOps API (externa)
- PAT requiere gestion segura (cifrado, rotacion)
- Si el parent work item no existe (L1 no sincronizada), la relacion falla silenciosamente
- Cambios en la spec post-aprobacion no se reflejan en el work item (one-way sync)
