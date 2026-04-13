# Especificaciones

Las especificaciones son el **componente central** del framework. Definen QUE construir, bajo que reglas y con que restricciones. Toda generacion de codigo parte de una especificacion versionada y aprobada.

---

## Modelo de 3 niveles

```
L1 (Dominio)  →  L2 (Sistema)  →  L3 (Cambio)
   QUE              COMO            DELTA
```

| Nivel | Nombre | Define | Ejemplo |
|-------|--------|--------|---------|
| **L1** | Dominio | Reglas de negocio, entidades, politicas | "Sistema de gestion de pedidos con validacion de inventario" |
| **L2** | Sistema | Contrato tecnico, dependencias, errores, tests | "API CreateOrder con input/output tipado y escenarios GWT" |
| **L3** | Cambio | Cambio incremental sobre un L2 | "Agregar campo descuento opcional al contrato de CreateOrder" |

### Relacion entre niveles
- Un L1 puede tener **varios L2**
- Un L2 puede tener **varios L3**
- Un L3 **pertenece** a un L2, que pertenece a un L1
- La trazabilidad es: L1 → L2 → L3 → codigo → commit → release
- **Un cambio NO debe mezclar niveles**

---

## Reglas de versionado

- Todas las especificaciones se versionan con **semver** (MAJOR.MINOR.PATCH)
- Todo cambio requiere **PR y revision tecnica**
- Las specs deben estar **aprobadas** antes de generar codigo (Gate 1)

| Tipo de cambio | Version | Ejemplo |
|---------------|---------|---------|
| Cambio mayor (rompe compatibilidad) | MAJOR | `1.0.0` → `2.0.0` |
| Nueva funcionalidad (compatible) | MINOR | `1.0.0` → `1.1.0` |
| Correccion/ajuste menor | PATCH | `1.0.0` → `1.0.1` |

---

## Estados de una especificacion

| Estado | Significado | Puede generar codigo? |
|--------|------------|----------------------|
| `DRAFT` | En construccion | No |
| `IN_REVIEW` | En revision via PR | No |
| `APPROVED` | Aprobada por arquitecto/lider | Si |
| `DEPRECATED` | Reemplazada por nueva version | No (usar la nueva) |

---

## Como crear una nueva especificacion

1. Copiar el template correspondiente de `templates/`
2. Renombrar segun convencion: `SPEC-L{n}-{dominio}-v{X.Y.Z}.md`
3. Llenar todos los campos obligatorios
4. Crear un PR para revision
5. Obtener aprobacion del arquitecto/lider tecnico
6. Cambiar estado a `APPROVED`
7. Ahora se puede generar codigo usando esta spec

---

## Contenido de cada carpeta

- `templates/` — Templates reutilizables para cada nivel (L1, L2, L3)
- `examples/` — Ejemplos completos usando el dominio "Gestion de Pedidos"
