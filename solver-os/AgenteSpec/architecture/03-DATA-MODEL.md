# 03 — Modelo de Datos

| Campo | Valor |
|-------|-------|
| **Proyecto** | Spec Agent Portal |
| **Version** | 1.0.0 |
| **Fecha** | 2026-04-09 |
| **Motor** | PostgreSQL 16 |

---

## Diagrama ER simplificado

```
┌──────────────┐       ┌──────────────────┐       ┌──────────────────┐
│   Project    │1     *│  Specification   │1     *│  ReviewCycle     │
│──────────────│───────│──────────────────│───────│──────────────────│
│ id (PK)      │       │ id (PK)          │       │ id (PK)          │
│ name (UQ)    │       │ project_id (FK)  │       │ spec_id (FK)     │
│ description  │       │ parent_spec_id   │       │ reviewer_id (FK) │
│ git_repo_url │       │ level (enum)     │       │ status (enum)    │
│ owner_id(FK) │       │ title            │       │ decision (enum)  │
│ devops_org   │       │ version          │       │ summary          │
│ devops_proj  │       │ status (enum)    │       │ created_at       │
│ devops_area  │       │ content (JSONB)  │       │ completed_at     │
│ devops_pat   │       │ data_class(JSONB)│       └──────────────────┘
│ created_at   │       │ approved_by (FK) │              │1
│ updated_at   │       │ approved_at      │              │
└──────────────┘       │ model            │       ┌──────▼───────────┐
       │1              │ tokens_used      │       │ ReviewComment    │
       │               │ created_by (FK)  │       │──────────────────│
       │               │ created_at       │       │ id (PK)          │
       │               │ updated_at       │       │ review_id (FK)   │
       │               └──────────────────┘       │ section          │
       │                      │1                  │ comment          │
       │               ┌──────▼───────────┐       │ created_by (FK)  │
       │               │  Prompt          │       │ created_at       │
       │               │──────────────────│       └──────────────────┘
       │               │ id (PK)          │
       │               │ spec_id (FK)     │
       │               │ spec_version     │
       │               │ skill_id (FK)    │
       │               │ content (JSONB)  │
       │               │ full_text        │
       │               │ generated_by(FK) │
       │               │ generated_at     │
       │               └──────────────────┘
       │
       │              ┌──────────────────┐       ┌──────────────────┐
       │*             │       ADR        │       │      Skill       │
       ├──────────────│──────────────────│       │──────────────────│
       │              │ id (PK)          │       │ id (PK)          │
       │              │ project_id (FK)  │       │ name (UQ)        │
       │              │ number           │       │ type (enum)      │
       │              │ title            │       │ version          │
       │              │ status (enum)    │       │ owner            │
       │              │ content (text)   │       │ phase            │
       │              │ created_by (FK)  │       │ criteria         │
       │              │ created_at       │       │ status (enum)    │
       │              │ updated_at       │       │ project_id (FK)  │
       │              └──────────────────┘       │ deprecated_reason│
       │                                         │ replaced_by (FK) │
       │              ┌──────────────────┐       │ created_at       │
       │              │ QualityGateCheck │       │ updated_at       │
       │              │──────────────────│       └──────────────────┘
       ├─────────────*│ id (PK)          │
       │              │ project_id (FK)  │       ┌──────────────────┐
       │              │ gate_number      │       │  WorkItemSync    │
       │              │ name             │       │──────────────────│
       │              │ validations(JSON)│       │ id (PK)          │
       │              │ blocking         │       │ spec_id (FK)     │
       │              │ created_at       │       │ project_id (FK)  │
       │              └──────────────────┘       │ devops_wi_id     │
       │                                         │ devops_wi_url    │
       │              ┌──────────────────┐       │ wi_type          │
       │              │    AuditLog      │       │ parent_wi_id     │
       │              │──────────────────│       │ sync_status(enum)│
       │              │ id (PK)          │       │ error            │
       │              │ project_id (FK)  │       │ synced_at        │
       │              │ entity_type      │       │ created_at       │
       │              │ entity_id        │       └──────────────────┘
       │              │ action           │
       │              │ actor_id (FK)    │       ┌──────────────────┐
       │              │ details (JSONB)  │       │      User        │
       │              │ correlation_id   │       │──────────────────│
       │              │ created_at       │       │ id (PK)          │
       │              └──────────────────┘       │ external_id      │
                                                 │ email (PII)      │
                                                 │ name (PII)       │
                                                 │ role (enum)      │
                                                 │ provider         │
                                                 │ created_at       │
                                                 │ last_login_at    │
                                                 └──────────────────┘
```

---

## Enums

### SpecLevel
```
L1    -- Dominio (negocio)
L2    -- Sistema (tecnico)
L3    -- Cambio (incremental)
```

### SpecStatus
```
DRAFT              -- Borrador editable
IN_REVIEW          -- En revision (Gate 1)
APPROVED           -- Aprobada
CHANGES_REQUESTED  -- Devuelta con comentarios (vuelve a DRAFT)
DEPRECATED         -- Deprecada
```

### ADRStatus
```
PROPOSED   -- Propuesto
ACCEPTED   -- Aceptado y vigente
DEPRECATED -- Reemplazado por otro ADR
SUPERSEDED -- Superado
```

### ReviewDecision
```
APPROVED           -- Spec aprobada
CHANGES_REQUESTED  -- Requiere cambios
```

### ReviewStatus
```
PENDING    -- Esperando decision del revisor
COMPLETED  -- Decision tomada
```

### SkillType
```
BUILD     -- Genera artefactos (codigo, tests, contratos)
WORKFLOW  -- Automatiza flujos (pipelines, validaciones)
```

### SkillStatus
```
ACTIVE      -- Disponible para uso
DEPRECATED  -- No usar en nuevos prompts
```

### SyncStatus
```
PENDING  -- En cola para sincronizacion
SYNCED   -- Work item creado exitosamente
FAILED   -- Error en sincronizacion
```

### UserRole
```
ARCHITECT   -- Arquitecto / lider de gobernanza
LEAD        -- Lider tecnico
SENIOR_DEV  -- Desarrollador senior
QA          -- Quality assurance
PO          -- Product owner
```

### AuditAction
```
PROJECT_CREATED
ADR_CREATED
ADR_GENERATED_AI
QUALITY_GATES_CONFIGURED
SPEC_CREATED
SPEC_CREATED_AI
SPEC_SECTION_REGENERATED
SPEC_SUBMITTED_FOR_REVIEW
SPEC_APPROVED
SPEC_CHANGES_REQUESTED
PROMPT_GENERATED
SKILL_CREATED
SKILL_UPDATED
SKILL_DEPRECATED
DEVOPS_WORK_ITEM_CREATED
DEVOPS_SYNC_FAILED
DEVOPS_SYNC_RETRIED
DEVOPS_CONFIGURED
```

---

## Tablas detalladas

### projects

```sql
CREATE TABLE projects (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name            VARCHAR(200) NOT NULL UNIQUE,
    description     TEXT,
    git_repo_url    VARCHAR(500),
    owner_id        UUID NOT NULL REFERENCES users(id),
    devops_org      VARCHAR(200),
    devops_project  VARCHAR(200),
    devops_area     VARCHAR(300),
    devops_pat_enc  BYTEA,  -- PAT cifrado (ADR-006: Sensible)
    created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at      TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
```

### specifications

```sql
CREATE TABLE specifications (
    id                  UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    project_id          UUID NOT NULL REFERENCES projects(id),
    parent_spec_id      UUID REFERENCES specifications(id),
    level               VARCHAR(2) NOT NULL CHECK (level IN ('L1','L2','L3')),
    title               VARCHAR(300) NOT NULL,
    version             VARCHAR(20) NOT NULL,  -- semver X.Y.Z
    status              VARCHAR(20) NOT NULL DEFAULT 'DRAFT',
    content             JSONB NOT NULL,        -- contenido flexible por nivel
    data_classification JSONB,                 -- requerido para L1
    approved_by         UUID REFERENCES users(id),
    approved_at         TIMESTAMPTZ,
    model               VARCHAR(50),           -- modelo LLM usado (si aplica)
    tokens_used         INTEGER,               -- tokens consumidos (si aplica)
    created_by          UUID NOT NULL REFERENCES users(id),
    created_at          TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at          TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    CONSTRAINT uq_spec_project_title_level_version
        UNIQUE (project_id, title, level, version),
    CONSTRAINT chk_parent_required
        CHECK (
            (level = 'L1' AND parent_spec_id IS NULL) OR
            (level IN ('L2','L3') AND parent_spec_id IS NOT NULL)
        ),
    CONSTRAINT chk_data_classification_l1
        CHECK (
            level != 'L1' OR data_classification IS NOT NULL
        )
);

CREATE INDEX idx_specs_project ON specifications(project_id);
CREATE INDEX idx_specs_parent ON specifications(parent_spec_id);
CREATE INDEX idx_specs_status ON specifications(status);
CREATE INDEX idx_specs_level ON specifications(level);
```

### adrs

```sql
CREATE TABLE adrs (
    id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    project_id  UUID NOT NULL REFERENCES projects(id),
    number      INTEGER NOT NULL,
    title       VARCHAR(300) NOT NULL,
    status      VARCHAR(20) NOT NULL DEFAULT 'PROPOSED',
    content     TEXT NOT NULL,  -- markdown (Confidencial: sanitizar antes de LLM)
    created_by  UUID NOT NULL REFERENCES users(id),
    created_at  TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at  TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    CONSTRAINT uq_adr_project_number UNIQUE (project_id, number)
);
```

### review_cycles

```sql
CREATE TABLE review_cycles (
    id           UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    spec_id      UUID NOT NULL REFERENCES specifications(id),
    reviewer_id  UUID NOT NULL REFERENCES users(id),
    status       VARCHAR(20) NOT NULL DEFAULT 'PENDING',
    decision     VARCHAR(20),
    summary      TEXT,
    created_at   TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    completed_at TIMESTAMPTZ,

    CONSTRAINT chk_author_ne_reviewer
        CHECK (reviewer_id != (SELECT created_by FROM specifications WHERE id = spec_id))
);

CREATE INDEX idx_reviews_spec ON review_cycles(spec_id);
CREATE INDEX idx_reviews_reviewer ON review_cycles(reviewer_id);
```

### review_comments

```sql
CREATE TABLE review_comments (
    id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    review_id   UUID NOT NULL REFERENCES review_cycles(id),
    section     VARCHAR(100) NOT NULL,
    comment     TEXT NOT NULL,
    created_by  UUID NOT NULL REFERENCES users(id),
    created_at  TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
```

### prompts

```sql
CREATE TABLE prompts (
    id            UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    spec_id       UUID NOT NULL REFERENCES specifications(id),
    spec_version  VARCHAR(20) NOT NULL,
    skill_id      UUID NOT NULL REFERENCES skills(id),
    content       JSONB NOT NULL,    -- PromptContent estructurado
    full_text     TEXT NOT NULL,      -- texto completo renderizado
    generated_by  UUID NOT NULL REFERENCES users(id),
    generated_at  TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_prompts_spec ON prompts(spec_id);
CREATE INDEX idx_prompts_skill ON prompts(skill_id);
```

### skills

```sql
CREATE TABLE skills (
    id                UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name              VARCHAR(200) NOT NULL,
    type              VARCHAR(10) NOT NULL CHECK (type IN ('BUILD','WORKFLOW')),
    version           VARCHAR(20) NOT NULL,
    owner             VARCHAR(200) NOT NULL,
    phase             VARCHAR(100) NOT NULL,
    criteria          TEXT,
    status            VARCHAR(15) NOT NULL DEFAULT 'ACTIVE',
    project_id        UUID REFERENCES projects(id),  -- null = global
    deprecated_reason TEXT,
    replaced_by       UUID REFERENCES skills(id),
    created_at        TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at        TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    CONSTRAINT uq_skill_name_version UNIQUE (name, version)
);
```

### quality_gate_checks

```sql
CREATE TABLE quality_gate_checks (
    id           UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    project_id   UUID NOT NULL REFERENCES projects(id),
    gate_number  INTEGER NOT NULL CHECK (gate_number BETWEEN 1 AND 3),
    name         VARCHAR(200) NOT NULL,
    validations  JSONB NOT NULL,
    blocking     BOOLEAN NOT NULL DEFAULT true,
    created_at   TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    CONSTRAINT uq_gate_project_number UNIQUE (project_id, gate_number)
);
```

### work_item_syncs

```sql
CREATE TABLE work_item_syncs (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    spec_id         UUID NOT NULL REFERENCES specifications(id),
    project_id      UUID NOT NULL REFERENCES projects(id),
    devops_wi_id    INTEGER,
    devops_wi_url   VARCHAR(500),
    wi_type         VARCHAR(20) NOT NULL,  -- Feature, User Story, Task
    parent_wi_id    INTEGER,
    sync_status     VARCHAR(10) NOT NULL DEFAULT 'PENDING',
    error           TEXT,
    synced_at       TIMESTAMPTZ,
    created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    CONSTRAINT uq_sync_spec UNIQUE (spec_id)
);

CREATE INDEX idx_sync_status ON work_item_syncs(sync_status);
```

### users

```sql
CREATE TABLE users (
    id            UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    external_id   VARCHAR(200) NOT NULL UNIQUE,  -- ID del provider OAuth
    email         VARCHAR(300),  -- PII: masking en logs (ADR-006)
    name          VARCHAR(200),  -- PII: masking en logs (ADR-006)
    role          VARCHAR(15) NOT NULL DEFAULT 'SENIOR_DEV',
    provider      VARCHAR(20) NOT NULL,  -- github, azure_ad
    created_at    TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    last_login_at TIMESTAMPTZ
);
```

### audit_logs

```sql
CREATE TABLE audit_logs (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    project_id      UUID REFERENCES projects(id),
    entity_type     VARCHAR(50) NOT NULL,
    entity_id       UUID NOT NULL,
    action          VARCHAR(50) NOT NULL,
    actor_id        UUID NOT NULL REFERENCES users(id),
    details         JSONB,          -- NO incluir PII ni PATs
    correlation_id  UUID NOT NULL,  -- ADR-003: trazabilidad
    created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_audit_entity ON audit_logs(entity_type, entity_id);
CREATE INDEX idx_audit_project ON audit_logs(project_id);
CREATE INDEX idx_audit_correlation ON audit_logs(correlation_id);
CREATE INDEX idx_audit_created ON audit_logs(created_at);
```

---

## Clasificacion de datos (ADR-006)

| Tabla.Campo | Clasificacion | Tratamiento |
|-------------|--------------|-------------|
| users.email | **Sensible (PII)** | Masking en logs: `j***@example.com`. No incluir en errores API |
| users.name | **Sensible (PII)** | Masking en logs: `Ju*** Co***`. No incluir en errores API |
| projects.devops_pat_enc | **Sensible** | Cifrado en reposo (BYTEA). Nunca en logs ni AuditLog |
| specifications.content | **Confidencial** | Propiedad intelectual. Cifrado en reposo recomendado |
| adrs.content | **Confidencial** | Sanitizar antes de enviar a LLM |
| prompts.content | **Confidencial** | Contiene spec + reglas |
| work_item_syncs.devops_wi_url | **Interno** | URL interna, no exponer externamente |
| audit_logs.details | **Interno** | NO incluir PII, PATs ni contenido completo de specs |

---

## Indices de performance

```sql
-- Queries frecuentes del dashboard
CREATE INDEX idx_specs_project_status ON specifications(project_id, status);
CREATE INDEX idx_specs_project_level ON specifications(project_id, level);
CREATE INDEX idx_specs_created_at ON specifications(created_at);

-- Query de trazabilidad (arbol L1→L2→L3)
CREATE INDEX idx_specs_parent_level ON specifications(parent_spec_id, level);

-- Metricas de skills
CREATE INDEX idx_prompts_skill_generated ON prompts(skill_id, generated_at);
```
