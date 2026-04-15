# /specifications — Especificaciones Ejecutables por IA

Contiene todas las especificaciones del proyecto en los 3 niveles. Estas specs son **AI-executable**: el agente las lee y genera codigo a partir de ellas.

## Estructura

| Carpeta | Nivel | Proposito |
|---------|-------|-----------|
| domain/ | L1 | Especificaciones de dominio — QUE desde el negocio |
| system/ | L2 | Especificaciones de sistema — COMO tecnicamente |
| changes/ | L3 | Especificaciones de cambio — cambios incrementales |
| context/ | — | Contexto inicial capturado (CONTEXT-INTAKE-TEMPLATE) |

## Reglas

- Toda spec debe estar **APPROVED** antes de generar codigo (Gate 1)
- Versionado semantico obligatorio (semver: MAJOR.MINOR.PATCH)
- Cambios a specs requieren PR y revision del arquitecto
- Las specs son la unica fuente de verdad — el prompt invoca la spec, no la reemplaza
- Clasificacion de datos (PII) obligatoria en specs L1

## DDD en especificaciones

- **L1** define: Bounded Context, Agregados, Entidades, Value Objects, Eventos de dominio, Lenguaje ubicuo
- **L2** define: Agregado principal por caso de uso, Eventos de dominio (contrato tecnico), Puertos (ACL)
- **L3** define: Delta sobre cualquier concepto de L2

## Relacion entre niveles

```
L1 (Dominio) ──1:N──> L2 (Sistema) ──1:N──> L3 (Cambio)
```

Trazabilidad: L1 → L2 → L3 → codigo → commit → release

> Los templates de specs estan en fase-1/specs/templates/
