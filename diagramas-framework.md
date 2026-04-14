# Diagramas del Framework AI-Driven Engineering - Tech&Solve

---

## 1. Visión General: Las 5 Fases del Framework

```mermaid
flowchart LR
    F1["**Fase 1**<br>Gobernanza"]
    F2["**Fase 2**<br>Especificación<br>Dirigida"]
    F3["**Fase 3**<br>Generación<br>Controlada"]
    F4["**Fase 4**<br>Validación<br>Automatizada"]
    F5["**Fase 5**<br>Mejora<br>Continua"]

    F1 --> F2 --> F3 --> F4 --> F5
    F5 -.->|Retroalimentación| F1

    style F1 fill:#4a90d9,color:#fff
    style F2 fill:#7b68ee,color:#fff
    style F3 fill:#e67e22,color:#fff
    style F4 fill:#27ae60,color:#fff
    style F5 fill:#8e44ad,color:#fff
```

---

## 2. Entradas y Salidas de Cada Fase

```mermaid
flowchart TB
    subgraph FASE1["FASE 1: Gobernanza"]
        direction TB
        I1["**Entradas:**<br>- Necesidades del negocio<br>- Restricciones regulatorias<br>- Lecciones aprendidas"]
        O1["**Salidas:**<br>- Principios de arquitectura<br>- ADRs aprobados<br>- Estándares de código y seguridad<br>- Clasificación de datos (DLP)"]
        I1 ~~~ O1
    end

    subgraph FASE2["FASE 2: Especificación Dirigida"]
        direction TB
        I2["**Entradas:**<br>- Principios y ADRs (Fase 1)<br>- Historias de usuario<br>- Requerimientos de negocio"]
        O2["**Salidas:**<br>- Spec L1 (dominio + clasificación datos)<br>- Spec L2 (sistema + contratos API)<br>- Spec L3 (cambio incremental)<br>- Todas versionadas con semver"]
        I2 ~~~ O2
    end

    subgraph FASE3["FASE 3: Generación Controlada"]
        direction TB
        I3["**Entradas:**<br>- Spec aprobada y versionada<br>- ADRs relevantes<br>- Contexto mínimo necesario<br>- Skill aplicable"]
        O3["**Salidas:**<br>- Código en la capa correcta<br>- Tests basados en spec<br>- Diff pequeño y revisable<br>- Masking de PII aplicado"]
        I3 ~~~ O3
    end

    subgraph FASE4["FASE 4: Validación Automatizada"]
        direction TB
        I4["**Entradas:**<br>- Código generado (Fase 3)<br>- Tests GWT definidos<br>- Pipeline CI/CD configurado"]
        O4["**Salidas:**<br>- Resultados de tests unitarios<br>- Reporte SAST/SCA<br>- Reporte DLP Scan<br>- PR validado para revisión"]
        I4 ~~~ O4
    end

    subgraph FASE5["FASE 5: Mejora Continua"]
        direction TB
        I5["**Entradas:**<br>- Métricas de calidad IA<br>- Métricas DORA<br>- Métricas DLP<br>- Métricas de skills<br>- Hallazgos post-deploy"]
        O5["**Salidas:**<br>- Specs refinadas<br>- Skills evolucionados<br>- ADRs actualizados<br>- Quality gates ajustados"]
        I5 ~~~ O5
    end

    FASE1 --> FASE2 --> FASE3 --> FASE4 --> FASE5
    FASE5 -.->|Retroalimenta| FASE1

    style FASE1 fill:#e8f4fd,stroke:#4a90d9
    style FASE2 fill:#ede7f6,stroke:#7b68ee
    style FASE3 fill:#fff3e0,stroke:#e67e22
    style FASE4 fill:#e8f5e9,stroke:#27ae60
    style FASE5 fill:#f3e5f5,stroke:#8e44ad
```

---

## 3. Roles y Responsabilidades por Fase

```mermaid
flowchart TB
    subgraph ROLES["Roles del Framework"]
        ARQ["🏛️ Arquitecto /<br>Líder Técnico"]
        DEV["💻 Desarrollador<br>Senior"]
        QA["🔍 QA / Revisor<br>/ Senior"]
    end

    subgraph F1["Fase 1: Gobernanza"]
        F1A["Define y aprueba<br>principios de arquitectura"]
        F1B["Crea y versiona ADRs"]
        F1C["Define estándares<br>de seguridad y DLP"]
    end

    subgraph F2["Fase 2: Especificación"]
        F2A["Define y aprueba<br>Specs L1/L2/L3"]
        F2B["Revisa clasificación<br>de datos"]
        F2C["Aprueba PR de specs"]
    end

    subgraph F3["Fase 3: Generación"]
        F3A["Diseña prompt<br>estructurado"]
        F3B["Ejecuta generación<br>con IA"]
        F3C["Propone PR<br>quirúrgico"]
    end

    subgraph F4["Fase 4: Validación"]
        F4A["Valida quality gates<br>(tests, SAST, SCA, DLP)"]
        F4B["Revisión humana<br>obligatoria"]
        F4C["Aprueba merge"]
    end

    ARQ -->|Lidera| F1
    ARQ -->|Lidera| F2
    DEV -->|Ejecuta| F3
    QA -->|Valida| F4
    ARQ -.->|Supervisa| F3
    DEV -.->|Apoya| F4

    style ARQ fill:#4a90d9,color:#fff
    style DEV fill:#e67e22,color:#fff
    style QA fill:#27ae60,color:#fff
```

---

## 4. Niveles de Especificación y su Relación

```mermaid
flowchart TB
    subgraph L1["Spec Nivel 1 — Dominio (QUÉ desde el negocio)"]
        L1C["Objetivo de negocio<br>Entidades y relaciones<br>Reglas de dominio<br>Clasificación de datos (PII)"]
    end

    subgraph L2A["Spec L2-A — Sistema A"]
        L2AC["Contrato API (OpenAPI)<br>Entradas/salidas tipadas<br>Dependencias permitidas<br>Protección de datos PII<br>Escenarios de prueba"]
    end

    subgraph L2B["Spec L2-B — Sistema B"]
        L2BC["Contrato API<br>Entradas/salidas<br>Dependencias<br>Protección datos<br>Escenarios"]
    end

    subgraph L3_1["Spec L3 v1.1.0"]
        L3_1C["Cambio: agregar cupón<br>Impacto en contrato<br>Impacto en comportamiento"]
    end

    subgraph L3_2["Spec L3 v1.2.0"]
        L3_2C["Cambio: límite de reservas<br>Nueva validación<br>Nuevo error"]
    end

    subgraph L3_3["Spec L3 v1.0.1"]
        L3_3C["Fix: validación fechas<br>Corrección en contrato"]
    end

    L1 -->|"1 a muchos"| L2A
    L1 -->|"1 a muchos"| L2B
    L2A -->|"1 a muchos"| L3_1
    L2A -->|"1 a muchos"| L3_2
    L2B -->|"1 a muchos"| L3_3

    TRACE["**Trazabilidad:**<br>L1 → L2 → L3 → código → commit → release"]

    style L1 fill:#4a90d9,color:#fff
    style L2A fill:#7b68ee,color:#fff
    style L2B fill:#7b68ee,color:#fff
    style L3_1 fill:#e67e22,color:#fff
    style L3_2 fill:#e67e22,color:#fff
    style L3_3 fill:#e67e22,color:#fff
    style TRACE fill:#f9e79f,stroke:#f1c40f
```

---

## 5. Quality Gates — Flujo de Control

```mermaid
flowchart LR
    START(("Inicio"))

    G1{"**Gate 1**<br>Spec versionada?<br>Aprobada?<br>Clasificación datos?"}
    G2{"**Gate 2**<br>Tests pasan?<br>SAST ok?<br>SCA ok?<br>DLP Scan ok?"}
    G3{"**Gate 3**<br>Revisión senior?<br>Protección datos<br>verificada?"}

    GEN["Generación<br>de Código<br>(Fase 3)"]
    REV["Revisión<br>Humana"]
    MERGE(("Merge<br>✓"))
    BLOCK1["⛔ BLOQUEADO<br>No se genera código"]
    BLOCK2["⛔ BLOQUEADO<br>No pasa a revisión"]
    BLOCK3["⛔ BLOQUEADO<br>No se hace merge"]

    START --> G1
    G1 -->|"✅ Aprobada"| GEN
    G1 -->|"❌ No aprobada"| BLOCK1
    GEN --> G2
    G2 -->|"✅ Todo pasa"| REV
    G2 -->|"❌ Falla algo"| BLOCK2
    REV --> G3
    G3 -->|"✅ Aprobado"| MERGE
    G3 -->|"❌ Rechazado"| BLOCK3

    BLOCK1 -.->|"Corregir spec"| G1
    BLOCK2 -.->|"Corregir código"| GEN
    BLOCK3 -.->|"Corregir PR"| GEN

    style G1 fill:#e74c3c,color:#fff
    style G2 fill:#e74c3c,color:#fff
    style G3 fill:#e74c3c,color:#fff
    style MERGE fill:#27ae60,color:#fff
    style BLOCK1 fill:#95a5a6,color:#fff
    style BLOCK2 fill:#95a5a6,color:#fff
    style BLOCK3 fill:#95a5a6,color:#fff
```

---

## 6. Cadena de Trazabilidad Completa

```mermaid
flowchart LR
    L1["Spec L1<br>(Dominio)"]
    L2["Spec L2<br>(Sistema)"]
    L3["Spec L3<br>(Cambio)"]
    CODE["Código<br>Generado"]
    TEST["Tests<br>GWT"]
    PR["Pull<br>Request"]
    COMMIT["Commit<br>Trazable"]
    REL["Release<br>Versionada"]

    L1 -->|define| L2
    L2 -->|deriva| L3
    L3 -->|guía| CODE
    L3 -->|define| TEST
    CODE --> PR
    TEST --> PR
    PR -->|aprobado| COMMIT
    COMMIT --> REL

    style L1 fill:#4a90d9,color:#fff
    style L2 fill:#7b68ee,color:#fff
    style L3 fill:#9b59b6,color:#fff
    style CODE fill:#e67e22,color:#fff
    style TEST fill:#27ae60,color:#fff
    style PR fill:#f1c40f,color:#000
    style COMMIT fill:#e74c3c,color:#fff
    style REL fill:#1abc9c,color:#fff
```

---

## 7. Catálogo de Skills y su Ubicación en Fases

```mermaid
flowchart TB
    subgraph F2["Fase 2: Especificación"]
        SK003["SK-003<br>Generación Contrato<br>API (OpenAPI)"]
        SK004["SK-004<br>Revisión Automatizada<br>de Spec"]
    end

    subgraph F3["Fase 3: Generación"]
        SK001["SK-001<br>Generación Código<br>Caso de Uso"]
        SK006["SK-006<br>Data Masking<br>(PII)"]
        SK007["SK-007<br>DLP Guard<br>LLM"]
    end

    subgraph F4["Fase 4: Validación"]
        SK002["SK-002<br>Generación Tests<br>GWT"]
        SK005["SK-005<br>Pipeline<br>Validación PR"]
    end

    SK003 -.->|"Contrato alimenta"| SK001
    SK002 -.->|"Tests antes de código"| SK001
    SK001 -.->|"Código generado"| SK005
    SK006 -.->|"Masking integrado"| SK001
    SK007 -.->|"Filtro DLP"| SK001

    style F2 fill:#ede7f6,stroke:#7b68ee,color:#000
    style F3 fill:#fff3e0,stroke:#e67e22,color:#000
    style F4 fill:#e8f5e9,stroke:#27ae60,color:#000
    style SK001 fill:#fde8cd,stroke:#e67e22,color:#000
    style SK002 fill:#d4edda,stroke:#27ae60,color:#000
    style SK003 fill:#d9d2f0,stroke:#7b68ee,color:#000
    style SK004 fill:#d9d2f0,stroke:#7b68ee,color:#000
    style SK005 fill:#d4edda,stroke:#27ae60,color:#000
    style SK006 fill:#fde8cd,stroke:#e67e22,color:#000
    style SK007 fill:#fde8cd,stroke:#e67e22,color:#000
```

---

## 8. Niveles de Madurez de Adopción

```mermaid
flowchart BT
    N1["**Nivel 1** — Sin estructura<br>Prompts sin formato, cada dev a su manera"]
    N2["**Nivel 2** — Organización inicial<br>Primeras plantillas, indicios de gobernanza"]
    N3["**Nivel 3** — Especificaciones<br>Specs versionadas por historia de usuario"]
    N4["**Nivel 4** — Gobernanza Operacional ⭐ OBJETIVO<br>Specs multinivel, quality gates, trazabilidad,<br>catálogo de skills, métricas convergentes"]
    N5["**Nivel 5** — IA con mayor control<br>Autonomía controlada (experimental)"]

    N1 --> N2 --> N3 --> N4 --> N5

    style N1 fill:#e74c3c,color:#fff
    style N2 fill:#e67e22,color:#fff
    style N3 fill:#f1c40f,color:#000
    style N4 fill:#27ae60,color:#fff
    style N5 fill:#3498db,color:#fff
```

---

## 9. Flujo DLP (Protección de Datos) End-to-End

```mermaid
flowchart TB
    subgraph DESIGN["Diseño (Privacy by Design)"]
        CL["Clasificación de datos<br>en Spec L1"]
        TR["Tratamiento PII<br>en Spec L2"]
    end

    subgraph DEV["Desarrollo"]
        MASK["SK-006: Masking<br>en código generado"]
        GUARD["SK-007: DLP Guard<br>pre/post LLM"]
        SYNTH["Datos sintéticos<br>en tests y fixtures"]
    end

    subgraph VALID["Validación"]
        DLP_SCAN["DLP Scan en CI/CD<br>(bloquea merge si PII)"]
        SAST["SAST / SCA"]
    end

    subgraph RUNTIME["Runtime"]
        LOG_FILTER["Filtro PII<br>en logs"]
        API_MASK["Masking en<br>respuestas API"]
        MONITOR["Monitoreo<br>de fugas"]
    end

    CL --> TR --> MASK
    TR --> GUARD
    TR --> SYNTH
    MASK --> DLP_SCAN
    SYNTH --> DLP_SCAN
    DLP_SCAN --> LOG_FILTER
    DLP_SCAN --> API_MASK
    API_MASK --> MONITOR

    style DESIGN fill:#e8f4fd,stroke:#4a90d9,color:#000
    style DEV fill:#fff3e0,stroke:#e67e22,color:#000
    style VALID fill:#e8f5e9,stroke:#27ae60,color:#000
    style RUNTIME fill:#f3e5f5,stroke:#8e44ad,color:#000
    style CL fill:#d0e8f7,stroke:#4a90d9,color:#000
    style TR fill:#d0e8f7,stroke:#4a90d9,color:#000
    style MASK fill:#fde8cd,stroke:#e67e22,color:#000
    style GUARD fill:#fde8cd,stroke:#e67e22,color:#000
    style SYNTH fill:#fde8cd,stroke:#e67e22,color:#000
    style DLP_SCAN fill:#d4edda,stroke:#27ae60,color:#000
    style SAST fill:#d4edda,stroke:#27ae60,color:#000
    style LOG_FILTER fill:#e8d5f0,stroke:#8e44ad,color:#000
    style API_MASK fill:#e8d5f0,stroke:#8e44ad,color:#000
    style MONITOR fill:#e8d5f0,stroke:#8e44ad,color:#000
```

---

## 10. Modelo de Costos — Modelo por Tipo de Tarea

```mermaid
flowchart LR
    subgraph OPUS["Modelo Grande (Opus) 💰💰💰"]
        T1["Arquitectura y<br>decisiones críticas"]
        T2["Generación de<br>specs y artefactos"]
    end

    subgraph SONNET["Modelo Mediano (Sonnet) 💰💰"]
        T3["Generación de código<br>desde specs"]
    end

    subgraph HAIKU["Modelo Pequeño (Haiku) 💰"]
        T4["Refactors y tareas<br>repetitivas"]
    end

    T1 -.->|"Specs alimentan"| T3
    T2 -.->|"Specs alimentan"| T3
    T3 -.->|"Código a refactorizar"| T4

    style OPUS fill:#e74c3c,color:#fff
    style SONNET fill:#f39c12,color:#fff
    style HAIKU fill:#27ae60,color:#fff
```

---

## 11. Plan de Adopción 30-60-90 Días

```mermaid
%%{init: {'theme': 'base', 'themeVariables': {'fontSize': '14px', 'primaryColor': '#4a90d9', 'primaryTextColor': '#000000', 'primaryBorderColor': '#3a7abd', 'secondaryColor': '#27ae60', 'tertiaryColor': '#7b68ee', 'textColor': '#000000', 'mainBkg': '#ffffff', 'sectionBkgColor': '#e8f4fd', 'sectionBkgColor2': '#f5f0ff', 'altSectionBkgColor': '#e8f5e9', 'taskBkgColor': '#4a90d9', 'taskTextColor': '#ffffff', 'taskBorderColor': '#3a7abd', 'taskTextOutsideColor': '#000000', 'taskTextClickableColor': '#000000', 'activeTaskBkgColor': '#27ae60', 'activeTaskBorderColor': '#1e8449', 'doneTaskBkgColor': '#7b68ee', 'doneTaskBorderColor': '#6a5acd', 'gridColor': '#cccccc', 'todayLineColor': '#e74c3c', 'titleColor': '#000000', 'labelColor': '#000000', 'loopTextColor': '#000000', 'noteBkgColor': '#ffffff', 'noteTextColor': '#000000'}}}%%
gantt
    title Plan de Adopción del Framework
    dateFormat  YYYY-MM-DD
    axisFormat  %d %b
    
    section 30 días — Fundamentos
    Template de spec           :done, a1, 2026-04-14, 14d
    ADRs críticos              :done, a2, 2026-04-14, 16d
    Quality gates              :done, a3, 2026-04-18, 14d
    Repo AI-Ready              :done, a4, 2026-04-22, 12d

    section 60 días — Piloto
    Piloto en dominio          :active, b1, 2026-05-14, 20d
    Trazabilidad completa      :active, b2, 2026-05-18, 16d
    Métricas operativas        :active, b3, 2026-05-22, 14d

    section 90 días — Escalamiento
    2-3 equipos con métricas   :c1, 2026-06-13, 20d
    Catálogo de skills         :c2, 2026-06-17, 16d
    Métricas por sprint        :c3, 2026-06-20, 14d
```

---

## 12. Ejemplo Completo: Sistema de Booking

```mermaid
flowchart TB
    CLIENT["Cliente<br>(Web App)"]

    subgraph API["Booking API"]
        CTRL["Controller"]
        SVC["Service<br>(CreateBooking)"]
    end

    AUTH["AuthService<br>🔐 Autenticación"]
    AVAIL["AvailabilityService<br>📅 Disponibilidad + Precio"]
    COUPON["CouponProvider<br>🎫 Cupón (opcional)"]
    DB["Database<br>💾 Transacción Atómica"]
    BUS["EventBus<br>📨 Publicación"]
    NOTIF["NotificationService<br>📧 Notificación"]

    CLIENT -->|"1. Request"| CTRL
    CTRL --> SVC
    SVC -->|"2. Validar auth"| AUTH
    SVC -->|"3. Validar disponibilidad"| AVAIL
    SVC -->|"4. Aplicar cupón"| COUPON
    SVC -->|"5. Persistir"| DB
    SVC -->|"6. Publicar evento"| BUS
    BUS -->|"eventual"| NOTIF
    SVC -->|"7. Respuesta"| CLIENT

    style CLIENT fill:#3498db,color:#fff
    style API fill:#fff3e0,stroke:#e67e22
    style AUTH fill:#e74c3c,color:#fff
    style AVAIL fill:#27ae60,color:#fff
    style COUPON fill:#f1c40f,color:#000
    style DB fill:#9b59b6,color:#fff
    style BUS fill:#1abc9c,color:#fff
    style NOTIF fill:#95a5a6,color:#fff
```

---

## 13. Anti-patrones vs. Patrones Correctos

```mermaid
flowchart LR
    subgraph BAD["Anti-patrones"]
        direction TB
        AP1["Prompt gigante a<br>modulo completo"]
        AP2["Sin spec ni<br>estructura"]
        AP3["PRs gigantes<br>mas de 400 lineas"]
        AP4["PII real en<br>prompts y tests"]
        AP5["Efecto wow:<br>aceptar sin criterio"]
    end

    subgraph GOOD["Patrones Correctos"]
        direction TB
        PP1["1 tarea tecnica<br>por prompt"]
        PP2["Spec aprobada<br>y versionada"]
        PP3["PRs quirurgicos<br>menos de 400 lineas"]
        PP4["Datos sinteticos<br>siempre"]
        PP5["Revision humana<br>obligatoria"]
    end

    AP1 -..->|"Reemplazar por"| PP1
    AP2 -..->|"Reemplazar por"| PP2
    AP3 -..->|"Reemplazar por"| PP3
    AP4 -..->|"Reemplazar por"| PP4
    AP5 -..->|"Reemplazar por"| PP5

    style BAD fill:#fde8e8,stroke:#e74c3c,color:#000
    style GOOD fill:#e8f5e9,stroke:#27ae60,color:#000
    style AP1 fill:#f8c4c4,stroke:#e74c3c,color:#000
    style AP2 fill:#f8c4c4,stroke:#e74c3c,color:#000
    style AP3 fill:#f8c4c4,stroke:#e74c3c,color:#000
    style AP4 fill:#f8c4c4,stroke:#e74c3c,color:#000
    style AP5 fill:#f8c4c4,stroke:#e74c3c,color:#000
    style PP1 fill:#c4edc9,stroke:#27ae60,color:#000
    style PP2 fill:#c4edc9,stroke:#27ae60,color:#000
    style PP3 fill:#c4edc9,stroke:#27ae60,color:#000
    style PP4 fill:#c4edc9,stroke:#27ae60,color:#000
    style PP5 fill:#c4edc9,stroke:#27ae60,color:#000
```

---

## 14. Alineacion DDD en Especificaciones

```mermaid
flowchart TB
    subgraph L1["Spec L1 - Dominio"]
        direction TB
        BC["Bounded Context<br>(Limite del contexto)"]
        AGG["Agregados<br>(Raiz, Entidades, VOs)"]
        BR["Reglas de negocio<br>(Invariantes del dominio)"]
        EVT1["Eventos de dominio<br>(nivel de negocio)"]
        UL["Lenguaje ubicuo<br>(Glosario + Nombre en codigo)"]
    end

    subgraph L2["Spec L2 - Sistema"]
        direction TB
        AGG_REF["Agregado principal<br>(caso de uso opera sobre)"]
        EVT2["Eventos de dominio<br>(contrato tecnico + payload)"]
        PORTS["Puertos<br>(Anti-Corruption Layer)"]
        DEPS["Dependencias permitidas<br>(solo las autorizadas)"]
    end

    subgraph L3["Spec L3 - Cambio"]
        direction TB
        DELTA["Delta sobre L2<br>(cambio incremental<br>sobre cualquier concepto)"]
    end

    subgraph GOV["Gobernanza"]
        direction TB
        ADR1["ADR-001<br>Ports and Adapters"]
        ADR7["ADR-007<br>Domain Modeling"]
    end

    BC --> AGG_REF
    AGG --> AGG_REF
    EVT1 --> EVT2
    UL -.->|"Nombres obligatorios<br>en codigo"| L2
    BR -.->|"Invariantes validan<br>en caso de uso"| L2
    L2 --> DELTA
    ADR1 -.->|"Regla transversal"| PORTS
    ADR7 -.->|"Regla transversal"| AGG

    style L1 fill:#e8f4fd,stroke:#4a90d9,color:#000
    style L2 fill:#ede7f6,stroke:#7b68ee,color:#000
    style L3 fill:#fff3e0,stroke:#e67e22,color:#000
    style GOV fill:#f5f5f5,stroke:#999,color:#000
    style BC fill:#d0e8f7,stroke:#4a90d9,color:#000
    style AGG fill:#d0e8f7,stroke:#4a90d9,color:#000
    style BR fill:#d0e8f7,stroke:#4a90d9,color:#000
    style EVT1 fill:#d0e8f7,stroke:#4a90d9,color:#000
    style UL fill:#d0e8f7,stroke:#4a90d9,color:#000
    style AGG_REF fill:#d9d2f0,stroke:#7b68ee,color:#000
    style EVT2 fill:#d9d2f0,stroke:#7b68ee,color:#000
    style PORTS fill:#d9d2f0,stroke:#7b68ee,color:#000
    style DEPS fill:#d9d2f0,stroke:#7b68ee,color:#000
    style DELTA fill:#fde8cd,stroke:#e67e22,color:#000
    style ADR1 fill:#e8e8e8,stroke:#999,color:#000
    style ADR7 fill:#e8e8e8,stroke:#999,color:#000
```

---

## Cómo Visualizar Estos Diagramas

Estos diagramas están en formato **Mermaid** y se pueden renderizar en:

1. **GitHub / GitLab** — renderiza Mermaid automáticamente en archivos `.md`
2. **VS Code** — extensión "Markdown Preview Mermaid Support"
3. **Mermaid Live Editor** — [mermaid.live](https://mermaid.live)
4. **Notion** — soporta bloques Mermaid nativamente
5. **Confluence** — con plugin de Mermaid
