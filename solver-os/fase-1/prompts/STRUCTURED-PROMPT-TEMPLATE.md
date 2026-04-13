# Template de prompt estructurado

Usa este template para construir prompts que invoquen especificaciones. Copia, llena los campos y usalo con la herramienta de IA de tu preferencia.

---

## Estructura del prompt

```
## Rol
Actua como un {{rol}} (ej: desarrollador senior, QA engineer, arquitecto).

## Invocacion de especificacion
Aplica la especificacion {{spec-id}} version {{X.Y.Z}}.
{{Adjuntar el contenido de la especificacion como contexto}}

## Tarea
{{Descripcion de la tarea especifica y concreta. Una sola tarea por prompt.}}

## Entrada
{{Tipo de input segun la spec. Referenciar la seccion "Input" del spec L2.}}

## Salida esperada
{{Tipo de output segun la spec. Referenciar la seccion "Output" del spec L2.}}

## Reglas
- Todas las reglas de negocio, dependencias autorizadas, criterios de aceptacion
  y salidas esperadas provienen EXCLUSIVAMENTE de la especificacion referenciada.
- NO inventes servicios, reglas ni dependencias fuera de la spec.
- Solo usa las dependencias listadas en la seccion "Dependencias permitidas" de la spec.
- Los errores deben corresponder a los "Tipos de error" de la spec.
- El codigo debe ubicarse en la capa arquitectonica correcta segun ADR-001.

## Contexto de gobernanza adjunto
{{Listar los ADRs y estandares que se adjuntan como contexto:}}
- ADR-001: Ports & Adapters
- ADR-002: Idempotency (si aplica)
- CODE-STANDARDS.md (secciones relevantes)

## Restricciones de salida
- Diff pequeno y revisable (< 400 lineas)
- Codigo en la capa correcta (domain/application/infrastructure)
- Tests incluidos basados en los escenarios GWT de la spec
- Cada archivo generado debe tener un proposito claro
```

---

## Guia de uso

### Paso 1: Verificar Gate 1
Antes de usar este template, verificar que la especificacion esta:
- [ ] Versionada con semver
- [ ] Aprobada (estado: APPROVED)
- [ ] Con ADRs referenciados vigentes

### Paso 2: Seleccionar contexto minimo
Solo adjuntar al prompt:
- El spec completo (L2 o L3 segun la tarea)
- Los ADRs referenciados en la spec
- Los estandares relevantes (no todos)

### Paso 3: Una tarea por prompt
No pedir multiples cosas. Ejemplos de buena granularidad:
- "Crea el caso de uso CreateOrder segun la spec"
- "Genera los tests GWT para los escenarios TS-001 a TS-004"
- "Implementa el DiscountPort segun la spec"

### Paso 4: Revisar la salida
Antes de hacer PR, verificar:
- [ ] El codigo esta en la capa correcta
- [ ] Las dependencias coinciden con la spec
- [ ] Los tests cubren los escenarios de la spec
- [ ] No hay logica inventada

---

## Variables del template

| Variable | Descripcion | Ejemplo |
|----------|------------|---------|
| `{{rol}}` | Rol que debe asumir la IA | `desarrollador senior`, `QA engineer` |
| `{{spec-id}}` | ID de la especificacion | `SPEC-L2-create-order` |
| `{{X.Y.Z}}` | Version de la especificacion | `1.0.0` |
| `{{tarea}}` | Descripcion de la tarea | `Crear el caso de uso CreateOrder` |
