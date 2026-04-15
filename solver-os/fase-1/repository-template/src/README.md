# /src — Codigo de Producto

Codigo fuente del proyecto, generado desde las especificaciones aprobadas.

## Estructura (varia segun stack)

### Backend (.NET 10 — Hexagonal)
```
src/
├── Domain/          -> Entidades, Value Objects, Puertos (interfaces)
├── Application/     -> Servicios de aplicacion, DTOs, casos de uso
├── Infrastructure/  -> Adaptadores (EF Core, Redis, Service Bus, LLM)
└── Api/             -> Endpoints Minimal API, middleware, auth
```

### Frontend (Angular 21)
```
src/
├── app/
│   ├── core/        -> Interceptors, guards, servicios globales, WebSocket
│   ├── shared/      -> Componentes compartidos, layout, pipes, directivas
│   ├── features/    -> Modulos lazy-loaded por feature
│   └── auth/        -> OAuth, JWT, route guards por rol
├── assets/          -> Imagenes, iconos, fuentes
└── environments/    -> Configuracion por ambiente (dev, staging, prod)
```

## Reglas de generacion

- Todo codigo se genera desde una spec L2 **APPROVED** (Gate 1 pasado)
- El prompt invoca la spec — NO contiene reglas de negocio
- PRs < 400 lineas — dividir si es mayor
- Tests incluidos basados en escenarios GWT de la spec
- Sin PII real en codigo, tests o fixtures
- Masking aplicado en logs para campos Sensible
- Backend y frontend se generan en **paralelo** con prompts pareados

> El codigo se genera usando los prompts (PROMPT-XX para BE, PROMPT-XX-FE para FE).
