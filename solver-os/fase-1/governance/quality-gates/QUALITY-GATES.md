# Quality Gates

Los Quality Gates son puntos de control obligatorios en el flujo de desarrollo. Ningun cambio puede avanzar sin pasar cada gate en orden.

---

## Vision general

```
Spec aprobada ──→ Generacion ──→ Validacion automatizada ──→ Revision humana ──→ Merge
     Gate 1                           Gate 2                      Gate 3
```

---

## Gate 1: Aprobacion de especificacion

**Cuando:** Antes de generar cualquier codigo  
**Quien:** Arquitecto / Lider tecnico  
**Bloqueante:** Si — sin spec aprobada, NO se genera codigo

### Checklist

- [ ] La especificacion (L1, L2 o L3) existe y esta versionada con semver
- [ ] La especificacion fue revisada via PR
- [ ] La especificacion fue aprobada por el arquitecto o lider tecnico
- [ ] Los ADRs referenciados en la spec estan vigentes (no DEPRECATED)
- [ ] Las dependencias autorizadas estan explicitamente listadas (para L2)
- [ ] Los escenarios de prueba GWT estan definidos (para L2)
- [ ] La trazabilidad es clara: L3 → L2 → L1

### Si falla
La especificacion debe ser corregida y re-aprobada antes de proceder. No se permite generar codigo con specs en estado DRAFT o IN_REVIEW.

---

## Gate 2: Validacion automatizada

**Cuando:** Antes de la revision humana (en CI/CD)  
**Quien:** Pipeline automatizado  
**Bloqueante:** Si — si falla, no se solicita revision humana

### Checklist

- [ ] Tests unitarios pasan (basados en GWT de la spec)
- [ ] Tests de contrato pasan (API segun spec L2)
- [ ] SAST (Static Application Security Testing) ejecutado — sin hallazgos criticos/altos sin resolver
- [ ] SCA (Software Composition Analysis) ejecutado — sin vulnerabilidades criticas/altas en dependencias
- [ ] Cobertura de tests cumple el umbral minimo: {{definir umbral, ej: 80%}}
- [ ] No se detectan dependencias no autorizadas (comparar imports vs spec L2)

### Si falla
El desarrollador debe corregir los hallazgos. Los resultados de SAST/SCA deben documentarse. Hallazgos aceptados como riesgo deben tener justificacion escrita.

---

## Gate 3: Revision senior obligatoria

**Cuando:** Antes del merge  
**Quien:** Desarrollador senior / Lider tecnico  
**Bloqueante:** Si — sin aprobacion senior, NO se hace merge

### Checklist

- [ ] PR tiene menos de 400 lineas de cambio
- [ ] La intencion del cambio es clara (descripcion del PR)
- [ ] El codigo esta en la capa arquitectonica correcta (dominio, aplicacion, infraestructura)
- [ ] Las dependencias usadas coinciden con las autorizadas en la spec
- [ ] La trazabilidad esta documentada: commit referencia spec ID + version
- [ ] No hay logica de negocio inventada (no presente en la spec)
- [ ] Los tests cubren los escenarios definidos en la spec
- [ ] No hay hallazgos de seguridad abiertos sin justificacion
- [ ] El codigo sigue los estandares del Engineering Statute

### Si falla
El reviewer solicita cambios. El desarrollador corrige y solicita re-revision. El PR no puede ser mergeado hasta que el reviewer apruebe.

---

## Checklist de decision de arquitectura

Antes de generar codigo para cualquier cambio, verificar:

- [ ] Especificacion versionada y aprobada
- [ ] ADRs relevantes y vigentes
- [ ] PR quirurgico (< 400 lineas)
- [ ] Tests y calidad definidos antes de generar codigo

---

## Proceso de escalacion

| Situacion | Escalacion |
|-----------|-----------|
| Gate 1 falla: spec no aprobada | El desarrollador no puede generar codigo. Debe completar la spec y solicitar revision |
| Gate 2 falla: tests/seguridad | El desarrollador corrige. Si es un falso positivo, documenta justificacion |
| Gate 3 falla: revision rechazada | El desarrollador corrige segun feedback. Si hay desacuerdo tecnico, escalar al arquitecto |
| PR supera 400 lineas | Dividir el PR en cambios mas pequenos. Cada uno debe ser trazable a un spec L3 |
| Dependencia no autorizada detectada | Evaluar si se necesita. Si si, crear spec L3 para agregarla. Si no, eliminarla |
| Hallazgo de seguridad critico | Bloquear merge. Corregir antes de continuar. Documentar en el PR |

---

## Configuracion en CI/CD

Estas son guias genericas. Adaptar segun la herramienta de CI/CD del proyecto:

```yaml
# Ejemplo conceptual de pipeline
stages:
  - name: gate-2-validation
    steps:
      - run: tests (unit + contract)
      - run: sast-scan
      - run: sca-scan
      - check: coverage >= threshold
      - check: no-unauthorized-dependencies
    on_failure: block_pr

  - name: gate-3-review
    steps:
      - require: senior-approval (min 1)
      - check: pr-size <= 400 lines
    on_failure: block_merge
```

---

*Referencia: Framework AI-Driven Engineering v1.0 — Tech&Solve*
