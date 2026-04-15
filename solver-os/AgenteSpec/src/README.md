# /src — Codigo del Spec Agent Portal

Este directorio contendra el codigo fuente generado desde las especificaciones aprobadas.

## Estructura esperada

### Backend (.NET 10 — Hexagonal)

```
src/backend/
├── Domain/
│   ├── Entities/          -> Specification, ReviewCycle, Project, Skill, Prompt
│   ├── ValueObjects/      -> SpecVersion, SpecStatus, ReviewDecision, SpecLevel
│   ├── Events/            -> SpecApproved, SpecStatusChanged, PromptGenerated
│   └── Ports/             -> ISpecRepository, ILLMProvider, IDLPFilter, etc.
├── Application/
│   ├── Services/          -> GovernanceService, SpecManagementService, ReviewService, etc.
│   ├── DTOs/              -> Request/Response objects
│   └── Validators/        -> Validacion de entrada
├── Infrastructure/
│   ├── Persistence/       -> EF Core configs, migrations, PostgreSQL
│   ├── LLM/               -> SemanticKernel adapter para Claude API
│   ├── DLP/               -> DLP Filter (pre/post-prompt)
│   ├── Cache/             -> Redis adapter
│   ├── Messaging/         -> Azure Service Bus adapter
│   └── DevOps/            -> Azure DevOps REST API adapter
└── Api/
    ├── Endpoints/         -> Minimal API endpoints
    ├── Middleware/         -> Auth, CORS, correlationId, error handling
    └── Program.cs         -> Configuracion y DI
```

### Frontend (Angular 21)

```
src/frontend/
├── app/
│   ├── core/              -> Interceptors (auth, correlationId, error), WebSocket
│   ├── shared/            -> Layout, sidebar, header, pipes, directivas
│   ├── features/
│   │   ├── projects/      -> PROMPT-01-FE
│   │   ├── specs/         -> PROMPT-02-FE
│   │   ├── reviews/       -> PROMPT-03-FE
│   │   ├── prompts/       -> PROMPT-04-FE
│   │   ├── metrics/       -> PROMPT-05-FE
│   │   └── skills/        -> PROMPT-06-FE
│   └── auth/              -> OAuth, JWT, guards por rol
├── assets/
└── environments/          -> dev, staging, prod
```

## Generacion

El codigo se genera usando los prompts en `prompts/`:
- Backend: PROMPT-01 a PROMPT-07
- Frontend: PROMPT-00-FE a PROMPT-06-FE
- Se ejecutan en **paralelo** (BE + FE por caso de uso)

> Referencia: CLAUDE-BACKEND.md y CLAUDE-FRONTEND.md para contexto IA por stack.
