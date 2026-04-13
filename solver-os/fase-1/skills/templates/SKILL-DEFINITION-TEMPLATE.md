# Definicion de Skill

| Campo | Valor |
|-------|-------|
| **ID** | SK-{{NNN}} |
| **Nombre** | {{nombre del skill}} |
| **Tipo** | [Build | Workflow] |
| **Version** | {{X.Y.Z}} |
| **Owner** | {{nombre del responsable (lider tecnico, arquitecto, QA lead)}} |
| **Fase del framework** | {{Fase N — nombre}} |
| **Estado** | [DRAFT | ACTIVE | DEPRECATED] |

---

## Descripcion

{{Que hace este skill. Debe ser una descripcion clara y concreta.}}

---

## Cuando usar

{{Condiciones bajo las cuales este skill debe ser invocado.}}

---

## Cuando NO usar

{{Condiciones bajo las cuales este skill NO debe ser invocado. Casos limite.}}

---

## Input requerido

| Input | Descripcion | Obligatorio |
|-------|------------|-------------|
| {{input 1}} | {{descripcion}} | Si/No |
| {{input 2}} | {{descripcion}} | Si/No |

---

## Output esperado

{{Que produce este skill. Artefactos, codigo, reportes, etc.}}

---

## Ejemplo de invocacion

```
Invoca el skill {{nombre}} con:
- Contexto del cambio: {{referencia}}
- Especificacion: {{spec-id}} version {{X.Y.Z}}
- {{otros parametros}}
```

---

## Metricas

| Metrica | Valor actual | Objetivo |
|---------|-------------|---------|
| % retrabajo | {{valor o "Por medir"}} | {{objetivo}} |
| Tiempo revision promedio | {{valor o "Por medir"}} | {{objetivo}} |
| Hallazgos asociados | {{valor o "Por medir"}} | {{objetivo}} |

---

## Historial de cambios

| Version | Fecha | Autor | Descripcion |
|---------|-------|-------|------------|
| {{X.Y.Z}} | {{YYYY-MM-DD}} | {{autor}} | {{cambio}} |
