# Especificacion de Sistema (Nivel 2)

| Campo | Valor |
|-------|-------|
| **Spec ID** | SPEC-L2-execute-research |
| **Nivel** | L2 — Sistema |
| **Version** | 1.0.0 |
| **Parent L1** | SPEC-L1-account-planning v2.0.0 |
| **Estado** | DRAFT |
| **Creado por** | @juan.gabriel.colorado |
| **Fecha de creacion** | 2026-04-09 |
| **Aprobado por** | Pendiente |
| **Fecha de aprobacion** | Pendiente |
| **PR** | Pendiente |

---

## Descripcion general

Modulo central del sistema que ejecuta la investigacion sobre un cliente en dos fases: busqueda rapida (confirmar cliente) y busqueda profunda (investigacion completa asincrona). Recopila informacion de multiples fuentes (web, LinkedIn, RAG interno), extrae hallazgos con atribucion de fuentes y stakeholders identificados.

---

## Sistema

**Nombre:** Research Engine

## Caso de uso

**Nombre:** ExecuteResearch

**Descripcion:** El comercial describe su necesidad en lenguaje natural. El sistema ejecuta una busqueda en dos fases: (1) busqueda rapida (~60s) para confirmar que el cliente es correcto, (2) tras confirmacion, busqueda profunda asincrona (2-5 min) que investiga en web, LinkedIn y RAG interno, generando hallazgos con atribucion de fuentes y stakeholders identificados. El comercial puede retomar la investigacion si cierra el navegador.

---

## Contexto

- Hoy los comerciales buscan manualmente en Google, LinkedIn, prensa, ChatGPT — proceso de 2-5 horas
- LinkedIn no permite acceso automatico via API facilmente — se necesita fallback manual
- El equipo usa Lusha como herramienta complementaria para obtener contactos
- Las fuentes web deben ser portales confiables y configurables (prensa empresarial, portales financieros)
- El RAG contiene: casos de exito de Tech&Solve, catalogo de servicios, account plannings aprobados
- El procesamiento pesado debe ser completamente asincrono — el sistema nunca bloquea al usuario
- El frontend consulta el estado del job cada ~5 segundos (polling, no SignalR en MVP)

---

## Contrato tecnico

### Input — StartQuickSearch

| Campo | Tipo | Requerido | Descripcion |
|-------|------|-----------|------------|
| clientId | string (UUID) | Si | ID del cliente a investigar |
| userQuery | string | Si | Descripcion en lenguaje natural de la necesidad del comercial. Ej: "Quiero preparar una reunion con Bancolombia para presentarles servicios de automatizacion" |

### Input — ConfirmAndStartDeepSearch

| Campo | Tipo | Requerido | Descripcion |
|-------|------|-----------|------------|
| investigationId | string (UUID) | Si | ID de la investigacion (generado en quick search) |
| confirmed | boolean | Si | true si el cliente es correcto, false para cancelar |
| linkedinManualInput | string | No | Texto del perfil de LinkedIn pegado manualmente si el auto-fetch fallo |

### Input — GetInvestigationStatus

| Campo | Tipo | Requerido | Descripcion |
|-------|------|-----------|------------|
| investigationId | string (UUID) | Si | ID de la investigacion a consultar |

### Output — QuickSearchResponse

| Campo | Tipo | Descripcion |
|-------|------|------------|
| investigationId | string (UUID) | ID unico de la investigacion creada |
| clientId | string (UUID) | ID del cliente |
| status | string | "CONFIRMING" — esperando confirmacion del comercial |
| quickSummary | object | Resumen para confirmar el cliente |
| quickSummary.companyName | string | Nombre de la empresa encontrado |
| quickSummary.industry | string | Sector identificado |
| quickSummary.country | string | Pais identificado |
| quickSummary.description | string | Descripcion breve de la empresa |
| quickSummary.confidence | string | HIGH, MEDIUM o LOW — que tan seguro esta el sistema de que es el cliente correcto |
| quickSummary.sources | string[] | Fuentes usadas para el resumen rapido |
| startedAt | string (ISO 8601) | Timestamp de inicio |

### Output — InvestigationStatusResponse

| Campo | Tipo | Descripcion |
|-------|------|------------|
| investigationId | string (UUID) | ID de la investigacion |
| clientId | string (UUID) | ID del cliente |
| status | string | CONFIRMING, DEEP_SEARCHING, COMPLETED, FAILED, CANCELLED |
| progress | object | Progreso de la busqueda profunda |
| progress.percentage | integer (0-100) | Porcentaje completado |
| progress.currentPhase | string | Fase actual: WEB_SEARCH, LINKEDIN, RAG_SEARCH, ANALYSIS |
| progress.estimatedTimeRemaining | integer | Segundos estimados restantes |
| findings | Finding[] | Hallazgos (disponibles progresivamente) |
| findings[].findingId | string (UUID) | ID del hallazgo |
| findings[].source | string | Nombre de la fuente (ej: "El Tiempo", "LinkedIn", "RAG interno") |
| findings[].sourceUrl | string | URL de la fuente (null para RAG) |
| findings[].date | string (ISO 8601) | Fecha del hallazgo/publicacion |
| findings[].confidence | string | HIGH, MEDIUM, LOW |
| findings[].content | string | Contenido del hallazgo |
| findings[].type | string | NEWS, FINANCIAL, PUBLICATION, LINKEDIN, CASE_STUDY, SERVICE_MATCH |
| stakeholders | Stakeholder[] | Personas clave identificadas |
| stakeholders[].stakeholderId | string (UUID) | ID del stakeholder |
| stakeholders[].name | string | Nombre |
| stakeholders[].role | string | Cargo en la organizacion |
| stakeholders[].relevance | string | Relevancia comercial para Tech&Solve |
| stakeholders[].linkedinUrl | string | URL del perfil de LinkedIn (si se encontro) |
| startedAt | string (ISO 8601) | Cuando inicio la investigacion |
| completedAt | string (ISO 8601) | Cuando termino (null si en progreso) |

---

## Comportamiento esperado

### Fase 1: Quick Search
1. El sistema recibe el clientId y userQuery del comercial
2. Valida que el cliente existe
3. Valida que no hay una investigacion en progreso para este cliente
4. Crea una investigacion con status "CONFIRMING"
5. Ejecuta una busqueda web rapida (~60s) usando el nombre del cliente y la query
6. Genera un resumen con: nombre de la empresa, sector, pais, descripcion breve, nivel de confianza
7. Retorna el resumen para que el comercial confirme

### Fase 2: Confirmacion + Deep Search
1. El comercial confirma (confirmed=true) o cancela (confirmed=false)
2. Si cancela: status pasa a "CANCELLED", fin del flujo
3. Si confirma: status pasa a "DEEP_SEARCHING"
4. El sistema lanza un job asincrono que:
   a. **Web Search** — busca en portales confiables configurables: prensa, noticias, pagina web del cliente, estados financieros publicos
   b. **LinkedIn** — intenta obtener el perfil de la empresa automaticamente. Si falla y el comercial no pego texto manual, marca como LINKEDIN_UNAVAILABLE pero continua
   c. **RAG Search** — busca en la base de conocimiento interna: casos de exito similares, servicios relevantes del catalogo, planes aprobados de empresas del mismo sector
   d. **Stakeholder Extraction** — identifica personas clave de los hallazgos (nombre, cargo, relevancia)
   e. **Source Attribution** — cada hallazgo se asocia a su fuente con URL y fecha
5. Los hallazgos se persisten progresivamente (el frontend puede ver resultados parciales)
6. Al completar todas las fases: status pasa a "COMPLETED"
7. Si hay error irrecuperable: status pasa a "FAILED" con detalle del error

### Polling (GetInvestigationStatus)
1. El frontend consulta cada ~5 segundos
2. Retorna el estado actual, progreso, hallazgos disponibles y stakeholders
3. Funciona incluso si el comercial cerro y reabrio el navegador

---

## Tipos de error

| Codigo de error | HTTP Status | Descripcion | Condicion de disparo |
|----------------|-------------|------------|---------------------|
| CLIENT_NOT_FOUND | 404 | El cliente no existe | clientId no encontrado |
| INVESTIGATION_NOT_FOUND | 404 | La investigacion no existe | investigationId no encontrado |
| INVESTIGATION_IN_PROGRESS | 409 | Ya hay una investigacion en progreso para este cliente | Se intenta iniciar otra quick search |
| SEARCH_TIMEOUT | 504 | La busqueda excedio el tiempo maximo | Quick > 90s o Deep > 10 min |
| LINKEDIN_UNAVAILABLE | 200 (parcial) | LinkedIn no disponible — no es bloqueante | Auto-fetch de LinkedIn fallo y no hay input manual |
| RAG_UNAVAILABLE | 200 (parcial) | RAG no disponible — no es bloqueante | Vector store no responde |
| NO_SOURCES_AVAILABLE | 503 | Ninguna fuente de datos responde | Web, LinkedIn y RAG todos fallan |
| INVALID_QUERY | 400 | Query del comercial esta vacia o es invalida | userQuery vacio o < 10 caracteres |

---

## Dependencias permitidas

> **REGLA (ADR-004):** Solo estas dependencias pueden ser usadas en la implementacion.

| Dependencia | Proposito | Tipo de interfaz |
|------------|----------|-----------------|
| ClientRepository | Validar que el cliente existe | Puerto (IClientRepository) |
| WebSearchEngine | Buscar en portales web confiables | Puerto (IWebSearchEngine) |
| LinkedInAdapter | Obtener perfil de LinkedIn de la empresa | Puerto (ILinkedInAdapter) |
| RAGEngine | Buscar en la base de conocimiento interna | Puerto (IRAGEngine) |
| StakeholderExtractor | Extraer personas clave de los hallazgos | Puerto (IStakeholderExtractor) |
| InvestigationRepository | Persistir investigaciones, hallazgos y stakeholders | Puerto (IInvestigationRepository) |
| BackgroundJobRunner | Ejecutar busqueda profunda como job asincrono | Puerto (IBackgroundJobRunner) |
| EventBus | Publicar eventos: InvestigationStarted, InvestigationCompleted | Puerto (IEventPublisher) |

---

## Requisitos de ejecucion

| Requisito | Valor | Notas |
|-----------|-------|-------|
| **Latencia maxima (Quick)** | 60000 ms (60s) | Busqueda rapida sincrona |
| **Latencia maxima (Deep)** | 300000 ms (5 min) | Busqueda profunda asincrona |
| **Latencia maxima (Polling)** | 200 ms (p95) | Solo lectura de estado desde DB |
| **Idempotencia** | No requerida | Cada investigacion puede dar resultados diferentes |
| **Atomicidad** | Eventual consistency | Hallazgos se persisten progresivamente |
| **Observabilidad** | Ver abajo | |

### Observabilidad requerida

- **Logs:** Investigacion iniciada (INFO), busqueda web completada (INFO), LinkedIn fallo (WARN), RAG consultado (INFO), stakeholder extraido (INFO), investigacion completada (INFO), investigacion fallida (ERROR)
- **Trazas:** span por investigacion completa, span por cada fuente (web, LinkedIn, RAG), span por extraccion de stakeholders
- **Metricas:** investigation_duration_seconds (por tipo quick/deep), findings_per_investigation, stakeholders_per_investigation, source_success_rate (por fuente), linkedin_fallback_rate
- **Alertas:** error rate > 10% en 5 min, deep search duration p95 > 7 min, no_sources_available > 0 en 1 min

---

## Restricciones de arquitectura

| ADR | Titulo | Impacto en esta spec |
|-----|--------|---------------------|
| ADR-001 | Ports & Adapters | ExecuteResearch depende de 8 puertos, no de implementaciones. LinkedInAdapter se puede reemplazar sin afectar el core |
| ADR-004 | Dependency Management | Solo las 8 dependencias listadas. No inventar scrapers o conectores adicionales |
| ADR-005 | Testing Strategy | Tests GWT antes de implementar. Mock de fuentes externas en tests |

---

## Restricciones y consideraciones

| Tipo | Descripcion |
|------|------------|
| **Limitaciones tecnicas** | LinkedIn no permite acceso automatico facilmente — siempre tener fallback manual. Sin SignalR en MVP — usar polling. Job async debe sobrevivir al cierre del navegador |
| **Integraciones necesarias** | Web search engine (configurable), LinkedIn (con fallback), Vector store para RAG |
| **Consideraciones de seguridad** | No almacenar PII de stakeholders sin consentimiento. Sanitizar datos antes de enviar a IA. No scraping agresivo de sitios web |
| **Consideraciones de rendimiento** | 10-20 investigaciones diarias. Deep search async con workers. Hallazgos se persisten progresivamente para feedback temprano al usuario |

---

## Criterios de aceptacion

- [ ] El comercial escribe una query en lenguaje natural y recibe un resumen de confirmacion en < 60 segundos
- [ ] Tras confirmar, la busqueda profunda se lanza en background sin bloquear la UI
- [ ] Los hallazgos incluyen fuente, URL, fecha y nivel de confianza
- [ ] Si LinkedIn falla, el sistema continua con web + RAG y muestra opcion de input manual
- [ ] El frontend puede consultar el progreso de la busqueda via polling
- [ ] Si el comercial cierra el navegador, la busqueda sigue y puede retomar al volver
- [ ] Los stakeholders identificados incluyen nombre, cargo y relevancia
- [ ] No se pueden tener dos investigaciones en progreso para el mismo cliente

---

## Escenarios de prueba (GWT)

> **REGLA (ADR-005):** Estos escenarios deben traducirse en tests ANTES de generar la implementacion.

| ID | Given (Dado que) | When (Cuando) | Then (Entonces) | Prioridad |
|----|-----------------|--------------|-----------------|-----------|
| TS-001 | Existe un cliente "Seguros Bolivar" registrado | Se inicia quick search con query "preparar reunion de automatizacion" | Retorna quickSummary con nombre, sector, pais, descripcion en < 60s, status CONFIRMING | Alta |
| TS-002 | Existe una investigacion en status CONFIRMING | El comercial confirma (confirmed=true) | Status cambia a DEEP_SEARCHING, job async se lanza | Alta |
| TS-003 | La busqueda profunda esta en progreso | Se consulta el estado (polling) | Retorna status DEEP_SEARCHING, progress con porcentaje y fase actual, hallazgos parciales disponibles | Alta |
| TS-004 | La busqueda profunda completo exitosamente | Se consulta el estado | Status COMPLETED, findings con atribucion de fuentes, stakeholders con nombre y cargo | Alta |
| TS-005 | LinkedIn auto-fetch falla durante deep search | La busqueda profunda continua | Status sigue DEEP_SEARCHING, hallazgos de web y RAG estan disponibles, findings de LinkedIn ausentes pero no bloquea | Alta |
| TS-006 | LinkedIn fallo y el comercial pega el texto del perfil manualmente | Se confirma deep search con linkedinManualInput | El texto pegado se procesa como fuente adicional con type LINKEDIN | Media |
| TS-007 | El RAG no tiene documentos relevantes para el sector | Se ejecuta deep search | La investigacion completa sin hallazgos de tipo CASE_STUDY/SERVICE_MATCH pero no falla | Media |
| TS-008 | Ya hay una investigacion DEEP_SEARCHING para "Bancolombia" | Se intenta iniciar otra quick search para "Bancolombia" | Error INVESTIGATION_IN_PROGRESS | Alta |
| TS-009 | Una investigacion esta en progreso y el comercial cierra el navegador | El comercial vuelve y consulta GetInvestigationStatus | Retorna el estado actual de la investigacion con hallazgos acumulados | Alta |
| TS-010 | El comercial escribe "Bancolombia automatizacion de procesos" | Se inicia quick search | El sistema interpreta la query y busca informacion de Bancolombia enfocada en automatizacion | Media |

---

## Referencias

- SPEC-L1-account-planning v2.0.0 — Dominio (BR-001, BR-002, BR-003, BR-006)
- context/scope.md — Secciones "Flujo conversacional de investigacion" y "Fuentes de investigacion"
- context/TranscriptNegocio.txt — Proceso manual actual del equipo comercial
- context/TranscriptSesiónPresentaciónAndres.txt — Arquitectura tecnica discutida

---

## Especificaciones relacionadas

| Spec ID | Nivel | Version | Relacion |
|---------|-------|---------|----------|
| SPEC-L1-account-planning | L1 | 2.0.0 | Parent — dominio |
| SPEC-L2-manage-clients | L2 | 1.0.0 | Dependencia — requiere cliente registrado |
| SPEC-L2-analyze-results | L2 | 1.0.0 | Siguiente paso — analiza los resultados de la investigacion |
| SPEC-L3-manual-linkedin-fallback | L3 | 1.0.0 | Child — detalle del fallback manual de LinkedIn |

---

## Historial de cambios

| Version | Fecha | Autor | Descripcion del cambio |
|---------|-------|-------|----------------------|
| 1.0.0 | 2026-04-09 | Juan Gabriel Colorado | Version inicial: flujo de investigacion en dos fases (quick + deep) con multiples fuentes |
