# Account Planning Inteligente con IA

Primer producto real construido con el framework AI-Driven Engineering de Tech&Solve.

## Estructura

```
account-planning/
├── specs/
│   ├── SPEC-L1-account-planning.md          ← Dominio: reglas de negocio
│   ├── SPEC-L2-create-account-plan.md       ← Sistema: crear plan de cuenta
│   ├── SPEC-L2-identify-opportunities.md    ← Sistema: motor de oportunidades IA
│   └── SPEC-L3-auto-scoring.md              ← Cambio: scoring automatico
└── presentation/
    └── PITCH-TECHANDSOLVE.md                ← Propuesta para Tech&Solve
```

## Trazabilidad

```
L1: Account Planning (dominio)
├── L2: CreateAccountPlan v1.0.0 (crear plan con IA)
│     └── 8 dependencias, 9 escenarios GWT, health score, stakeholder mapping
├── L2: IdentifyOpportunities v1.0.0 (motor de oportunidades)
│     └── 6 dependencias, 8 escenarios GWT, matching con aceleradores, ROI
└── L3: AutoScoring v1.1.0 (scoring dinamico)
      └── 3 nuevas dependencias, 5 escenarios GWT, re-scoring periodico
```
