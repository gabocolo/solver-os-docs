# Especificacion de Cambio (Nivel 3)

| Campo | Valor |
|-------|-------|
| **Spec ID** | SPEC-L3-manual-linkedin-fallback |
| **Nivel** | L3 — Cambio |
| **Version** | 1.1.0 |
| **Parent L2** | SPEC-L2-execute-research v1.0.0 |
| **Estado** | DRAFT |
| **Creado por** | @juan.gabriel.colorado |
| **Fecha de creacion** | 2026-04-09 |
| **Aprobado por** | Pendiente |
| **Fecha de aprobacion** | Pendiente |
| **PR** | Pendiente |

---

## Descripcion general

Agregar la capacidad de que el comercial pegue manualmente el texto de un perfil de LinkedIn cuando el sistema no puede obtenerlo automaticamente. LinkedIn bloquea scraping y APIs — este fallback garantiza que la informacion de LinkedIn siempre este disponible para el analisis.

---

## Contexto

- LinkedIn no permite obtener informacion ni siquiera a traves de APIs facilmente (mencionado por Andres Muñeton en la sesion tecnica)
- Los comerciales actualmente copian y pegan publicaciones de LinkedIn a ChatGPT manualmente (mencionado por Yosheleen)
- El equipo usa el flujo: "Te vas a sus publicaciones y copias la pagina, la trasladamos a la IA"
- Sin LinkedIn, se pierde informacion valiosa sobre stakeholders, publicaciones y contexto de las personas clave
- El scope.md define: "LinkedIn: intento automatico de obtener el perfil; si no es posible, el comercial puede pegar el texto manualmente"

---

## Motivacion

LinkedIn es la fuente mas rica de informacion sobre personas clave y contexto corporativo, pero su acceso automatico es inestable. Este fallback permite que el sistema siempre tenga acceso a la informacion de LinkedIn — sea automatica o manual — sin bloquear el flujo de investigacion.

---

## Impacto en contrato

### Campos agregados

| Campo | Tipo | Requerido | Descripcion |
|-------|------|-----------|------------|
| linkedinManualInput | string | No | Texto del perfil de LinkedIn pegado manualmente por el comercial. Se procesa como fuente LINKEDIN |

> Este campo se agrega en el input de `ConfirmAndStartDeepSearch` (ya definido en L2 execute-research pero se detalla aqui el comportamiento)

### Campos modificados

Sin campos modificados.

### Campos eliminados

Sin campos eliminados.

---

## Impacto en comportamiento

**Comportamiento esperado tras el cambio:**

1. Durante la busqueda profunda, el sistema intenta obtener el perfil de LinkedIn automaticamente
2. Si el auto-fetch falla:
   a. El sistema marca la fuente LinkedIn como LINKEDIN_UNAVAILABLE en el status
   b. La busqueda profunda **continua** con web y RAG (no se bloquea)
   c. En el progreso, se indica al frontend que LinkedIn no esta disponible
   d. El frontend muestra al comercial un area de texto para pegar el perfil de LinkedIn manualmente
3. Si el comercial pega texto (linkedinManualInput):
   a. El sistema procesa el texto como fuente LINKEDIN
   b. Extrae hallazgos (publicaciones, informacion de la empresa, contexto)
   c. Extrae stakeholders adicionales si los hay
   d. Los hallazgos se agregan a la investigacion con source="LinkedIn (manual)" y type=LINKEDIN
4. Si el comercial no pega texto:
   a. La investigacion completa sin hallazgos de LinkedIn
   b. El analisis y portfolio se generan con la informacion disponible (web + RAG)
   c. El sistema no falla — degrada elegantemente

**Cambios puntuales:**

- El progreso de la investigacion ahora incluye un indicador de si LinkedIn esta disponible o requiere input manual
- El frontend puede enviar linkedinManualInput en cualquier momento durante o despues de la busqueda profunda
- Los hallazgos de LinkedIn manual se marcan con source="LinkedIn (manual)" para distinguirlos de los automaticos

---

## Compatibilidad hacia atras

| Aspecto | Valor |
|---------|-------|
| **Tipo de cambio** | Non-breaking |
| **Requiere migracion** | No |
| **Notas de migracion** | linkedinManualInput es opcional. Investigaciones existentes sin cambios. Si no se envia, el comportamiento es identico al actual |

---

## Dependencias afectadas

### Nuevas dependencias

Sin nuevas dependencias.

### Cambios en dependencias existentes

| Dependencia | Cambio |
|------------|--------|
| LinkedInAdapter | Debe reportar claramente cuando el auto-fetch falla (no solo timeout generico) |
| StakeholderExtractor | Debe poder procesar texto pegado manualmente ademas de datos estructurados de LinkedIn |
| InvestigationRepository | Debe soportar agregar hallazgos a una investigacion en progreso (append, no solo crear completa) |

---

## Restricciones y consideraciones

| Tipo | Descripcion |
|------|------------|
| **Limitaciones tecnicas** | El texto pegado manualmente puede tener formato inconsistente. El StakeholderExtractor debe ser robusto ante diferentes formatos de texto de LinkedIn |
| **Consideraciones de seguridad** | Sanitizar el texto pegado antes de procesarlo con IA. No almacenar datos sensibles de terceros (correos, telefonos) sin consentimiento |
| **Consideraciones de rendimiento** | El procesamiento del texto manual debe ser < 10s. No debe retrasar la investigacion en curso |

---

## Criterios de aceptacion

- [ ] Si LinkedIn auto-fetch falla, la busqueda profunda continua sin bloquearse
- [ ] El frontend muestra un area de texto para input manual cuando LinkedIn no esta disponible
- [ ] El texto pegado se procesa correctamente y genera hallazgos de tipo LINKEDIN
- [ ] Los stakeholders del texto manual se extraen e incluyen en la investigacion
- [ ] Sin input manual ni auto-fetch, la investigacion completa con web + RAG
- [ ] Los hallazgos de LinkedIn manual se distinguen con source="LinkedIn (manual)"

---

## Escenarios de prueba afectados

### Tests existentes que requieren actualizacion

| ID del test original | Cambio necesario |
|---------------------|-----------------|
| TS-005 (execute-research) | Verificar que cuando LinkedIn falla, el progreso indica `linkedinAvailable=false` y el frontend puede mostrar el input manual |
| TS-006 (execute-research) | Ya cubre el caso de texto pegado — verificar que los hallazgos se crean con source="LinkedIn (manual)" |

### Nuevos escenarios de prueba (GWT)

| ID | Given (Dado que) | When (Cuando) | Then (Entonces) | Prioridad |
|----|-----------------|--------------|-----------------|-----------|
| TS-011 | LinkedIn auto-fetch fallo durante deep search | El comercial pega texto de las publicaciones de LinkedIn del CTO del cliente | Se generan hallazgos de tipo LINKEDIN con source="LinkedIn (manual)", se extrae el CTO como stakeholder | Alta |
| TS-012 | LinkedIn auto-fetch fallo y el comercial no pega texto | La busqueda profunda completa | Investigacion COMPLETED sin hallazgos de tipo LINKEDIN, analisis y portfolio se generan solo con web + RAG | Alta |
| TS-013 | La busqueda profunda ya completo sin LinkedIn | El comercial pega texto de LinkedIn despues | Los hallazgos de LinkedIn se agregan a la investigacion existente, el analisis puede regenerarse con mas informacion | Media |

---

## Referencias

- SPEC-L1-account-planning v2.0.0 — BR-006 (fallback manual de LinkedIn)
- context/scope.md — Seccion "Fuentes de investigacion" (LinkedIn: intento automatico + fallback manual)
- context/TranscriptNegocio.txt — Yosheleen: "Te vas a sus publicaciones y copias la pagina, la trasladamos a la IA"
- context/TranscriptSesiónPresentaciónAndres.txt — Discusion sobre limitaciones de LinkedIn API

---

## Trazabilidad

```
L1: SPEC-L1-account-planning v2.0.0 (dominio)
  └── L2: SPEC-L2-execute-research v1.0.0 (flujo de investigacion)
        └── L3: SPEC-L3-manual-linkedin-fallback v1.1.0 (fallback manual de LinkedIn)
              └── Commits esperados: agregar procesamiento de linkedinManualInput, actualizar progreso con linkedinAvailable flag, agregar source="LinkedIn (manual)" a hallazgos manuales
```

---

## Historial de cambios

| Version | Fecha | Autor | Descripcion del cambio |
|---------|-------|-------|----------------------|
| 1.1.0 | 2026-04-09 | Juan Gabriel Colorado | Version inicial: fallback manual de LinkedIn cuando auto-fetch falla |
