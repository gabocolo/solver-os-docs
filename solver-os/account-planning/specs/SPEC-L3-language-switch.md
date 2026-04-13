# Especificacion de Cambio (Nivel 3)

| Campo | Valor |
|-------|-------|
| **Spec ID** | SPEC-L3-language-switch |
| **Nivel** | L3 — Cambio |
| **Version** | 1.1.0 |
| **Parent L2** | SPEC-L2-generate-portfolio v1.0.0 |
| **Estado** | DRAFT |
| **Creado por** | @juan.gabriel.colorado |
| **Fecha de creacion** | 2026-04-09 |
| **Aprobado por** | Pendiente |
| **Fecha de aprobacion** | Pendiente |
| **PR** | Pendiente |

---

## Descripcion general

Permitir al comercial cambiar el idioma del portfolio (espanol, ingles, portugues) sin relanzar la investigacion ni el analisis. Solo se regenera el documento en el nuevo idioma, preservando las ediciones manuales del comercial cuando sea posible.

---

## Contexto

- Tech&Solve opera principalmente en espanol pero tiene clientes en Brasil y mercados angloparlantes
- El equipo comercial necesita llevar portfolios en el idioma del interlocutor
- La investigacion y el analisis son costosos en tiempo (2-5 minutos) — no tiene sentido relanzarlos solo por un cambio de idioma
- El scope.md define esta funcionalidad como parte del MVP: "En cualquier momento, el comercial puede cambiar el idioma del portafolio. El cambio no relanza la investigacion — solo regenera el documento en el idioma seleccionado"

---

## Motivacion

Los comerciales preparan reuniones con interlocutores de diferentes paises e idiomas. Poder cambiar el idioma del portfolio sin perder tiempo en re-investigar maximiza la productividad y permite reutilizar una misma investigacion para multiples presentaciones.

---

## Impacto en contrato

### Campos agregados

Sin campos nuevos — `language` ya existe en el input de GeneratePortfolio y RegeneratePortfolio.

### Campos modificados

| Campo | Cambio | Antes | Despues |
|-------|--------|-------|---------|
| RegeneratePortfolio.language | Comportamiento clarificado | Idioma del portfolio | Al cambiar idioma, regenera SOLO el documento sin relanzar investigacion ni analisis. Las secciones editadas manualmente se traducen si es posible, o se mantienen en el idioma original con flag `translationPending=true` |

### Campos eliminados

Sin campos eliminados.

---

## Impacto en comportamiento

**Comportamiento esperado tras el cambio:**

1. El comercial tiene un portfolio en espanol (status DRAFT)
2. Solicita regenerar en ingles (RegeneratePortfolio con language=EN)
3. El sistema:
   a. Recupera el analisis original (NO relanza investigacion)
   b. Invoca el PortfolioAgent con language=EN
   c. Para secciones NO editadas: regenera en ingles
   d. Para secciones editadas manualmente: intenta traducir con el agente. Si la traduccion no es confiable, marca `translationPending=true` para que el comercial revise
4. El portfolio se actualiza con el nuevo idioma
5. El comercial puede seguir editando secciones en el nuevo idioma

**Cambios puntuales:**

- La regeneracion con cambio de idioma NO relanza investigacion ni analisis
- Las secciones editadas manualmente se intentan traducir automaticamente
- Se agrega flag `translationPending` en secciones donde la traduccion automatica requiere revision

---

## Compatibilidad hacia atras

| Aspecto | Valor |
|---------|-------|
| **Tipo de cambio** | Non-breaking |
| **Requiere migracion** | No |
| **Notas de migracion** | Portfolios existentes sin cambios. El campo translationPending es nuevo pero opcional (default false) |

---

## Dependencias afectadas

### Nuevas dependencias

Sin nuevas dependencias — el PortfolioAgent ya soporta multiples idiomas.

### Cambios en dependencias existentes

| Dependencia | Cambio |
|------------|--------|
| PortfolioAgent | Debe soportar traduccion de secciones editadas manualmente ademas de generacion en idioma |

---

## Restricciones y consideraciones

| Tipo | Descripcion |
|------|------------|
| **Limitaciones tecnicas** | La traduccion automatica de texto editado manualmente puede no ser perfecta — por eso existe translationPending |
| **Consideraciones de seguridad** | Sin impacto |
| **Consideraciones de rendimiento** | La regeneracion en otro idioma debe ser < 30s (igual que generacion inicial) |

---

## Criterios de aceptacion

- [ ] El comercial puede cambiar el idioma del portfolio sin que se relance la investigacion
- [ ] Las secciones generadas por IA se regeneran correctamente en el nuevo idioma
- [ ] Las secciones editadas manualmente se traducen o se marcan como pendientes de revision
- [ ] El portfolio mantiene las 15 secciones con contenido coherente en el nuevo idioma
- [ ] Se soportan los 3 idiomas: ES, EN, PT

---

## Escenarios de prueba afectados

### Tests existentes que requieren actualizacion

| ID del test original | Cambio necesario |
|---------------------|-----------------|
| TS-006 (generate-portfolio) | Validar que la regeneracion en EN no relanza investigacion ni analisis — solo regenera documento |

### Nuevos escenarios de prueba (GWT)

| ID | Given (Dado que) | When (Cuando) | Then (Entonces) | Prioridad |
|----|-----------------|--------------|-----------------|-----------|
| TS-014 | Un portfolio en espanol con 15 secciones (3 editadas manualmente) | Se regenera con language=EN | Las 12 secciones no editadas se generan en ingles. Las 3 editadas se traducen o se marcan con translationPending=true | Alta |
| TS-015 | Un portfolio en espanol | Se regenera con language=PT | Portfolio completo en portugues, investigacion y analisis no se relanzaron | Alta |
| TS-016 | Un portfolio en ingles con secciones traducidas y marcadas translationPending | El comercial edita la seccion pendiente | translationPending pasa a false, isEdited se mantiene true | Media |

---

## Referencias

- SPEC-L1-account-planning v2.0.0 — BR-007 (cambio de idioma sin re-investigar)
- context/scope.md — Seccion "Cambio de idioma"

---

## Trazabilidad

```
L1: SPEC-L1-account-planning v2.0.0 (dominio)
  └── L2: SPEC-L2-generate-portfolio v1.0.0 (generacion de portfolio)
        └── L3: SPEC-L3-language-switch v1.1.0 (cambio de idioma)
              └── Commits esperados: agregar logica de traduccion en PortfolioAgent, agregar campo translationPending en PortfolioSection
```

---

## Historial de cambios

| Version | Fecha | Autor | Descripcion del cambio |
|---------|-------|-------|----------------------|
| 1.1.0 | 2026-04-09 | Juan Gabriel Colorado | Version inicial: cambio de idioma sin re-investigacion |
