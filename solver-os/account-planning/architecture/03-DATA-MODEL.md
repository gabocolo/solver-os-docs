# Modelo de datos

## Diagrama entidad-relacion

```
┌──────────────────┐       ┌──────────────────────┐       ┌──────────────────┐
│     ACCOUNT      │       │    ACCOUNT_PLAN       │       │   STAKEHOLDER    │
├──────────────────┤       ├──────────────────────┤       ├──────────────────┤
│ id          UUID │◀──┐   │ id            UUID    │   ┌──▶│ id          UUID │
│ name      STRING │   ├───│ account_id    UUID FK │   │   │ plan_id    UUID FK│
│ industry  STRING │   │   │ owner_id      UUID FK │   │   │ name      STRING │
│ size        ENUM │   │   │ status        ENUM    │   │   │ role      STRING │
│ website   STRING │   │   │ period_start  DATE    │   │   │ type        ENUM │
│ country   STRING │   │   │ period_end    DATE    │   │   │ influence    INT │
│ health_score INT │   │   │ objectives    JSONB   │   │   │ relationship ENUM│
│ created_at  TSTZ │   │   │ health_score  JSONB   │   │   │ coverage    ENUM │
│ updated_at  TSTZ │   │   │ summary       JSONB   │   │   │ notes     STRING │
└──────────────────┘   │   │ created_at    TSTZ    │   │   │ email     STRING │
                       │   │ updated_at    TSTZ    │   │   │ created_at  TSTZ │
                       │   └──────────────────────┘   │   └──────────────────┘
                       │              │                │
                       │              │ 1:N            │ 1:N
                       │              ▼                │
                       │   ┌──────────────────────┐   │
                       │   │    OPPORTUNITY        │   │
                       │   ├──────────────────────┤   │
                       │   │ id            UUID    │   │
                       │   │ plan_id       UUID FK │───┘
                       │   │ type          ENUM    │
                       │   │ title        STRING   │
                       │   │ description  STRING   │
                       │   │ source        ENUM    │──── MANUAL | AI_SUGGESTED
                       │   │ score          INT    │
                       │   │ score_trend   ENUM    │──── UP | DOWN | STABLE
                       │   │ score_factors JSONB   │
                       │   │ complexity    ENUM    │
                       │   │ time_to_value ENUM    │
                       │   │ estimated_value DEC   │
                       │   │ roi_estimate  JSONB   │
                       │   │ matched_accel JSONB   │
                       │   │ ai_rationale STRING   │
                       │   │ status        ENUM    │──── IDENTIFIED | PROPOSED
                       │   │ last_scored_at TSTZ   │     | APPROVED | IN_PROGRESS
                       │   │ auto_scoring  JSONB   │     | DELIVERED | REJECTED
                       │   │ created_at    TSTZ    │
                       │   │ updated_at    TSTZ    │
                       │   └──────────────────────┘
                       │              │
                       │              │ 1:N
                       │              ▼
                       │   ┌──────────────────────┐
                       │   │   SCORE_HISTORY       │
                       │   ├──────────────────────┤
                       │   │ id            UUID    │
                       │   │ opportunity_id UUID FK│
                       │   │ score          INT    │
                       │   │ factors       JSONB   │
                       │   │ trigger       STRING  │──── INITIAL | RESCORING | MANUAL
                       │   │ scored_at     TSTZ    │
                       │   └──────────────────────┘
                       │
                       │   ┌──────────────────────┐
                       ├───│   ACTION_ITEM         │
                       │   ├──────────────────────┤
                       │   │ id            UUID    │
                       │   │ plan_id       UUID FK │
                       │   │ opportunity_id UUID FK│──── nullable
                       │   │ description  STRING   │
                       │   │ owner        STRING   │
                       │   │ due_date      DATE    │
                       │   │ priority      ENUM    │──── HIGH | MEDIUM | LOW
                       │   │ status        ENUM    │──── PENDING | IN_PROGRESS
                       │   │ completed_at  TSTZ    │     | COMPLETED | CANCELLED
                       │   │ created_at    TSTZ    │
                       │   └──────────────────────┘
                       │
                       │   ┌──────────────────────┐
                       ├───│   INTERACTION         │
                       │   ├──────────────────────┤
                       │   │ id            UUID    │
                       │   │ account_id    UUID FK │
                       │   │ type          ENUM    │──── MEETING | CALL | EMAIL
                       │   │ date          TSTZ    │     | WORKSHOP | DEMO | OTHER
                       │   │ summary      STRING   │
                       │   │ stakeholder_ids UUID[]│
                       │   │ next_actions  JSONB   │
                       │   │ logged_by     UUID FK │
                       │   │ created_at    TSTZ    │
                       │   └──────────────────────┘
                       │
                       │   ┌──────────────────────┐
                       │   │  CLIENT_PROCESS       │
                       │   ├──────────────────────┤
                       │   │ id            UUID    │
                       │   │ plan_id       UUID FK │
                       │   │ name         STRING   │
                       │   │ description  STRING   │
                       │   │ frequency     ENUM    │
                       │   │ effort_hours   DEC    │
                       │   │ involved_teams JSONB  │
                       │   │ current_tools  JSONB  │
                       │   │ pain_points    JSONB  │
                       │   │ automation_score INT  │
                       │   │ dimensions    JSONB   │
                       │   │ created_at    TSTZ    │
                       │   └──────────────────────┘
                       │
  ┌────────────────────┘ (tabla independiente)
  │
  │    ┌──────────────────────┐
  │    │   ACCELERATOR         │
  │    ├──────────────────────┤
  │    │ id            UUID    │
  │    │ name         STRING   │
  │    │ description  STRING   │
  │    │ industries   STRING[] │
  │    │ process_types STRING[]│
  │    │ components    JSONB   │
  │    │ customization JSONB   │
  │    │ success_metrics JSONB │
  │    │ version      STRING   │
  │    │ status        ENUM    │──── ACTIVE | DEPRECATED
  │    │ created_at    TSTZ    │
  │    │ updated_at    TSTZ    │
  │    └──────────────────────┘
  │
  │    ┌──────────────────────┐
  │    │   APP_USER            │
  │    ├──────────────────────┤
  │    │ id            UUID    │
  │    │ email        STRING   │
  │    │ name         STRING   │
  │    │ role          ENUM    │──── ADMIN | DIRECTOR |
  │    │ avatar_url   STRING   │     ACCOUNT_MANAGER | VIEWER
  │    │ created_at    TSTZ    │
  │    └──────────────────────┘
```

---

## Enums (C# → SQL Server)

En .NET los enums se definen en C# y EF Core los mapea como `nvarchar` o `int` en SQL Server:

```csharp
// Domain/Enums.cs
public enum AccountSize { Startup, SMB, Enterprise }
public enum PlanStatus { Draft, Active, Closed, Archived }
public enum StakeholderType { DecisionMaker, Influencer, Executor, Champion, Blocker }
public enum RelationshipLevel { Strong, Neutral, Weak, Unknown }
public enum CoverageStatus { Covered, Gap }
public enum OpportunityType { FullAgentization, PartialAgentization, AiAssisted, WorkflowAutomation, NewService, Upsell, CrossSell }
public enum OpportunitySource { Manual, AiSuggested }
public enum OpportunityStatus { Identified, Proposed, Approved, InProgress, Delivered, Rejected }
public enum ScoreTrend { Up, Down, Stable }
public enum ComplexityLevel { Low, Medium, High }
public enum TimeToValue { Weeks, Months, Quarters }
public enum ActionPriority { High, Medium, Low }
public enum ActionStatus { Pending, InProgress, Completed, Cancelled }
public enum InteractionType { Meeting, Call, Email, Workshop, Demo, Other }
public enum ProcessFrequency { Daily, Weekly, Monthly, Quarterly }
public enum UserRole { Admin, Director, AccountManager, Viewer }
```

EF Core configuration (almacenar como string para legibilidad):
```csharp
// En OnModelCreating
modelBuilder.Entity<Opportunity>()
    .Property(o => o.Type)
    .HasConversion<string>();
```

---

## JSON columns (EF Core Owned Types / nvarchar(max))

SQL Server usa `nvarchar(max)` con JSON. EF Core 8 soporta mapeo nativo de JSON columns.

### health_score (en account_plan)

```csharp
// Value Object (C# record)
public record HealthScore(
    int Overall,
    int Satisfaction,
    int Engagement,
    int GrowthPotential,
    int ServiceHealth,
    DateTimeOffset CalculatedAt);

// EF Core config
modelBuilder.Entity<AccountPlan>()
    .OwnsOne(p => p.HealthScore, b => b.ToJson());
```

```json
{
  "overall": 72,
  "satisfaction": 80,
  "engagement": 65,
  "growthPotential": 85,
  "serviceHealth": 60,
  "calculatedAt": "2026-04-08T14:00:00Z"
}
```

### roi_estimate (en opportunity)
```json
{
  "currentCostPerMonth": 6300,
  "projectedCostPerMonth": 1200,
  "monthlySavings": 5100,
  "implementationCost": 15000,
  "paybackMonths": 3
}
```

### matched_accel (en opportunity)
```json
[
  {
    "acceleratorId": "uuid",
    "name": "Inventory Reconciler",
    "fitScore": 87,
    "customizationNeeded": "Adaptar conectores a ERP del cliente"
  }
]
```

### auto_scoring (en opportunity)
```json
{
  "enabled": true,
  "frequency": "WEEKLY",
  "factors": ["MARKET_DATA", "IMPLEMENTATION_FEEDBACK", "CATALOG_CHANGES"],
  "lastRunAt": "2026-04-07T03:00:00Z"
}
```

---

## Indices clave (EF Core Fluent API)

```csharp
// En AppDbContext.OnModelCreating
modelBuilder.Entity<AccountPlan>(entity =>
{
    entity.HasIndex(p => p.AccountId);
    entity.HasIndex(p => p.Status);
    entity.HasIndex(p => p.IdempotencyKey).IsUnique();
});

modelBuilder.Entity<Opportunity>(entity =>
{
    entity.HasIndex(o => o.PlanId);
    entity.HasIndex(o => o.Score).IsDescending();
    entity.HasIndex(o => o.Status);
});

modelBuilder.Entity<Stakeholder>()
    .HasIndex(s => s.PlanId);

modelBuilder.Entity<Interaction>()
    .HasIndex(i => new { i.AccountId, i.Date })
    .IsDescending(false, true);

modelBuilder.Entity<ScoreHistory>()
    .HasIndex(h => new { h.OpportunityId, h.ScoredAt })
    .IsDescending(false, true);
```

SQL Server equivalente generado por EF Core migrations:
```sql
CREATE INDEX IX_AccountPlan_AccountId ON AccountPlans(AccountId);
CREATE UNIQUE INDEX IX_AccountPlan_IdempotencyKey ON AccountPlans(IdempotencyKey);
CREATE INDEX IX_Opportunity_Score ON Opportunities(Score DESC);
CREATE INDEX IX_Interaction_AccountId_Date ON Interactions(AccountId, Date DESC);
```

---

*Modelo de datos v2.0 — Account Planning Inteligente — Tech&Solve — Stack: .NET 8 + EF Core + SQL Server*
