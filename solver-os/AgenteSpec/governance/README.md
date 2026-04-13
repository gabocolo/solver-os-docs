# Gobernanza — Spec Agent Portal

Artefactos de gobernanza especificos del proyecto Spec Agent Portal. Estos complementan los ADRs transversales del framework (fase-1/governance/).

## Estructura

```
governance/
├── adrs/
│   ├── ADR-P001-llm-model-selection.md      ← Seleccion de modelos LLM por tipo de tarea
│   ├── ADR-P002-async-generation.md         ← Generacion asincrona via Service Bus
│   ├── ADR-P003-spec-storage-jsonb.md       ← Almacenamiento de specs en JSONB
│   ├── ADR-P004-devops-sync-strategy.md     ← Estrategia de sincronizacion con Azure DevOps
│   └── ADR-P005-dlp-filter-architecture.md  ← Arquitectura del filtro DLP bidireccional
├── quality-gates/
│   └── QUALITY-GATES-PROJECT.md             ← Quality gates configurados para este proyecto
└── README.md                                ← Este archivo
```

## Relacion con ADRs transversales

Los ADRs del proyecto (prefijo `ADR-P`) son decisiones locales derivadas de los ADRs transversales del framework:

| ADR Proyecto | Derivado de | Relacion |
|-------------|-------------|----------|
| ADR-P001 | Framework Seccion 11 (Costos) | Implementa la seleccion de modelo por tipo de tarea |
| ADR-P002 | ADR-002 (Idempotencia) | Garantiza idempotencia en generacion asincrona |
| ADR-P003 | ADR-001 (Ports & Adapters) | Decision de persistencia para la capa de infraestructura |
| ADR-P004 | ADR-003 (Trazabilidad) | Extiende trazabilidad hasta Azure DevOps |
| ADR-P005 | ADR-006 (DLP) | Implementa DLP bidireccional para integracion con LLM |
