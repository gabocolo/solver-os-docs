# ADR-003: Trazabilidad obligatoria spec → commit → release

| Campo | Valor |
|-------|-------|
| **ADR ID** | ADR-003 |
| **Estado** | ACCEPTED |
| **Fecha** | 2026-04-07 |
| **Autor** | Equipo de Arquitectura |
| **PR** | Pendiente |
| **Reemplaza** | N/A |
| **Reemplazado por** | N/A |

## Contexto

En un entorno donde la IA genera codigo rapidamente, es critico poder rastrear el origen de cada cambio: que especificacion lo motivo, que prompt lo genero, y en que release se desplego. Sin esta trazabilidad, es imposible auditar decisiones, identificar el origen de defectos o medir la efectividad del framework.

## Decision

Todo cambio de codigo debe ser trazable desde una especificacion aprobada hasta el release. La cadena de trazabilidad es:

```
Spec (L1/L2/L3) → Commit (referencia spec ID + version) → PR → Release
```

Cada commit debe referenciar el spec ID y version que lo origino. Cada PR debe incluir el link a la especificacion.

## Alternativas evaluadas

| Opcion | Descripcion | Pros | Contras |
|--------|------------|------|---------|
| **Trazabilidad spec → commit → release** (elegida) | Cadena completa desde la especificacion hasta produccion | Auditabilidad total, facilita aprendizaje del sistema, permite medir efectividad | Requiere disciplina en commits y PRs, overhead en el flujo |
| **Trazabilidad solo por ticket (Jira/Linear)** | Referenciar solo el ticket de trabajo, no la spec | Familiar para equipos, menor cambio de proceso | Pierde la trazabilidad hacia la especificacion tecnica, no mide efectividad del framework |
| **Sin trazabilidad formal** | Cada developer documenta a su criterio | Sin overhead adicional | Imposible auditar, imposible medir, imposible reproducir |

## Consecuencias

### Positivas
- Cadena completa de auditoria: de negocio hasta produccion
- Facilita el aprendizaje del sistema (se puede recorrer la historia de decisiones)
- Permite medir el "efecto de trazabilidad" (metrica del framework)
- Facilita el debugging: ante un defecto, se puede rastrear la spec original

### Negativas
- Requiere que cada commit incluya la referencia al spec
- Los PRs deben incluir link a la especificacion
- Overhead adicional en el flujo de trabajo (mitigado con templates)

## Cumplimiento

- En Gate 1: verificar que existe spec aprobada antes de generar codigo
- En commits: el mensaje debe incluir `spec: {SPEC-ID} v{X.Y.Z}` 
- En PRs: el template debe incluir link a la especificacion
- En Gate 3: el reviewer verifica la trazabilidad antes de aprobar
- Metrica: medir % de cambios trazables desde spec aprobada

## ADRs relacionados

- ADR-002: Idempotency — el idempotency key contribuye a la trazabilidad operacional
- ADR-005: Testing Strategy — los tests tambien deben ser trazables a la spec
