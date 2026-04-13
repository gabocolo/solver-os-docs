# 05 — Wireframes UI

| Campo | Valor |
|-------|-------|
| **Proyecto** | Spec Agent Portal |
| **Version** | 1.0.0 |
| **Fecha** | 2026-04-09 |

---

## Navegacion principal

```
┌──────────────────────────────────────────────────────────────────────┐
│  [Logo] Spec Agent Portal          [Proyecto ▼]    [User ▼] [Logout]│
├──────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Sidebar:                          Content area:                     │
│  ┌──────────────┐                  ┌──────────────────────────────┐  │
│  │ > Proyectos  │                  │                              │  │
│  │ > Specs      │                  │  (Cambia segun la seccion    │  │
│  │ > Revision   │                  │   seleccionada)              │  │
│  │ > Prompts    │                  │                              │  │
│  │ > Metricas   │                  │                              │  │
│  │ > Skills     │                  │                              │  │
│  │ > DevOps     │                  │                              │  │
│  │              │                  │                              │  │
│  │ ───────────  │                  │                              │  │
│  │ > Gobernanza │                  │                              │  │
│  │   ADRs       │                  │                              │  │
│  │   Gates      │                  │                              │  │
│  └──────────────┘                  └──────────────────────────────┘  │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
```

---

## Pantalla 1: Dashboard de proyecto

```
┌──────────────────────────────────────────────────────────────────────┐
│  Proyecto: Account Planning                                [Editar] │
├──────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐ ┌──────────────┐  │
│  │   12        │ │    4        │ │    3        │ │   85%        │  │
│  │  Total Specs│ │  Approved   │ │  In Review  │ │ Data Classif │  │
│  │             │ │   ✓        │ │   ◎        │ │  Coverage    │  │
│  └─────────────┘ └─────────────┘ └─────────────┘ └──────────────┘  │
│                                                                      │
│  Arbol de trazabilidad                                               │
│  ─────────────────────                                               │
│  ▼ [L1] Booking v1.0.0 (APPROVED)                                   │
│    ├── [L2] CreateBooking v1.0.0 (APPROVED) — 3 prompts             │
│    │   └── [L3] AddCoupon v1.1.0 (DRAFT)                            │
│    ├── [L2] CancelBooking v1.0.0 (IN_REVIEW)                        │
│    └── [L2] GetAvailability v1.0.0 (DRAFT)                          │
│  ▼ [L1] Payments v1.0.0 (DRAFT)                                     │
│                                                                      │
│  Actividad reciente                                                  │
│  ─────────────────                                                   │
│  • CreateBooking aprobada por @architect — hace 2h                   │
│  • CancelBooking enviada a revision — hace 4h                        │
│  • ADR-003 registrado por @architect — hace 1d                       │
│                                                                      │
│  Quick Actions                                                       │
│  ─────────────                                                       │
│  [+ Nueva Spec L1]  [+ Nuevo ADR]  [Ver Metricas]                   │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
```

---

## Pantalla 2: Editor de especificacion (con Monaco)

```
┌──────────────────────────────────────────────────────────────────────┐
│  Spec: CreateBooking v1.0.0          [DRAFT]   [Guardar] [A Review] │
├──────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Metadata                                                            │
│  ┌────────────────────────────────────────────────────────────────┐  │
│  │ Nivel: L2    Parent: [Booking v1.0.0 ▼]    Caso: [CU-01 ▼]   │  │
│  │ Version: 1.0.0    Autor: dev-senior-1    Modelo: Sonnet       │  │
│  └────────────────────────────────────────────────────────────────┘  │
│                                                                      │
│  ┌──── Secciones ────┐  ┌──── Editor (Monaco) ─────────────────┐   │
│  │                    │  │                                       │   │
│  │ > Descripcion      │  │  ## Contrato tecnico                 │   │
│  │ > Contrato tecnico │  │                                       │   │
│  │ > Dependencias     │  │  ### Input — CreateBooking            │   │
│  │ > Errores          │  │                                       │   │
│  │ > Requisitos       │  │  | Campo    | Tipo   | Requerido |   │   │
│  │ > Proteccion datos │  │  |----------|--------|-----------|   │   │
│  │ > Tests GWT        │  │  | userId   | UUID   | Si        |   │   │
│  │ > Referencias      │  │  | roomId   | UUID   | Si        |   │   │
│  │                    │  │  | dates    | Range  | Si        |   │   │
│  │ ────────────────── │  │  | coupon?  | string | No        |   │   │
│  │ [Regenerar seccion]│  │                                       │   │
│  │ con IA             │  │  ### Output — BookingConfirmation     │   │
│  │                    │  │  ...                                   │   │
│  └────────────────────┘  └───────────────────────────────────────┘   │
│                                                                      │
│  ┌──── Preview (renderizado) ──────────────────────────────────┐    │
│  │  Vista previa del markdown renderizado                       │    │
│  └──────────────────────────────────────────────────────────────┘    │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
```

---

## Pantalla 3: Generacion con IA

```
┌──────────────────────────────────────────────────────────────────────┐
│  Generar nueva especificacion con IA                                 │
├──────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Paso 1: Seleccionar nivel                                           │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐                           │
│  │  [L1]    │  │  [L2]    │  │  [L3]    │                           │
│  │ Dominio  │  │ Sistema  │  │ Cambio   │                           │
│  └──────────┘  └──────────┘  └──────────┘                           │
│                                                                      │
│  Paso 2: Contexto                                                    │
│  ┌────────────────────────────────────────────────────────────────┐  │
│  │ Parent: [Booking v1.0.0 (APPROVED) ▼]                         │  │
│  │ Caso de uso: [CU-01: CreateBooking ▼]                         │  │
│  └────────────────────────────────────────────────────────────────┘  │
│                                                                      │
│  Paso 3: Descripcion en lenguaje natural                             │
│  ┌────────────────────────────────────────────────────────────────┐  │
│  │ Sistema de reservas de habitaciones con validacion de          │  │
│  │ disponibilidad y precios. Debe soportar cupones opcionales    │  │
│  │ y notificar al usuario via email...                           │  │
│  │                                                                │  │
│  │                                                     500/2000  │  │
│  └────────────────────────────────────────────────────────────────┘  │
│                                                                      │
│  Contexto adicional (opcional):                                      │
│  ┌────────────────────────────────────────────────────────────────┐  │
│  │ Stack: .NET 10, PostgreSQL. Integracion con payment gateway.  │  │
│  └────────────────────────────────────────────────────────────────┘  │
│                                                                      │
│  Modelo: Opus (L1) | Sonnet (L2/L3)     Tokens est: ~4,000          │
│                                                                      │
│  [Generar con IA]                                                    │
│                                                                      │
│  ┌──── Progreso ──────────────────────────────────────────────┐     │
│  │  ◎ Sanitizando input (DLP pre-prompt)...                    │     │
│  │  ◎ Generando borrador con Claude Opus...                    │     │
│  │  ◎ Validando respuesta (DLP post-response)...              │     │
│  │  ✓ Borrador generado (87s, 4,231 tokens)                   │     │
│  │                                                              │     │
│  │  [Ver borrador en editor]                                    │     │
│  └──────────────────────────────────────────────────────────────┘    │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
```

---

## Pantalla 4: Flujo de revision

```
┌──────────────────────────────────────────────────────────────────────┐
│  Revision: CreateBooking v1.0.0          Ciclo #2    [IN_REVIEW]     │
├──────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Validacion automatica (SK-004)                                      │
│  ┌────────────────────────────────────────────────────────────────┐  │
│  │  ✓ Campos obligatorios completos                               │  │
│  │  ✓ Version semver valida (1.0.0)                               │  │
│  │  ✓ Clasificacion de datos presente                             │  │
│  │  ✓ Parent L1 aprobado                                          │  │
│  │  ⚠ ADR-006 no referenciado (WARNING — spec tiene campos PII)  │  │
│  │  ✓ Escenarios GWT: 8 definidos                                 │  │
│  │                                                                 │  │
│  │  Resultado: PASSED (1 warning)                                  │  │
│  └────────────────────────────────────────────────────────────────┘  │
│                                                                      │
│  Contenido de la spec (solo lectura)                                 │
│  ┌────────────────────────────────────────────────────────────────┐  │
│  │  [Vista completa de la spec en markdown renderizado]           │  │
│  │  ...                                                           │  │
│  └────────────────────────────────────────────────────────────────┘  │
│                                                                      │
│  Comentarios                                                         │
│  ┌────────────────────────────────────────────────────────────────┐  │
│  │  Seccion: [Contrato tecnico ▼]                                 │  │
│  │  ┌──────────────────────────────────────────────────────────┐  │  │
│  │  │  Falta definir el error de timeout para AvailabilityService│ │  │
│  │  └──────────────────────────────────────────────────────────┘  │  │
│  │  [Agregar comentario]                                          │  │
│  │                                                                 │  │
│  │  Comentarios anteriores:                                        │  │
│  │  • @architect en "errores": "Agregar AVAILABILITY_TIMEOUT" (Ciclo #1) │
│  └────────────────────────────────────────────────────────────────┘  │
│                                                                      │
│  Decision                                                            │
│  ┌────────────────────────────────────────────────────────────────┐  │
│  │  Resumen: [                                                    ]  │
│  │                                                                 │  │
│  │  [Aprobar ✓]              [Solicitar cambios ↩]                │  │
│  └────────────────────────────────────────────────────────────────┘  │
│                                                                      │
│  Historial de ciclos                                                 │
│  ┌────────────────────────────────────────────────────────────────┐  │
│  │  Ciclo #1: CHANGES_REQUESTED por @architect — 2h              │  │
│  │  Ciclo #2: IN_REVIEW — actual                                  │  │
│  └────────────────────────────────────────────────────────────────┘  │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
```

---

## Pantalla 5: Prompt Builder

```
┌──────────────────────────────────────────────────────────────────────┐
│  Generador de Prompts Estructurados                                  │
├──────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Seleccion                                                           │
│  ┌────────────────────────────────────────────────────────────────┐  │
│  │  Spec: [CreateBooking v1.0.0 (APPROVED) ▼]                    │  │
│  │  Skill: [SK-001 Generacion de caso de uso v1.0.0 ▼]          │  │
│  │  Instrucciones adicionales:                                    │  │
│  │  [                                                            ]│  │
│  │                                                                 │  │
│  │  [Generar prompt]                                               │  │
│  └────────────────────────────────────────────────────────────────┘  │
│                                                                      │
│  Preview del prompt                                                  │
│  ┌────────────────────────────────────────────────────────────────┐  │
│  │  ## Rol                                                        │  │
│  │  Actua como un desarrollador senior.                           │  │
│  │                                                                 │  │
│  │  ## Invocacion de especificacion                               │  │
│  │  Aplica la especificacion SPEC-L2-create-booking v1.0.0.      │  │
│  │                                                                 │  │
│  │  ## Tarea                                                       │  │
│  │  Genera la implementacion del caso de uso CreateBooking        │  │
│  │  segun la especificacion, respetando arquitectura hexagonal.   │  │
│  │                                                                 │  │
│  │  ## Entrada                                                     │  │
│  │  BookingRequest: userId (UUID), roomId (UUID), dates (Range),  │  │
│  │  couponCode (string, opcional)                                  │  │
│  │                                                                 │  │
│  │  ## Salida esperada                                             │  │
│  │  BookingConfirmation segun spec                                 │  │
│  │                                                                 │  │
│  │  ## Reglas                                                      │  │
│  │  - Solo usar dependencias: AvailabilityService, CouponProvider │  │
│  │  - Errores: ROOM_NOT_AVAILABLE, INVALID_DATES, INVALID_COUPON │  │
│  │  - No inventar servicios ni dependencias fuera de la spec      │  │
│  │                                                                 │  │
│  │  ## Gobernanza                                                  │  │
│  │  - ADR-001: Ports & Adapters                                    │  │
│  │  - ADR-002: Idempotencia                                        │  │
│  │  - ADR-006: Clasificacion de datos y DLP                        │  │
│  │                                                                 │  │
│  │  ## Restricciones DLP                                           │  │
│  │  - No incluir datos reales (PII) en codigo, tests o fixtures   │  │
│  │  - Aplicar masking en logs para userId (Sensible)               │  │
│  │  - Usar datos sinteticos siempre                                │  │
│  │                                                                 │  │
│  └────────────────────────────────────────────────────────────────┘  │
│                                                                      │
│  [Editar] [Copiar al clipboard]                                      │
│                                                                      │
│  Trazabilidad: Spec v1.0.0 + SK-001 v1.0.0 — generado por @dev     │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
```

---

## Pantalla 6: Dashboard de metricas

```
┌──────────────────────────────────────────────────────────────────────┐
│  Metricas — Account Planning          Periodo: [Sprint actual ▼]     │
├──────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌────────┐ ┌────────┐ ┌──────────┐ ┌──────────┐ ┌──────────────┐  │
│  │  12    │ │  33%   │ │  1.2d    │ │  85%     │ │  100%        │  │
│  │ Total  │ │ Apprvd │ │ Avg Time │ │ Data Cls │ │ Data Protect │  │
│  │ Specs  │ │ Rate   │ │ Approval │ │ Coverage │ │ Coverage     │  │
│  └────────┘ └────────┘ └──────────┘ └──────────┘ └──────────────┘  │
│                                                                      │
│  Specs por estado                     Uso de skills                  │
│  ┌──────────────────────┐             ┌──────────────────────────┐  │
│  │ ████████░░░░ DRAFT 5 │             │ SK-001  ████████ 12      │  │
│  │ ████░░░░░░░ REVIEW 3 │             │ SK-002  ██████░░  8      │  │
│  │ ██████░░░░░ APPRVD 4 │             │ SK-003  ████░░░░  5      │  │
│  └──────────────────────┘             │ SK-006  ██░░░░░░  3      │  │
│                                        └──────────────────────────┘  │
│  Alertas de skills                                                   │
│  ┌────────────────────────────────────────────────────────────────┐  │
│  │  ⚠ SK-003 tiene 25% retrabajo sostenido por 2 sprints.       │  │
│  │    Considere redisenar o eliminar.                             │  │
│  └────────────────────────────────────────────────────────────────┘  │
│                                                                      │
│  Ciclos de revision promedio: 1.8 ciclos antes de aprobacion         │
│  Tokens consumidos este sprint: 45,230 (Opus: 28,100 | Sonnet: 17,130) │
│                                                                      │
│  [Exportar CSV]  [Exportar JSON]  [Ver trazabilidad completa]        │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
```

---

## Consideraciones UX

| Aspecto | Implementacion |
|---------|---------------|
| Responsive | Angular Material breakpoints + Tailwind utilities |
| Loading states | Skeleton loaders para dashboard, spinner para generacion IA |
| Errores | Snackbar para errores transitorios, dialogo para errores criticos |
| Navegacion | Breadcrumbs + sidebar colapsable |
| Accesibilidad | ARIA labels, keyboard navigation, contraste AA |
| Feedback IA | Progress bar con pasos (sanitizar → generar → validar) |
| Monaco Editor | Syntax highlighting markdown, auto-complete para secciones de spec |
