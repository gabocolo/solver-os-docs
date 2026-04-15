# Quality Gates — Spec Agent Portal

| Campo | Valor |
|-------|-------|
| **Proyecto** | Spec Agent Portal |
| **Version** | 1.0.0 |
| **Fecha** | 2026-04-09 |

---

## Gate 1: Spec aprobada antes de generar codigo

**Cuando**: antes de generar prompts y codigo.
**Bloqueante**: Si — sin spec APPROVED no se genera prompt.

### Validaciones automaticas (SK-004)

| # | Validacion | Tipo | Aplica a | Regla |
|---|-----------|------|----------|-------|
| V-01 | Campos obligatorios | CRITICAL | L1/L2/L3 | Todos los campos requeridos segun nivel deben estar presentes |
| V-02 | Semver valido | CRITICAL | L1/L2/L3 | Version cumple X.Y.Z y es mayor que version anterior |
| V-03 | Clasificacion de datos | CRITICAL | L1 | `data_classification` presente con al menos 1 entidad clasificada |
| V-04 | Proteccion de datos | CRITICAL | L2 | Si la L1 parent tiene campos Sensible, la L2 debe tener seccion proteccion |
| V-05 | ADRs referenciados | WARNING | L1/L2 | Referenciar ADR-001 si usa hexagonal, ADR-006 si tiene PII |
| V-06 | Spec no duplicada | CRITICAL | L1/L2/L3 | No existe otra spec activa con mismo titulo y nivel |
| V-07 | Parent aprobado | CRITICAL | L2/L3 | El parent esta en estado APPROVED |
| V-08 | Escenarios de test | WARNING | L2 | Al menos 3 escenarios GWT definidos |
| V-09 | Dependencias listadas | WARNING | L2 | Seccion de dependencias no vacia |
| V-10 | Errores definidos | WARNING | L2 | Al menos 1 tipo de error definido |

### Validacion humana

| Aspecto | Quien | Criterio |
|---------|-------|---------|
| Calidad de contenido | ARCHITECT o LEAD | Reglas de negocio correctas, contratos coherentes |
| Coherencia con L1 parent | ARCHITECT o LEAD | L2 no contradice L1 |
| Completitud de clasificacion | ARCHITECT | Todos los campos sensibles identificados |
| Viabilidad tecnica | LEAD | Dependencias existen, latencias alcanzables |

### Restricciones
- Reviewer != Autor (SELF_REVIEW_NOT_ALLOWED)
- Solo ARCHITECT y LEAD pueden aprobar
- Decision: APPROVED o CHANGES_REQUESTED (no existe "REJECTED definitivo")

---

## Gate 2: Tests + seguridad antes de revision de PR

**Cuando**: en cada PR de codigo generado desde spec.
**Bloqueante**: Si — PR no puede mergearse sin pasar.

### Validaciones automaticas (CI pipeline)

| # | Validacion | Herramienta | Criterio |
|---|-----------|-------------|---------|
| V-11 | Tests unitarios | dotnet test / ng test | 100% GWT de la spec cubiertos, cobertura >= 80% |
| V-12 | Tests de contrato | dotnet test --filter Contract | Contratos API coinciden con spec |
| V-13 | SAST | security-scan / SonarQube | 0 hallazgos criticos, 0 altos |
| V-14 | SCA | dotnet list --vulnerable / npm audit | 0 vulnerabilidades criticas |
| V-15 | DLP Scan - secretos | detect-secrets | 0 secretos detectados |
| V-16 | DLP Scan - PII | custom scanner | 0 PII en codigo, tests o fixtures |
| V-17 | Tamano de PR | git diff --stat | Alerta si > 400 lineas (no bloquea, pero alerta) |
| V-18 | Lint | dotnet format / eslint | 0 errores de formato |

---

## Gate 3: Revision senior antes del merge

**Cuando**: despues de pasar Gate 2, antes del merge a main.
**Bloqueante**: Si — revision humana obligatoria siempre.

### Checklist de revision

| # | Aspecto | Criterio |
|---|---------|---------|
| R-01 | Codigo en capa correcta | Domain/Application/Infrastructure segun ADR-001 |
| R-02 | Dependencias autorizadas | Solo las listadas en la spec L2 |
| R-03 | Errores implementados | Corresponden a los tipos de error de la spec |
| R-04 | Tests cubren spec | Escenarios GWT implementados correctamente |
| R-05 | Sin logica inventada | No hay reglas de negocio que no esten en la spec |
| R-06 | Proteccion de datos | Masking en logs para campos Sensible, datos sinteticos en tests |
| R-07 | Observabilidad | Logs estructurados, correlationId, metricas definidas |
| R-08 | Trazabilidad | Commit referencia spec ID y version |

### Restricciones
- Revisor con rol ARCHITECT o LEAD
- Revisor != Autor del PR
- Si el PR toca proteccion de datos, ARCHITECT debe revisar

---

## Resumen visual

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│    GATE 1       │     │    GATE 2       │     │    GATE 3       │
│                 │     │                 │     │                 │
│ Spec aprobada   │────>│ Tests + SAST +  │────>│ Revision senior │───> MERGE
│                 │     │ SCA + DLP       │     │ + proteccion    │
│ SK-004 + humano │     │ CI pipeline     │     │ datos           │
│                 │     │                 │     │                 │
│ BLOQUEANTE      │     │ BLOQUEANTE      │     │ BLOQUEANTE      │
└─────────────────┘     └─────────────────┘     └─────────────────┘
```
