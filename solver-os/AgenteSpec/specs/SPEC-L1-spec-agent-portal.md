# Especificacion de Dominio (Nivel 1)

| Campo | Valor |
|-------|-------|
| **Spec ID** | SPEC-L1-spec-agent-portal |
| **Nivel** | L1 — Dominio |
| **Version** | 1.1.0 |
| **Estado** | DRAFT |
| **Creado por** | Equipo de Arquitectura |
| **Fecha de creacion** | 2026-04-09 |
| **Aprobado por** | Pendiente |
| **Fecha de aprobacion** | Pendiente |
| **PR** | Pendiente |

---

## Descripcion general

Portal web para administrar el agente de generacion de especificaciones del framework AI-Driven Engineering. Orquesta el ciclo completo: desde la entrada de negocio en lenguaje natural hasta la especificacion versionada, aprobada y lista para generar codigo. Implementa las 5 fases del framework como un producto operativo con enforcement automatico de gobernanza, quality gates y proteccion de datos.

---

## Dominio

**Nombre:** Spec Agent Portal

## Objetivo de negocio

Convertir el framework AI-Driven Engineering de un conjunto de documentos y convenciones manuales en una **herramienta operativa** que enforce automaticamente las reglas de gobernanza, acelere la creacion de especificaciones con IA, y proporcione trazabilidad y metricas en tiempo real.

### Problema que resuelve

- Las especificaciones se crean manualmente sin enforcement de estructura ni campos obligatorios
- No hay trazabilidad automatica entre gobernanza (ADRs) y especificaciones
- Los quality gates se verifican manualmente y se pueden saltar
- Los prompts se construyen de forma inconsistente por cada desarrollador
- Las metricas del framework no se miden o se miden manualmente
- La clasificacion de datos (ADR-006) se aplica a criterio individual
- No hay flujo formal de revision y aprobacion de specs

### Valor esperado

- **Reduccion de tiempo**: de 2-4 horas a < 30 minutos por spec L1 (generacion asistida por IA)
- **Enforcement**: 100% de specs con estructura completa y clasificacion de datos
- **Trazabilidad**: cadena completa spec → prompt → commit → release
- **Consistencia**: todos los equipos siguen el mismo proceso con la misma herramienta
- **Metricas**: visibilidad en tiempo real del estado del framework por equipo

---

## Contexto

- El framework AI-Driven Engineering v1.1 esta documentado en `claude.md` con 16 secciones, 5 fases, 3 quality gates, 7 skills y 12 principios
- Existe un repositorio AI-Ready con templates de specs (L1/L2/L3), ADRs, skills y prompts en `solver-os/fase-1/`
- El primer producto desarrollado con el framework es Account Planning (`solver-os/account-planning/`)
- Los equipos usan Git (GitHub/Azure DevOps) para versionado y CI/CD
- La IA se invoca hoy de forma manual (Claude Code, Cursor, ChatGPT) — no hay orquestacion
- El framework define 3 niveles de specs (L1 dominio, L2 sistema, L3 cambio) con relacion jerarquica y versionado semver
- ADR-006 establece clasificacion obligatoria de datos y prevencion de fuga (DLP)

---

## Entidades

| Entidad | Descripcion | Atributos clave |
|---------|------------|----------------|
| **Project** | Proyecto o dominio que agrupa specs, ADRs y configuracion de gobernanza | projectId, name, description, gitRepoUrl, devOpsProjectUrl, devOpsOrganization, defaultBranch, createdAt, ownerId |
| **Specification** | Especificacion versionada en cualquiera de los 3 niveles (L1/L2/L3) | specId, projectId, level (L1/L2/L3), parentSpecId, version, status, title, content (structured), dataClassification, createdBy, createdAt, approvedBy, approvedAt |
| **ADR** | Architecture Decision Record asociado a un proyecto | adrId, projectId, number, title, status (PROPOSED/ACCEPTED/DEPRECATED/SUPERSEDED), content, createdBy, createdAt |
| **QualityGateCheck** | Ejecucion de un quality gate sobre una spec o PR | checkId, specId, gateNumber (1/2/3), status (PASSED/FAILED/PENDING), findings[], executedAt, executedBy |
| **ReviewCycle** | Ciclo de revision de una spec con comentarios y decision | reviewId, specId, reviewerId, status (PENDING/APPROVED/CHANGES_REQUESTED), comments[], startedAt, completedAt |
| **Prompt** | Prompt estructurado generado desde una spec aprobada | promptId, specId, specVersion, skillId, role, task, context, constraints, generatedAt, generatedBy |
| **Skill** | Skill registrado en el catalogo con metricas | skillId, projectId, name, type (BUILD/WORKFLOW), version, owner, phase, status (ACTIVE/DEPRECATED), metrics |
| **Metric** | Metrica calculada del framework | metricId, projectId, type, value, period, calculatedAt |
| **User** | Usuario autenticado del portal | userId, email, name, role (ARCHITECT/LEAD/SENIOR_DEV/QA/PO), oauthProvider, lastLoginAt |
| **AuditLog** | Registro de auditoria de acciones criticas | logId, userId, action, entityType, entityId, timestamp, details |
| **WorkItemSync** | Registro de sincronizacion de specs aprobadas con Azure DevOps como work items | syncId, specId, projectId, devOpsWorkItemId, devOpsUrl, syncStatus (PENDING/SYNCED/FAILED), syncedAt, error |

---

## Clasificacion de datos (ADR-006)

| Entidad.Atributo | Clasificacion | Tratamiento |
|------------------|---------------|-------------|
| User.email | Sensible (PII) | Masking en logs, no exponer en APIs publicas |
| User.name | Sensible (PII) | Masking en logs |
| User.oauthProvider | Interno | No exponer externamente |
| Specification.content | Confidencial | Puede contener reglas de negocio sensibles del cliente |
| Specification.dataClassification | Interno | Metadata de gobernanza |
| ADR.content | Confidencial | Decisiones tecnicas internas |
| Prompt.context | Confidencial | Puede contener contexto de negocio — sanitizar antes de enviar a LLM |
| Prompt.constraints | Interno | Reglas del framework |
| AuditLog.details | Interno | No exponer externamente |
| QualityGateCheck.findings | Confidencial | Puede contener hallazgos de seguridad |
| Project.gitRepoUrl | Interno | URL interna del repositorio |
| Project.devOpsProjectUrl | Interno | URL del proyecto en Azure DevOps |
| WorkItemSync.devOpsWorkItemId | Interno | ID del work item creado en Azure DevOps |
| WorkItemSync.devOpsUrl | Interno | URL del work item en Azure DevOps |

---

## Reglas de negocio

| ID | Regla | Descripcion | Justificacion |
|----|-------|------------|---------------|
| BR-001 | Sin spec aprobada no se genera | El portal no permite generar prompts ni invocar skills sobre specs en estado DRAFT o IN_REVIEW. Solo specs APPROVED pueden generar artefactos | Principio fundamental del framework: primero especificar, luego invocar |
| BR-002 | Versionado semver obligatorio | Toda spec se versiona con semver. Un cambio en L1 que rompe compatibilidad incrementa major; nuevas features incrementan minor; correcciones incrementan patch | Trazabilidad y gobernanza de cambios |
| BR-003 | Jerarquia L1→L2→L3 obligatoria | No se puede crear una L2 sin L1 parent aprobada. No se puede crear una L3 sin L2 parent aprobada | El framework define niveles independientes pero trazables entre si |
| BR-004 | Clasificacion de datos obligatoria | Toda spec L1 debe incluir clasificacion de datos por entidad/atributo segun ADR-006. El portal bloquea la transicion a IN_REVIEW si la clasificacion esta incompleta | Cumplimiento ADR-006 y normativas de proteccion de datos |
| BR-005 | Revision humana siempre | Toda spec requiere al menos un revisor humano con rol ARCHITECT o LEAD para pasar a APPROVED. La IA puede asistir la revision pero no aprobar | Principio del framework: la revision humana es el control principal de calidad |
| BR-006 | Prompt invoca spec y version | Los prompts generados deben referenciar explicitamente el specId y la version. No se permiten prompts sin referencia a spec | Trazabilidad prompt → spec → commit |
| BR-007 | DLP en prompts al LLM | Antes de enviar contexto al LLM para generar una spec, el portal sanitiza el input eliminando datos clasificados como Sensible o Confidencial que no sean necesarios para la generacion | Prevencion de fuga de datos segun ADR-006 |
| BR-008 | Auditoria de acciones criticas | Toda aprobacion, rechazo, generacion de prompt y cambio de estado de spec genera un registro en AuditLog con usuario, timestamp y detalles | Trazabilidad y compliance |
| BR-009 | PR menor a 400 lineas | El portal advierte cuando una spec L3 implica un cambio que probablemente supere 400 lineas de codigo, sugiriendo dividir en multiples L3 | Principio del framework: PRs quirurgicos |
| BR-010 | Skills versionados y medidos | Todo skill en el catalogo debe tener version, owner y metricas. Si un skill tiene > 20% retrabajo sostenido por 2 sprints, el portal alerta para rediseno | Mejora continua basada en datos |
| BR-011 | Sync a Azure DevOps al aprobar | Al aprobar una spec, el portal crea automaticamente un work item en Azure DevOps (Feature para L1, User Story para L2, Task para L3) con la informacion de la spec, trazabilidad y criterios de aceptacion. Si falla la sincronizacion, la spec queda aprobada pero se registra el error para reintento | Trazabilidad bidireccional spec → backlog. El equipo trabaja desde Azure DevOps Boards con referencia directa a la spec |

---

## Politicas de negocio

- El portal es una herramienta interna de Tech&Solve — sin acceso de clientes externos en MVP
- El contenido de las specs es propiedad intelectual de Tech&Solve y/o sus clientes — clasificado como Confidencial
- La autenticacion se delega a OAuth (GitHub/Azure AD) — no se implementa autenticacion propia
- El portal no reemplaza el repositorio Git — los artefactos aprobados se sincronizan bidirecionalmente
- El agente IA es un asistente: genera borradores, el humano valida y aprueba
- Los costos de LLM se controlan: Opus para generacion de L1 (contexto complejo), Sonnet para L2/L3 (tarea acotada), Haiku para validaciones de estructura
- En MVP no hay diferenciacion granular de permisos — los roles determinan que acciones puede hacer cada usuario
- La informacion de specs nunca se usa para entrenar modelos de terceros — se usan APIs con data retention deshabilitado

---

## Validaciones

| Validacion | Condicion | Error si falla |
|-----------|----------|---------------|
| Spec existe | El specId debe corresponder a una spec registrada en el proyecto | SPEC_NOT_FOUND |
| Parent aprobado | Para crear L2, el L1 parent debe estar APPROVED. Para L3, el L2 parent debe estar APPROVED | PARENT_NOT_APPROVED |
| Estructura completa | La spec debe tener todos los campos obligatorios segun su nivel (L1: entidades, reglas, casos de uso; L2: contrato, dependencias, errores; L3: cambio, impacto) | INCOMPLETE_STRUCTURE |
| Clasificacion de datos | Toda spec L1 debe tener clasificacion de datos por entidad | MISSING_DATA_CLASSIFICATION |
| Proteccion de datos | Toda spec L2 con campos Sensible debe tener seccion de proteccion de datos | MISSING_DATA_PROTECTION |
| Version valida | La version debe seguir formato semver (X.Y.Z) y ser mayor que la version anterior | INVALID_VERSION |
| Revisor diferente | El revisor no puede ser el mismo que el autor de la spec | SELF_REVIEW_NOT_ALLOWED |
| ADRs referenciados | La spec debe referenciar al menos los ADRs relevantes (ADR-001 si usa ports/adapters, ADR-006 si tiene PII) | MISSING_ADR_REFERENCES |
| Spec no duplicada | No puede haber dos specs del mismo nivel con el mismo nombre en estado DRAFT/IN_REVIEW | DUPLICATE_SPEC |
| Prompt sobre spec aprobada | Solo se puede generar prompt sobre spec en estado APPROVED | SPEC_NOT_APPROVED |

---

## Casos de uso

### CU-01: Gestionar proyecto y gobernanza

```
Como arquitecto o lider tecnico,
quiero crear un proyecto y configurar su gobernanza (ADRs, estandares, quality gates),
para que el equipo tenga las reglas claras antes de empezar a especificar.
```

**Comportamiento esperado:**

1. El arquitecto crea un proyecto con nombre, descripcion y URL del repositorio Git
2. El portal crea la estructura base del proyecto: carpeta de ADRs, estandares, quality gates
3. El arquitecto registra los ADRs criticos del proyecto (puede crearlos asistido por IA o importarlos del repositorio)
4. El arquitecto configura los quality gates: que valida cada gate, que es bloqueante
5. El portal muestra un dashboard de gobernanza: ADRs vigentes, estandares activos, quality gates configurados

### CU-02: Crear spec asistida por IA

```
Como desarrollador senior o lider tecnico,
quiero describir un requerimiento de negocio en lenguaje natural y que el agente genere la spec L1,
para que obtenga una especificacion completa y estructurada en minutos en lugar de horas.
```

**Comportamiento esperado:**

1. El usuario selecciona el proyecto y el nivel de spec a crear (L1, L2 o L3)
2. Si es L1: el usuario escribe la descripcion del dominio en lenguaje natural (problema, entidades, reglas)
3. El portal envia el contexto al agente IA con: input del usuario + template de spec + ADRs del proyecto + estandares aplicables
4. El agente genera un borrador de spec L1 con: entidades, reglas de negocio, casos de uso, clasificacion de datos, validaciones
5. El usuario revisa, edita secciones y ajusta el borrador
6. Si es L2: el usuario selecciona la L1 parent (aprobada) y describe el caso de uso tecnico. El agente genera la L2 con contrato, dependencias, errores, proteccion de datos
7. Si es L3: el usuario selecciona la L2 parent y describe el cambio. El agente genera la L3 con impacto y versionado
8. El usuario guarda la spec como DRAFT

### CU-03: Revisar y aprobar spec (Gate 1)

```
Como arquitecto o revisor,
quiero revisar una spec en un flujo estructurado con comentarios y decision formal,
para que la aprobacion quede registrada y la spec avance al siguiente paso del framework.
```

**Comportamiento esperado:**

1. El autor cambia el estado de la spec de DRAFT a IN_REVIEW
2. El portal valida automaticamente: estructura completa, clasificacion de datos, ADRs referenciados, version valida (validacion automatica pre-revision)
3. Si la validacion automatica falla, el portal muestra los hallazgos y no permite avanzar a IN_REVIEW
4. El revisor asignado recibe notificacion
5. El revisor lee la spec, agrega comentarios en linea (por seccion)
6. El revisor decide: APPROVED o CHANGES_REQUESTED
7. Si CHANGES_REQUESTED: el autor recibe los comentarios, ajusta y vuelve a enviar a revision
8. Si APPROVED: la spec queda en estado APPROVED con firma (revisor + fecha), se genera registro en AuditLog
9. El ciclo de revision se registra completo: comentarios, decisiones, tiempos
10. Al aprobar, el portal sincroniza automaticamente con Azure DevOps: crea un work item (Feature/User Story/Task segun nivel) en el proyecto configurado, con titulo, descripcion, criterios de aceptacion y link a la spec

### CU-04: Generar prompt estructurado

```
Como desarrollador senior,
quiero generar un prompt estructurado desde una spec aprobada,
para que pueda invocar la IA con el contexto correcto y las reglas del framework.
```

**Comportamiento esperado:**

1. El usuario selecciona una spec en estado APPROVED
2. El usuario selecciona el skill a invocar (SK-001 para caso de uso, SK-002 para tests, etc.)
3. El portal genera el prompt con: rol, invocacion de spec (ID + version), tarea, entrada/salida, reglas de gobernanza (ADRs relevantes), restricciones DLP
4. El usuario puede editar el prompt antes de copiarlo
5. El prompt generado se registra con referencia a spec + skill + usuario para trazabilidad
6. El usuario puede copiar el prompt al clipboard o enviarlo directamente a la herramienta de IA configurada

### CU-05: Dashboard de metricas y trazabilidad

```
Como arquitecto o lider tecnico,
quiero ver metricas del framework en tiempo real,
para que pueda tomar decisiones de mejora basadas en datos y no en intuicion.
```

**Comportamiento esperado:**

1. El portal muestra un dashboard por proyecto con:
   - **Gobernanza**: ADRs vigentes, specs por estado (DRAFT/IN_REVIEW/APPROVED), clasificacion de datos coverage
   - **Velocidad**: tiempo promedio de creacion de spec, tiempo promedio de ciclo de aprobacion
   - **Calidad**: % specs con estructura completa, % con clasificacion de datos, % con proteccion de datos
   - **Skills**: uso por skill, retrabajo por skill
   - **Trazabilidad**: cadena spec → prompt → commit (cuando se integre con Git en v2)
2. El dashboard permite filtrar por periodo, equipo y nivel de spec
3. Las metricas se calculan automaticamente desde los datos del portal

### CU-06: Gestionar catalogo de skills

```
Como arquitecto,
quiero administrar el catalogo de skills con versionado y metricas,
para que el equipo sepa que skills usar, cuando y con que efectividad.
```

**Comportamiento esperado:**

1. El arquitecto registra un skill con: nombre, tipo (Build/Workflow), version, owner, fase, criterios de uso
2. El portal asocia el skill a los prompts que lo invocan (trazabilidad skill → prompt → spec)
3. El portal calcula metricas por skill: frecuencia de uso, retrabajo asociado
4. Si un skill tiene > 20% retrabajo sostenido, el portal genera una alerta al owner
5. El arquitecto puede deprecar un skill y el portal previene su uso en nuevos prompts

### CU-07: Validacion automatica de estructura (pre-revision)

```
Como sistema,
quiero validar automaticamente la estructura de una spec antes de enviarla a revision,
para que el revisor humano se enfoque en la calidad del contenido, no en problemas de formato.
```

**Comportamiento esperado:**

1. Cuando una spec cambia a IN_REVIEW, el portal ejecuta SK-004 (revision automatizada)
2. Valida: campos obligatorios, versionado semver, clasificacion de datos (L1), proteccion de datos (L2), referencias a ADRs
3. Si hay hallazgos: muestra lista de errores con ubicacion y descripcion
4. Hallazgos criticos bloquean la transicion a IN_REVIEW
5. Hallazgos menores se muestran como advertencias (no bloquean pero se registran)

### CU-08: Sincronizar spec aprobada con Azure DevOps

```
Como arquitecto o lider tecnico,
quiero que al aprobar una spec se cree automaticamente un work item en Azure DevOps,
para que el equipo pueda planificar y trackear el trabajo desde Azure DevOps Boards con trazabilidad directa a la spec.
```

**Comportamiento esperado:**

1. Al aprobar una spec (transicion a APPROVED), el portal detecta que el proyecto tiene configurado un Azure DevOps project
2. El portal determina el tipo de work item segun el nivel de spec: Feature (L1), User Story (L2), Task (L3)
3. El portal crea el work item via Azure DevOps REST API con:
   - Titulo: `[SPEC-{level}] {titulo de la spec} v{version}`
   - Descripcion: resumen de la spec con link al portal
   - Criterios de aceptacion: extraidos de la spec (escenarios GWT para L2, reglas de negocio para L1)
   - Tags: `spec-agent`, `{nivel}`, `{version}`
   - Area Path: configurable por proyecto
   - Si es L2/L3: relacion Parent con el work item del spec parent
4. El portal registra la sincronizacion en WorkItemSync (specId, devOpsWorkItemId, URL)
5. Si la sincronizacion falla (Azure DevOps no disponible, error de permisos), la spec permanece APPROVED y el error se registra para reintento manual o automatico
6. El portal muestra el link al work item de Azure DevOps en la vista de la spec aprobada

---

## Criterios de aceptacion del dominio

- [ ] Un usuario puede crear un proyecto con gobernanza basica (ADRs + quality gates) en menos de 15 minutos
- [ ] El agente IA genera un borrador de spec L1 completo (entidades, reglas, casos de uso, clasificacion de datos) desde input en lenguaje natural en menos de 2 minutos
- [ ] El agente IA genera un borrador de spec L2 desde L1 aprobada en menos de 1 minuto
- [ ] La validacion automatica de estructura detecta el 100% de campos obligatorios faltantes
- [ ] El flujo de revision registra autor, revisor, comentarios, decision y timestamps en AuditLog
- [ ] No se puede generar un prompt sobre una spec no aprobada (enforcement de Gate 1)
- [ ] La clasificacion de datos es obligatoria: no se puede enviar a revision una L1 sin clasificacion
- [ ] Los prompts generados referencian explicitamente specId + version + skill
- [ ] El dashboard muestra metricas actualizadas sin intervencion manual
- [ ] El portal sanitiza el contexto antes de enviarlo al LLM (DLP pre-prompt)
- [ ] Al aprobar una spec, se crea automaticamente un work item en Azure DevOps con tipo correcto (Feature/User Story/Task)
- [ ] Si Azure DevOps no esta disponible, la spec queda aprobada y el error se registra para reintento

---

## Restricciones y consideraciones

| Tipo | Descripcion |
|------|------------|
| **Limitaciones de negocio** | MVP enfocado en equipos internos de Tech&Solve. Sin multi-organizacion. Sin integracion con CI/CD automatizada (v2). Maximo 50 usuarios concurrentes, 100 specs por proyecto |
| **Integraciones necesarias** | LLM externo (Claude API / OpenAI) para generacion asistida, OAuth (GitHub/Azure AD) para autenticacion, Git (GitHub/Azure DevOps) para sincronizacion de artefactos (v2) |
| **Consideraciones de seguridad** | OAuth para autenticacion. Contenido de specs es Confidencial. DLP pre-prompt obligatorio (SK-007). No enviar datos Sensibles/Confidenciales innecesarios al LLM. Auditoria de acciones criticas. Cumplimiento Ley 1581 y GDPR |
| **Consideraciones de rendimiento** | Generacion de spec L1 por IA < 2 min. Generacion de L2/L3 < 1 min. Validacion automatica < 10s. Dashboard < 3s. Todo procesamiento de IA es asincrono — el portal nunca bloquea al usuario |

---

## Glosario del dominio

| Termino | Definicion |
|---------|-----------|
| Spec (Specification) | Artefacto versionado que define QUE construir (L1), COMO tecnicamente (L2) o QUE cambiar (L3) |
| L1 / L2 / L3 | Los tres niveles de especificacion del framework: Dominio, Sistema, Cambio |
| ADR | Architecture Decision Record — decision tecnica documentada, enumerada y trazable |
| Quality Gate | Punto de control obligatorio que valida condiciones antes de avanzar al siguiente paso |
| Gate 1 | Spec versionada + aprobada + clasificacion de datos completa (antes de generar codigo) |
| Gate 2 | Tests + SAST + SCA + DLP Scan pasan (antes de revision humana de codigo) |
| Gate 3 | Revision de senior obligatoria (antes del merge) |
| Skill | Tarea repetible estandarizada con inputs, outputs y metricas (Build o Workflow) |
| Prompt estructurado | Prompt que invoca una spec versionada con rol, tarea y restricciones — no contiene reglas de negocio |
| DLP | Data Loss Prevention — prevencion de fuga de datos sensibles |
| Semver | Versionado semantico: MAJOR.MINOR.PATCH |
| Privacy by Design | Principio de proteger datos desde la especificacion, no como parche posterior |
| Trazabilidad | Cadena completa y auditable: spec → prompt → commit → release |
| Agente IA | Componente del portal que usa LLMs para asistir la generacion y validacion de specs |

---

## Diseno / Mockups

### Pantallas principales del MVP

1. **Dashboard de proyecto**: lista de specs por estado, ADRs vigentes, metricas rapidas
2. **Editor de spec**: formulario estructurado por nivel (L1/L2/L3) con asistente IA lateral
3. **Panel de revision**: spec con comentarios en linea, botones de aprobar/rechazar
4. **Prompt builder**: selector de spec + skill, preview del prompt generado, boton de copiar
5. **Dashboard de metricas**: graficos de cobertura, tiempos, calidad por periodo
6. **Catalogo de skills**: lista de skills con metricas, estados y acciones

---

## Referencias

- `claude.md` — Framework AI-Driven Engineering v1.1
- `solver-os/fase-1/specs/templates/` — Templates de especificacion L1, L2, L3
- `solver-os/fase-1/governance/adrs/` — ADRs del framework (001-006)
- `solver-os/fase-1/governance/quality-gates/QUALITY-GATES.md` — Quality gates
- `solver-os/fase-1/skills/SKILLS-CATALOG.md` — Catalogo de skills v1.1
- `solver-os/account-planning/specs/SPEC-L1-account-planning.md` — Ejemplo de L1 completo
- `context/AgenteSpec/scope.md` — Vision y alcance del producto

---

## Especificaciones relacionadas

| Spec ID | Nivel | Version | Descripcion |
|---------|-------|---------|------------|
| SPEC-L2-manage-projects | L2 | Pendiente | Gestion de proyectos y gobernanza (CU-01) |
| SPEC-L2-create-specs | L2 | Pendiente | Creacion de specs asistida por IA (CU-02) |
| SPEC-L2-review-specs | L2 | Pendiente | Flujo de revision y aprobacion (CU-03) |
| SPEC-L2-generate-prompts | L2 | Pendiente | Generacion de prompts estructurados (CU-04) |
| SPEC-L2-metrics-dashboard | L2 | Pendiente | Dashboard de metricas y trazabilidad (CU-05) |
| SPEC-L2-manage-skills | L2 | Pendiente | Gestion del catalogo de skills (CU-06) |
| SPEC-L2-sync-devops | L2 | Pendiente | Sincronizacion de specs con Azure DevOps (CU-08) |

---

## Historial de cambios

| Version | Fecha | Autor | Descripcion del cambio |
|---------|-------|-------|----------------------|
| 1.0.0 | 2026-04-09 | Equipo de Arquitectura | Version inicial — inception del Spec Agent Portal |
| 1.1.0 | 2026-04-09 | Equipo de Arquitectura | Agregar sincronizacion con Azure DevOps (CU-08, BR-011, entidad WorkItemSync) |
