# Especificacion de Cambio (Nivel 3)

| Campo | Valor |
|-------|-------|
| **Spec ID** | SPEC-L3-{{change-name}} |
| **Nivel** | L3 — Cambio |
| **Version** | {{X.Y.Z}} |
| **Parent L2** | {{SPEC-L2-xxx}} v{{X.Y.Z}} |
| **Estado** | `DRAFT` | `IN_REVIEW` | `APPROVED` | `IMPLEMENTED` | `DEPRECATED` |
| **Creado por** | @{{autor}} |
| **Fecha de creacion** | {{YYYY-MM-DD}} |
| **Aprobado por** | @{{aprobador}} |
| **Fecha de aprobacion** | {{YYYY-MM-DD}} |
| **PR** | {{link al PR}} |

---

## Descripcion general

_Resumen en una o dos oraciones de que cambia y por que._

{{Ej: "Agregar campo de codigo de descuento opcional en la creacion de pedidos para soportar la campana de fidelizacion."}}

---

## Contexto

_Que motiva este cambio? Estado actual, necesidad de negocio o tecnica, decisiones previas relevantes._

---

## Motivacion

_Que problema resuelve este cambio especifico? Que valor aporta?_

{{Por que se necesita este cambio? Puede ser una necesidad de negocio o tecnica.}}

---

## Impacto en contrato

### Campos agregados

| Campo | Tipo | Requerido | Descripcion |
|-------|------|-----------|------------|
| {{campo_nuevo}} | {{tipo}} | Si/No | {{descripcion}} |

### Campos modificados

| Campo | Cambio | Antes | Despues |
|-------|--------|-------|---------|
| {{campo}} | {{tipo de cambio}} | {{valor/tipo anterior}} | {{valor/tipo nuevo}} |

### Campos eliminados

| Campo | Justificacion |
|-------|--------------|
| {{campo}} | {{por que se elimina}} |

> Si no hay cambios en alguna seccion, indicar: "Sin cambios"

---

## Impacto en comportamiento

_Que logica de negocio cambia? Que nuevas validaciones se agregan? Que flujo se modifica?_

**Comportamiento esperado tras el cambio:**

1. {{Paso 1 del flujo modificado...}}
2. {{Paso 2...}}
3. {{...}}

**Cambios puntuales:**

- {{Cambio de comportamiento 1}}
- {{Cambio de comportamiento 2}}

---

## Compatibilidad hacia atras

| Aspecto | Valor |
|---------|-------|
| **Tipo de cambio** | [Breaking | Non-breaking] |
| **Requiere migracion** | [Si | No] |
| **Notas de migracion** | {{si aplica, describir pasos}} |

---

## Dependencias afectadas

### Nuevas dependencias

| Dependencia | Proposito | Tipo |
|------------|----------|------|
| {{nueva_dependencia}} | {{para que}} | {{tipo}} |

> Si no hay nuevas dependencias, indicar: "Sin nuevas dependencias"

### Cambios en dependencias existentes

| Dependencia | Cambio |
|------------|--------|
| {{dependencia}} | {{que cambia}} |

> Si no hay cambios, indicar: "Sin cambios en dependencias"

---

## Restricciones y consideraciones

| Tipo | Descripcion |
|------|------------|
| **Limitaciones tecnicas** | {{restricciones especificas de este cambio}} |
| **Consideraciones de seguridad** | {{impacto en seguridad del cambio}} |
| **Consideraciones de rendimiento** | {{impacto en rendimiento del cambio}} |

> Si no aplica alguna, indicar: "Sin impacto"

---

## Criterios de aceptacion

- [ ] {{Criterio 1: descripcion verificable del cambio}}
- [ ] {{Criterio 2: descripcion verificable de compatibilidad}}
- [ ] {{Criterio 3: descripcion verificable de validaciones nuevas}}

---

## Escenarios de prueba afectados

### Tests existentes que requieren actualizacion

| ID del test original | Cambio necesario |
|---------------------|-----------------|
| {{TS-NNN}} | {{que debe cambiar en el test}} |

### Nuevos escenarios de prueba (GWT)

| ID | Given (Dado que) | When (Cuando) | Then (Entonces) | Prioridad |
|----|-----------------|--------------|-----------------|-----------|
| TS-{{NNN}} | {{precondicion}} | {{accion}} | {{resultado esperado}} | [Alta | Media | Baja] |
| TS-{{NNN}} | {{precondicion — caso de error}} | {{accion}} | {{error esperado}} | [Alta | Media | Baja] |

---

## Referencias

- {{Enlace a ticket / historia de usuario que origina el cambio}}
- {{Enlace a documentacion tecnica relevante}}
- {{Enlace a discusiones o decisiones previas}}

---

## Trazabilidad

```
L1: {{SPEC-L1-xxx}} v{{X.Y.Z}} (dominio)
  └── L2: {{SPEC-L2-xxx}} v{{X.Y.Z}} (sistema)
        └── L3: {{este spec}} v{{X.Y.Z}} (este cambio)
              └── Commits esperados: {{descripcion de los commits}}
```

---

## Historial de cambios

| Version | Fecha | Autor | Descripcion del cambio |
|---------|-------|-------|----------------------|
| {{X.Y.Z}} | {{YYYY-MM-DD}} | {{autor}} | {{descripcion}} |
