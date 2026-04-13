# Wireframes de UI

Pantallas principales del Account Planning con IA.

---

## 1. Dashboard principal

```
┌──────────────────────────────────────────────────────────────────────────────┐
│  🏢 Account Planning                              [Juan Gabriel] [Settings] │
├──────────┬───────────────────────────────────────────────────────────────────┤
│          │                                                                   │
│ ■ Home   │  RESUMEN GENERAL                                    Periodo: Q2  │
│          │  ─────────────────────────────────────────────────────────────── │
│ □ Cuentas│                                                                   │
│          │  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐ ┌────────────┐ │
│ □ Planes │  │  12         │ │  47         │ │  $1.2M      │ │  73        │ │
│          │  │  Cuentas    │ │  Oportunid. │ │  Pipeline   │ │  Health    │ │
│ □ Oportu-│  │  activas    │ │  activas    │ │  total      │ │  promedio  │ │
│   nidades│  └─────────────┘ └─────────────┘ └─────────────┘ └────────────┘ │
│          │                                                                   │
│ □ Inter- │  CUENTAS POR HEALTH SCORE                                        │
│   acciones│  ─────────────────────────                                       │
│          │                                                                   │
│ □ Repor- │  ████████████████████████  Saludables (>70)    │  8 cuentas     │
│   tes    │  ████████████             En riesgo (50-70)    │  3 cuentas     │
│          │  ████                     Criticas (<50)       │  1 cuenta      │
│ □ Acele- │                                                                   │
│   radores│  TOP OPORTUNIDADES                                                │
│          │  ──────────────────                                               │
│ □ Config │  ┌────────────────────────────────────────────────────────────┐   │
│          │  │ Score │ Cuenta    │ Oportunidad           │ Valor  │ Estado│   │
│          │  │  92   │ RetailCo  │ Agente de reportes    │ $50K   │ ●New  │   │
│          │  │  87   │ RetailCo  │ Concil. inventario    │ $75K   │ ●New  │   │
│          │  │  74   │ FinanceCo │ Automat. compliance   │ $120K  │ ▶Prog │   │
│          │  │  68   │ LogiCo    │ Tracking automatico   │ $40K   │ ◐Prop │   │
│          │  └────────────────────────────────────────────────────────────┘   │
│          │                                                                   │
│          │  ACCIONES PENDIENTES (esta semana)                                │
│          │  ─────────────────────────────────                                │
│          │  □ Agendar demo con RetailCo (stakeholder: Dir. Ops) — Vence: 10 │
│          │  □ Enviar propuesta FinanceCo compliance — Vence: 12 Apr          │
│          │  ☑ Completar mapa stakeholders LogiCo — Completado                │
│          │                                                                   │
└──────────┴───────────────────────────────────────────────────────────────────┘
```

---

## 2. Vista de cuenta + Plan

```
┌──────────────────────────────────────────────────────────────────────────────┐
│  ← Cuentas  │  RetailCo                            [Editar] [Nuevo analisis]│
├──────────────┴───────────────────────────────────────────────────────────────┤
│                                                                              │
│  PERFIL                          HEALTH SCORE                                │
│  ────────                        ─────────────                               │
│  Industria: Retail               ┌─────────────────────┐                     │
│  Tamano: Enterprise              │                     │                     │
│  Servicios actuales:             │     ╭──────╮        │                     │
│   • Desarrollo backend           │    ╱   72   ╲       │    Satisfaction: 80 │
│   • Soporte L2                   │   ╱  /100    ╲      │    Engagement:   65 │
│  Dolores:                        │   ╲          ╱      │    Growth:       85 │
│   • Reportes manuales            │    ╲        ╱       │    Service:      60 │
│   • Inventario desactualizado    │     ╰──────╯        │                     │
│                                  └─────────────────────┘                     │
│                                                                              │
├──────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  MAPA DE STAKEHOLDERS                                                        │
│  ────────────────────                                                        │
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐     │
│  │                        DECISION MAKERS                              │     │
│  │  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐          │     │
│  │  │ Maria Lopez  │    │ Carlos Ruiz  │    │   [+ Agregar] │          │     │
│  │  │ CTO          │    │ CFO          │    │   GAP: CEO    │          │     │
│  │  │ ●●●●○ Infl.  │    │ ●●●○○ Infl.  │    │              │          │     │
│  │  │ 🟢 Strong    │    │ 🟡 Neutral   │    │              │          │     │
│  │  └──────────────┘    └──────────────┘    └──────────────┘          │     │
│  │                                                                     │     │
│  │                         INFLUENCERS                                 │     │
│  │  ┌──────────────┐    ┌──────────────┐                              │     │
│  │  │ Ana Torres   │    │ Pedro Gomez  │                              │     │
│  │  │ Dir. Ops     │    │ Arq. Soluc.  │                              │     │
│  │  │ ●●●●● Infl.  │    │ ●●●○○ Infl.  │                              │     │
│  │  │ 🟢 Strong    │    │ 🔴 Weak      │                              │     │
│  │  └──────────────┘    └──────────────┘                              │     │
│  │                                                                     │     │
│  │                          EXECUTORS                                  │     │
│  │  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐          │     │
│  │  │ Luis Mendez  │    │ Sofia Pena   │    │ Diego Vargas │          │     │
│  │  │ Lead Dev     │    │ QA Lead      │    │ DevOps       │          │     │
│  │  │ ●●○○○ Infl.  │    │ ●●○○○ Infl.  │    │ ●●●○○ Infl.  │          │     │
│  │  │ 🟢 Strong    │    │ 🟡 Neutral   │    │ 🟡 Neutral   │          │     │
│  │  └──────────────┘    └──────────────┘    └──────────────┘          │     │
│  └─────────────────────────────────────────────────────────────────────┘     │
│  ⚠️ Gap detectado: No hay relacion con el CEO (Decision Maker)               │
│                                                                              │
├──────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  OPORTUNIDADES                          Filtro: [Todas ▼]  [Score ▼]        │
│  ──────────────                                                              │
│                                                                              │
│  ┌────────────────────────────────────────────────────────────────────┐      │
│  │ 🤖 Score: 92  │  Agente de reportes automaticos                    │      │
│  │ AI Suggested  │  Tipo: FULL_AGENTIZATION  │ Complejidad: LOW       │      │
│  │               │                                                    │      │
│  │  Ahorro: $4,200/mes  │  Payback: 2 meses  │  Time to value: WEEKS │      │
│  │                                                                    │      │
│  │  Acelerador: Report Automator v2 (fit: 94%)                        │      │
│  │                                                                    │      │
│  │  IA dice: "El proceso de reportes diarios consume 10 hrs/semana    │      │
│  │  con alta repetitividad (5/5) y es 100% basado en reglas.          │      │
│  │  Candidato ideal para agentizacion completa."                      │      │
│  │                                                                    │      │
│  │  Score trend: ↗ UP (was 88 → now 92, new accelerator matched)      │      │
│  │                                                   [Proponer] [▼]   │      │
│  └────────────────────────────────────────────────────────────────────┘      │
│                                                                              │
│  ┌────────────────────────────────────────────────────────────────────┐      │
│  │ 🤖 Score: 87  │  Conciliacion de inventario automatica             │      │
│  │ AI Suggested  │  Tipo: FULL_AGENTIZATION  │ Complejidad: MEDIUM    │      │
│  │               │                                                    │      │
│  │  Ahorro: $6,300/mes  │  Payback: 3 meses  │  Time to value: MONTHS│      │
│  │  Acelerador: Inventory Reconciler (fit: 87%)                       │      │
│  │                                                   [Proponer] [▼]   │      │
│  └────────────────────────────────────────────────────────────────────┘      │
│                                                                              │
├──────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  PLAN DE ACCION                                                              │
│  ──────────────                                                              │
│                                                                              │
│  HIGH                                                                        │
│  ☐ Agendar demo de Report Automator con Dir. Ops (Ana Torres)  — 10 Apr     │
│  ☐ Preparar business case para conciliacion inventario          — 15 Apr     │
│  ☐ Establecer relacion con CEO (gap de stakeholders)            — 20 Apr     │
│                                                                              │
│  MEDIUM                                                                      │
│  ☐ Proponer POC de catalogo asistido por IA                     — 30 Apr     │
│  ☐ Revisar contrato de soporte L2 para upsell                  — 30 Apr     │
│                                                                              │
│  Pipeline total: $226,800 USD/ano                                            │
│                                                                              │
└──────────────────────────────────────────────────────────────────────────────┘
```

---

## 3. Crear nuevo plan (wizard)

```
┌──────────────────────────────────────────────────────────────────────────────┐
│  Nuevo Account Plan                                          Step 1 de 4     │
├──────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ● Perfil  ○ Stakeholders  ○ Procesos  ○ Revision                           │
│  ─────────────────────────────────────────────                               │
│                                                                              │
│  Cuenta:         [ RetailCo                    ▼ ]                           │
│  Periodo:        [ 2026-04-01 ] a [ 2026-06-30 ]                            │
│  Responsable:    [ Juan Gabriel Colorado       ▼ ]                           │
│                                                                              │
│  Industria:      [ Retail                      ▼ ]                           │
│  Tamano:         ○ Startup  ● Enterprise  ○ SMB                              │
│                                                                              │
│  Servicios actuales:                                                         │
│  [x] Desarrollo backend   [x] Soporte L2   [ ] DevOps                       │
│  [ ] QA                   [ ] Arquitectura  [ ] Otro: [________]             │
│                                                                              │
│  Dolores / Pain points:                                                      │
│  ┌──────────────────────────────────────────────────────────────┐            │
│  │ • Reportes manuales que consumen mucho tiempo               │            │
│  │ • Inventario desactualizado                                  │            │
│  │ • [+ Agregar dolor]                                          │            │
│  └──────────────────────────────────────────────────────────────┘            │
│                                                                              │
│  Objetivos estrategicos del cliente:                                         │
│  ┌──────────────────────────────────────────────────────────────┐            │
│  │ • Reducir costos operativos en 20%                           │            │
│  │ • Digitalizar procesos de warehouse                          │            │
│  │ • [+ Agregar objetivo]                                       │            │
│  └──────────────────────────────────────────────────────────────┘            │
│                                                                              │
│  Objetivos del plan (este trimestre):                                        │
│  ┌──────────────────────────────────────────────────────────────┐            │
│  │ • Identificar 3 procesos automatizables                      │            │
│  │ • Presentar propuesta de agente de reportes                  │            │
│  │ • [+ Agregar objetivo]                                       │            │
│  └──────────────────────────────────────────────────────────────┘            │
│                                                                              │
│                                              [Cancelar]  [Siguiente →]       │
└──────────────────────────────────────────────────────────────────────────────┘
```

```
┌──────────────────────────────────────────────────────────────────────────────┐
│  Nuevo Account Plan                                          Step 3 de 4     │
├──────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ○ Perfil  ○ Stakeholders  ● Procesos  ○ Revision                           │
│  ─────────────────────────────────────────────                               │
│                                                                              │
│  Procesos del cliente (la IA los analizara para detectar oportunidades)      │
│                                                                              │
│  ┌──────────────────────────────────────────────────────────────────────┐    │
│  │ Proceso #1                                                           │    │
│  │ Nombre:      [ Generacion de reportes de ventas                 ]    │    │
│  │ Descripcion: [ Extraer datos de 3 fuentes, consolidar en Excel, ]    │    │
│  │              [ formatear y enviar por email a 5 stakeholders     ]    │    │
│  │ Frecuencia:  ● Diario  ○ Semanal  ○ Mensual  ○ Trimestral           │    │
│  │ Hrs/semana:  [ 10 ]                                                  │    │
│  │ Equipos:     [ Ops, Finanzas ]                                       │    │
│  │ Herramientas:[ Excel, SAP, Email ]                                   │    │
│  │ Dolores:     [ Errores manuales, toma 2 horas diarias          ]    │    │
│  └──────────────────────────────────────────────────────────────────────┘    │
│                                                                              │
│  ┌──────────────────────────────────────────────────────────────────────┐    │
│  │ Proceso #2                                                           │    │
│  │ Nombre:      [ Conciliacion de inventario                       ]    │    │
│  │ ...                                                                  │    │
│  └──────────────────────────────────────────────────────────────────────┘    │
│                                                                              │
│  [+ Agregar proceso]                                                         │
│                                                                              │
│  💡 Tip: Mientras mas detalle en la descripcion y dolores,                   │
│     mejores seran las sugerencias de la IA.                                  │
│                                                                              │
│                                    [← Anterior]  [Analizar con IA →]         │
│                                                                              │
│  Al hacer clic en "Analizar con IA", el sistema:                             │
│  1. Evaluara cada proceso en 5 dimensiones de automatizacion                 │
│  2. Identificara oportunidades de agentizacion                               │
│  3. Cruzara con el catalogo de aceleradores de Tech&Solve                    │
│  4. Calculara ROI estimado                                                   │
│  ⏱ Esto puede tomar 10-15 segundos                                          │
│                                                                              │
└──────────────────────────────────────────────────────────────────────────────┘
```

---

## 4. Pipeline de oportunidades (vista Director)

```
┌──────────────────────────────────────────────────────────────────────────────┐
│  Pipeline de oportunidades               Periodo: Q2 2026  [Exportar CSV]   │
├──────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  RESUMEN                                                                     │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐     │
│  │ $1.2M    │  │ 47       │  │ 12       │  │ 8        │  │ 5        │     │
│  │ Pipeline │  │ Total    │  │ Quick    │  │ En       │  │ Entrega- │     │
│  │ total    │  │ oportun. │  │ wins     │  │ progreso │  │ dos      │     │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘  └──────────┘     │
│                                                                              │
│  POR ESTADO (Kanban)                                                         │
│  ──────────────────                                                          │
│  IDENTIFIED(15)  PROPOSED(12)   APPROVED(8)    IN_PROGRESS(7)  DELIVERED(5) │
│  ┌──────────┐   ┌──────────┐  ┌──────────┐  ┌──────────┐   ┌──────────┐   │
│  │RetailCo  │   │FinanceCo │  │LogiCo    │  │HealthCo  │   │EduCo     │   │
│  │Catalogo  │   │Compliance│  │Tracking  │  │Reportes  │   │Enrollment│   │
│  │$34K      │   │$120K     │  │$40K      │  │$55K      │   │$28K      │   │
│  │Score: 74 │   │Score: 74 │  │Score: 68 │  │Score: 82 │   │Score: 90 │   │
│  ├──────────┤   ├──────────┤  ├──────────┤  ├──────────┤   ├──────────┤   │
│  │RetailCo  │   │RetailCo  │  │FinanceCo │  │LogiCo    │   │RetailCo  │   │
│  │Ordenes   │   │Reportes  │  │Auditing  │  │Manifiest.│   │Inventory │   │
│  │$25K      │   │$50K      │  │$90K      │  │$35K      │   │$75K      │   │
│  │Score: 68 │   │Score: 92 │  │Score: 71 │  │Score: 65 │   │Score: 87 │   │
│  └──────────┘   └──────────┘  └──────────┘  └──────────┘   └──────────┘   │
│  ...            ...           ...           ...              ...            │
│                                                                              │
│  POR CUENTA (tabla)                                                          │
│  ─────────────────                                                           │
│  │ Cuenta    │ Oport. │ Pipeline │ Health │ Quick wins │ AM              │   │
│  │ RetailCo  │   8    │  $226K   │   72   │    2       │ Juan Gabriel   │   │
│  │ FinanceCo │   6    │  $340K   │   65   │    1       │ Hernan         │   │
│  │ LogiCo    │   5    │  $180K   │   78   │    3       │ Oscar          │   │
│  │ HealthCo  │   4    │  $220K   │   82   │    1       │ Juan Gabriel   │   │
│                                                                              │
└──────────────────────────────────────────────────────────────────────────────┘
```

---

*Wireframes v2.0 — Account Planning Inteligente — Tech&Solve — Frontend: Angular 18 + Angular Material + Tailwind*
