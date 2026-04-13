# Flujos de secuencia

Diagramas de secuencia para cada caso de uso principal.

---

## Flujo 1: Crear Account Plan (CreateAccountPlan)

```
Actor           Angular            .NET API             AI Engine          SQL Server       EventBus
  │                │                  │                     │                  │                │
  │  Llenar form   │                  │                     │                  │                │
  │───────────────▶│                  │                     │                  │                │
  │                │  POST /api/plans │                     │                  │                │
  │                │─────────────────▶│                     │                  │                │
  │                │                  │                     │                  │                │
  │                │                  │── FluentValidation ┐│                  │                │
  │                │                  │◀─────────────────┘ │                  │                │
  │                │                  │                     │                  │                │
  │                │                  │── EF Core: check idempotencyKey ────▶│                │
  │                │                  │◀────────────────── (no existe) ──────│                │
  │                │                  │                     │                  │                │
  │                │                  │── EF Core: validar account ─────────▶│                │
  │                │                  │◀────────────────── (existe) ─────────│                │
  │                │                  │                     │                  │                │
  │                │                  │── Analizar stakeholder coverage ──┐  │                │
  │                │                  │◀──── (gaps identificados) ───────┘  │                │
  │                │                  │                     │                  │                │
  │                │                  │── Calcular health score ──────────┐  │                │
  │                │                  │◀──── (score: 72) ────────────────┘  │                │
  │                │                  │                     │                  │                │
  │                │                  │  POST /analyze       │                  │                │
  │                │                  │────────────────────▶│                  │                │
  │                │                  │                     │── Analizar       │                │
  │                │                  │                     │   procesos con   │                │
  │                │                  │                     │   Claude Sonnet  │                │
  │                │                  │                     │                  │                │
  │                │                  │                     │── Matchear vs    │                │
  │                │                  │                     │   catalogo       │                │
  │                │                  │                     │   aceleradores   │                │
  │                │                  │                     │                  │                │
  │                │                  │                     │── Calcular ROI   │                │
  │                │                  │                     │   por oportunidad│                │
  │                │                  │                     │                  │                │
  │                │                  │  AI Opportunities    │                  │                │
  │                │                  │◀────────────────────│                  │                │
  │                │                  │                     │                  │                │
  │                │                  │── Merge: manual + AI opportunities ─┐ │                │
  │                │                  │── Priorizar por score ─────────────┘ │                │
  │                │                  │                     │                  │                │
  │                │                  │── Generar action plan ─────────────┐  │                │
  │                │                  │◀──── (acciones priorizadas) ───────┘  │                │
  │                │                  │                     │                  │                │
  │                │                  │── EF Core SaveChangesAsync ──────────▶│                │
  │                │                  │   (transaction: plan + stakeholders + │                │
  │                │                  │    opportunities + actions + score)   │                │
  │                │                  │◀──────────────── COMMIT ─────────────│                │
  │                │                  │                     │                  │                │
  │                │                  │── Emit: AccountPlanCreated ─────────────────────────▶│
  │                │                  │                     │                  │                │
  │                │  201 Created     │                     │                  │                │
  │                │  { planId, ...}  │                     │                  │                │
  │                │◀─────────────────│                     │                  │                │
  │  Angular route │                  │                     │                  │                │
  │  → plan detail │                  │                     │                  │                │
  │◀───────────────│                  │                     │                  │                │
```

### Tiempos estimados

| Paso | Latencia |
|------|---------|
| Validaciones + DB reads | ~50ms |
| Stakeholder analysis | ~20ms |
| Health score calculation | ~30ms |
| **AI analysis (Claude)** | **~2000-4000ms** |
| Merge + prioritization | ~30ms |
| Action plan generation | ~20ms |
| DB transaction (write) | ~50ms |
| Event publish | ~10ms |
| **Total** | **~2500-4500ms** |

---

## Flujo 2: Identificar oportunidades con IA (IdentifyOpportunities)

```
.NET API             AI Engine              Claude API           SQL Server
  │                     │                      │                     │
  │  POST /analyze      │                      │                     │
  │────────────────────▶│                      │                     │
  │                     │                      │                     │
  │                     │── Construir prompt    │                     │
  │                     │   con procesos del    │                     │
  │                     │   cliente + industria │                     │
  │                     │                      │                     │
  │                     │── Enviar a Claude ───▶│                     │
  │                     │   "Analiza estos      │                     │
  │                     │    procesos y         │                     │
  │                     │    evalua cada uno    │                     │
  │                     │    en 5 dimensiones"  │                     │
  │                     │                      │                     │
  │                     │   Structured output   │                     │
  │                     │   (JSON con scores    │                     │
  │                     │    y rationale)       │                     │
  │                     │◀─────────────────────│                     │
  │                     │                      │                     │
  │                     │── Para cada proceso   │                     │
  │                     │   con score > 50:     │                     │
  │                     │                      │                     │
  │                     │── Buscar aceleradores ────────────────────▶│
  │                     │   por industria +     │                     │
  │                     │   tipo de proceso     │                     │
  │                     │◀──── aceleradores ────────────────────────│
  │                     │   matcheados          │                     │
  │                     │                      │                     │
  │                     │── Enviar a Claude ───▶│                     │
  │                     │   (Haiku) "Evalua     │                     │
  │                     │    fit entre proceso  │                     │
  │                     │    y acelerador"      │                     │
  │                     │◀── fitScore + notes ──│                     │
  │                     │                      │                     │
  │                     │── Calcular ROI:       │                     │
  │                     │   currentCost =       │                     │
  │                     │     effort * hourRate  │                     │
  │                     │   projectedCost =     │                     │
  │                     │     effort * (1-auto%) │                     │
  │                     │     * hourRate         │                     │
  │                     │   savings = diff       │                     │
  │                     │   payback = implCost   │                     │
  │                     │     / savings          │                     │
  │                     │                      │                     │
  │                     │── Ordenar por score   │                     │
  │                     │   compuesto:          │                     │
  │                     │   0.4*automation +    │                     │
  │                     │   0.3*roi +           │                     │
  │                     │   0.2*fit +           │                     │
  │                     │   0.1*timeToValue     │                     │
  │                     │                      │                     │
  │  Response:          │                      │                     │
  │  opportunities[]    │                      │                     │
  │  + processAnalysis[]│                      │                     │
  │  + summary          │                      │                     │
  │◀────────────────────│                      │                     │
```

### Prompt template para Claude (analisis de procesos)

```
Eres un consultor senior de automatizacion y agentizacion de procesos.

Industria del cliente: {industry}
Servicios actuales contratados: {currentServices}

Analiza los siguientes procesos del cliente y para cada uno evalua:
1. Repetitividad (1-5): que tan repetitivo y predecible es
2. Volumen (1-5): cuanto tiempo/esfuerzo consume
3. Basado en reglas (1-5): que tan determinista es (vs juicio humano)
4. Aplicabilidad cross-team (1-5): podria aplicarse en otros equipos/empresas
5. Disponibilidad de datos (1-5): hay datos estructurados para automatizar

Procesos:
{processes_json}

Para cada proceso con score promedio > 3, sugiere:
- Tipo de automatizacion: FULL_AGENTIZATION | PARTIAL_AGENTIZATION | AI_ASSISTED | WORKFLOW_AUTOMATION
- Descripcion de la oportunidad
- Rationale de por que recomiendas esto

Responde en JSON con el schema proporcionado.
No inventes procesos que no esten en la lista.
No sugieras servicios que el cliente ya tiene contratados.
```

---

## Flujo 3: Auto-scoring periodico

```
Hangfire Job          .NET API             AI Engine           SQL Server
  │                     │                     │                  │
  │── Trigger job ─────▶│                     │                  │
  │   (weekly cron)     │                     │                  │
  │                     │── Get opportunities  │                  │
  │                     │   with autoScoring   │                  │
  │                     │   enabled ──────────────────────────▶│
  │                     │◀──── opportunities[] ────────────────│
  │                     │                     │                  │
  │                     │── For each opportunity:               │
  │                     │                     │                  │
  │                     │   ┌─ Factor: CATALOG_CHANGES          │
  │                     │   │  Check new accelerators ────────▶│
  │                     │   │◀──── new matches ───────────────│
  │                     │   │  deltaScore += matchImpact       │
  │                     │   │                                   │
  │                     │   ├─ Factor: IMPLEMENTATION_FEEDBACK  │
  │                     │   │  Get similar implementations ──▶│
  │                     │   │◀──── success/failure rate ──────│
  │                     │   │  deltaScore += feedbackImpact    │
  │                     │   │                                   │
  │                     │   ├─ Factor: MARKET_DATA              │
  │                     │   │  POST /analyze-trend ──▶│         │
  │                     │   │◀── industry trend ─────│         │
  │                     │   │  deltaScore += trendImpact       │
  │                     │   │                                   │
  │                     │   └─ newScore = oldScore + delta      │
  │                     │                     │                  │
  │                     │── If |delta| > 10:  │                  │
  │                     │   Update score ────────────────────▶│
  │                     │   Save to score_history ────────────▶│
  │                     │   Emit OpportunityScoreChanged       │
  │                     │                     │                  │
  │                     │── If |delta| <= 10: │                  │
  │                     │   Update score only ────────────────▶│
  │                     │   (no event)        │                  │
  │                     │                     │                  │
  │◀── Job complete ────│                     │                  │
```

---

## Flujo 4: Dashboard / Consulta de plan

```
Actor           Angular                   .NET API             SQL Server
  │                │                        │                     │
  │  Abrir plan    │                        │                     │
  │───────────────▶│                        │                     │
  │                │  GET /api/plans/{id}    │                     │
  │                │───────────────────────▶│                     │
  │                │                        │── EF Core query ──▶│
  │                │                        │   + stakeholders    │
  │                │                        │   + opportunities   │
  │                │                        │   + actions         │
  │                │                        │   + score_history   │
  │                │                        │◀──── data ─────────│
  │                │                        │                     │
  │                │  200 OK (full plan)     │                     │
  │                │◀───────────────────────│                     │
  │                │                        │                     │
  │  Renderizar:   │                        │                     │
  │  • Health gauge│                        │                     │
  │  • Stakeholder │                        │                     │
  │    map visual  │                        │                     │
  │  • Oportunities│                        │                     │
  │    ranked list │                        │                     │
  │  • Score trend │                        │                     │
  │    chart       │                        │                     │
  │  • Action plan │                        │                     │
  │    timeline    │                        │                     │
  │  • Pipeline    │                        │                     │
  │    value       │                        │                     │
  │◀───────────────│                        │                     │
```

---

## Flujo 5: Registrar interaccion con cliente

```
Actor           Angular                 .NET API                SQL Server        EventBus
  │                │                      │                       │                  │
  │  Log meeting   │                      │                       │                  │
  │───────────────▶│                      │                       │                  │
  │                │  POST /api/interactions│                      │                  │
  │                │  { accountId, type,   │                       │                  │
  │                │    summary, stk[],    │                       │                  │
  │                │    nextActions[] }    │                       │                  │
  │                │─────────────────────▶│                       │                  │
  │                │                      │── Persist ───────────▶│                  │
  │                │                      │                       │                  │
  │                │                      │── Recalculate health  │                  │
  │                │                      │   (engagement dim) ──▶│                  │
  │                │                      │                       │                  │
  │                │                      │── Emit: InteractionLogged ────────────▶│
  │                │                      │   (triggers: update plan dashboard,    │
  │                │                      │    notify stakeholder owners)           │
  │                │                      │                       │                  │
  │                │  201 Created          │                       │                  │
  │                │◀─────────────────────│                       │                  │
  │◀───────────────│                      │                       │                  │
```

---

*Flujos de secuencia v2.0 — Account Planning Inteligente — Tech&Solve — Stack: Angular + .NET 8 + Python (AI)*
