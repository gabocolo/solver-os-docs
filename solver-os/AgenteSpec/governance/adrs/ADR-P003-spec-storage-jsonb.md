# ADR-P003: Almacenamiento de specs en PostgreSQL JSONB

| Campo | Valor |
|-------|-------|
| **Estado** | ACCEPTED |
| **Fecha** | 2026-04-09 |
| **Derivado de** | ADR-001 (Ports & Adapters) |
| **Decisor** | Equipo de Arquitectura |

---

## Contexto

Las especificaciones tienen estructura variable segun su nivel (L1, L2, L3). Una L1 tiene entidades, reglas y clasificacion de datos. Una L2 tiene contrato tecnico, dependencias, errores y tests GWT. Una L3 tiene cambio, impacto en contrato e impacto en comportamiento. Necesitamos un modelo de almacenamiento que soporte esta flexibilidad sin sacrificar la capacidad de consulta.

---

## Decision

Usar PostgreSQL con columnas JSONB para el contenido variable de las specs, combinado con columnas relacionales para metadata consultable.

### Estructura

```sql
specifications (
    -- Metadata relacional (consultable, indexable)
    id, project_id, parent_spec_id, level, title, version, status,
    approved_by, approved_at, model, tokens_used, created_by, created_at,

    -- Contenido flexible (JSONB)
    content JSONB,           -- estructura variable por nivel
    data_classification JSONB -- requerido para L1
)
```

### Contenido JSONB por nivel

**L1 (content)**:
```json
{
  "objetivo": "...",
  "entidades": [...],
  "reglas": [...],
  "casosDeUso": [...],
  "validaciones": [...]
}
```

**L2 (content)**:
```json
{
  "descripcion": "...",
  "contratoTecnico": { "inputs": [...], "outputs": [...] },
  "dependencias": [...],
  "errores": [...],
  "requisitos": {...},
  "proteccionDatos": [...],
  "testsGWT": [...],
  "referencias": [...]
}
```

**L3 (content)**:
```json
{
  "cambio": "...",
  "impactoContrato": "...",
  "impactoComportamiento": "...",
  "testsImpactados": [...]
}
```

---

## Alternativas evaluadas

### Alternativa 1: Columnas relacionales para cada campo
- **Pro**: Schema estricto, validacion a nivel de DB
- **Contra**: 3 tablas distintas o columnas nullable. Schema rigido, dificil evolucionar
- **Descartada**: Complejidad excesiva para contenido que varia por nivel

### Alternativa 2: MongoDB (documento puro)
- **Pro**: Nativo para documentos JSON, flexible
- **Contra**: Pierde relaciones SQL (joins con projects, users, reviews). Otro motor a mantener
- **Descartada**: PostgreSQL con JSONB ofrece lo mejor de ambos mundos

### Alternativa 3: Archivos markdown en Git
- **Pro**: Versionado nativo con Git, diffable
- **Contra**: No permite queries, no soporta dashboard de metricas, no es transaccional
- **Descartada**: No es viable para un portal interactivo con metricas

---

## Consecuencias

### Positivas
- Un solo motor (PostgreSQL) para todo
- Metadata indexable para queries del dashboard
- Contenido flexible que evoluciona sin migraciones de schema
- JSONB soporta operadores de consulta (@>, ?, #>>)

### Negativas
- Validacion de estructura del JSONB se hace en aplicacion, no en DB
- Queries sobre campos profundos de JSONB son mas lentas que columnas
- Requiere indices GIN para consultas frecuentes sobre JSONB

### Indices recomendados
```sql
CREATE INDEX idx_specs_content_gin ON specifications USING GIN (content);
CREATE INDEX idx_specs_data_class_gin ON specifications USING GIN (data_classification);
```
