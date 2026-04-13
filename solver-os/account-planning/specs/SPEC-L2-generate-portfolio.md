# Especificacion de Sistema (Nivel 2)

| Campo | Valor |
|-------|-------|
| **Spec ID** | SPEC-L2-generate-portfolio |
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

Modulo core de generacion de valor del sistema. Toma el analisis de una investigacion y genera un documento de account planning personalizado de 15 secciones, adaptado al rol del interlocutor (CEO, CTO, CFO, etc.) y al idioma seleccionado. El documento se genera en texto estructurado listo para copiar y pegar en PowerPoint. El comercial puede editar secciones individuales y, al aprobar, el plan alimenta automaticamente el RAG.

---

## Sistema

**Nombre:** Portfolio Generation Agent

## Caso de uso

**Nombre:** GeneratePortfolio

**Descripcion:** El comercial define el enfoque (tipo de empresa y rol del interlocutor) y el sistema genera un documento de account planning de 15 secciones con tono y foco adaptados. El PortfolioAgent (Semantic Kernel) produce texto ejecutivo, comercial y directo — cero relleno, cada punto genera una posible accion comercial. El comercial edita secciones individuales, y al aprobar, el plan aprobado se ingesta en el RAG.

---

## Contexto

- Los prompts del equipo comercial (Promejemplo.txt) definen una estructura de 15 puntos ya validada en la practica
- Hoy los comerciales personalizan manualmente un portafolio base de ~25 slides seleccionando 3-4 casos de exito relevantes
- El documento debe ser ejecutivo, comercial, directo — orientado a accion, no descriptivo
- La adaptacion por rol es clave: un CEO recibe vision estrategica, un CTO recibe soluciones tecnicas, un CFO recibe ROI
- La generacion no es un resumen de la investigacion — es una interpretacion estrategica con oportunidades accionables
- Los estilos esperados: "Lenguaje ejecutivo y comercial, cero tecnicismos innecesarios, enfoque en impacto, riesgo y valor, pensado para presentar a comite o direccion"
- El portafolio aprobado se convierte en fuente de verdad y alimenta el RAG

---

## Contrato tecnico

### Input — GeneratePortfolio

| Campo | Tipo | Requerido | Descripcion |
|-------|------|-----------|------------|
| investigationId | string (UUID) | Si | ID de la investigacion (debe tener analisis completado) |
| targetCompanyType | string | Si | Tipo de empresa (ej: "Aseguradora de vida y generales", "Banco comercial") |
| targetRole | string | Si | Rol del interlocutor: CEO, CTO, CDO, CFO, RISK_DIRECTOR, EXPERIENCE_DIRECTOR, OPERATIONS_DIRECTOR, OTHER |
| language | string | No | Idioma del portfolio: ES (default), EN, PT |
| customInstructions | string | No | Instrucciones adicionales del comercial (ej: "Enfocarse en temas de IA y automatizacion") |

### Input — EditPortfolioSection

| Campo | Tipo | Requerido | Descripcion |
|-------|------|-----------|------------|
| portfolioId | string (UUID) | Si | ID del portfolio |
| sectionKey | string | Si | Key de la seccion a editar (ej: "executiveSummary", "opportunities") |
| content | string | Si | Contenido editado por el comercial |

### Input — ApprovePortfolio

| Campo | Tipo | Requerido | Descripcion |
|-------|------|-----------|------------|
| portfolioId | string (UUID) | Si | ID del portfolio a aprobar |

### Input — RegeneratePortfolio

| Campo | Tipo | Requerido | Descripcion |
|-------|------|-----------|------------|
| portfolioId | string (UUID) | Si | ID del portfolio existente |
| language | string | No | Nuevo idioma (si se quiere cambiar) |
| targetRole | string | No | Nuevo rol (si se quiere cambiar) |
| customInstructions | string | No | Nuevas instrucciones |

### Output — PortfolioResponse

| Campo | Tipo | Descripcion |
|-------|------|------------|
| portfolioId | string (UUID) | ID unico del portfolio |
| investigationId | string (UUID) | ID de la investigacion fuente |
| targetRole | string | Rol del interlocutor |
| language | string | Idioma del documento |
| status | string | DRAFT, APPROVED |
| sections | PortfolioSection[] | Las 15 secciones del documento |
| sections[].key | string | Identificador de la seccion |
| sections[].title | string | Titulo de la seccion |
| sections[].content | string | Contenido en texto estructurado (markdown) |
| sections[].isEdited | boolean | Si el comercial edito esta seccion |
| generatedAt | string (ISO 8601) | Timestamp de generacion |
| approvedAt | string (ISO 8601) | Timestamp de aprobacion (null si DRAFT) |

### Estructura de las 15 secciones

| # | Key | Titulo | Descripcion | Adaptacion por rol |
|---|-----|--------|------------|-------------------|
| 1 | executiveSummary | Resumen Ejecutivo | Lectura estrategica (no descriptiva). Nivel de madurez, foco estrategico, por que es cuenta atractiva | CEO: vision de negocio. CTO: posicion tecnologica. CFO: potencial financiero |
| 2 | companyProfile | Perfil de la Compania | Solo lo relevante para vender: nombre, industria, ubicacion, tamano, posicionamiento | Igual para todos — factual |
| 3 | mainChallenges | Principales Retos hacia 2026 | Estrategicos, operativos/tech, experiencia de cliente, regulatorios/riesgo | CEO: estrategicos. CTO: tech/operativos. CFO: eficiencia/regulatorios |
| 4 | techAdoptionVision | Vision de Adopcion Tecnologica | Que esta haciendo hoy + hacia donde deberia ir (vision 2026) | CTO/CDO: profundo. CEO: alto nivel. CFO: ROI de adopcion |
| 5 | stakeholders | Stakeholders y Decisores | Decisores estrategicos, tech, influenciadores, gatekeeper, decisor financiero. Para cada uno: que valora, dolores probables, como abordarlo | Adapta el enfoque segun quien sea el interlocutor |
| 6 | decisionStructure | Estructura de Decision | Modelo (comite/jerarquico), quien influye mas, como se aprueban compras | Igual para todos — critico para estrategia de venta |
| 7 | parentCompanyAssociations | Casa Matriz y Agremiaciones | Pais, casa matriz, agremiaciones relevantes (FASECOLDA, etc.) | Igual para todos |
| 8 | whatTheyBuy | Que Compra | Tecnologia y servicios que compra actualmente | CTO: detalle tecnico. CFO: inversion/gasto. CEO: vision de partners |
| 9 | whoTheyBuyFrom | A Quien le Compra | Proveedores globales, partners locales, oportunidades de reemplazo o complemento | Todos: enfocado en posicionamiento competitivo |
| 10 | strategicReading | Lectura Estrategica de Publicaciones | Temas recurrentes, mensajes implicitos, lectura comercial | CEO: tendencias. CTO: innovacion. CFO: indicadores |
| 11 | opportunities | Oportunidades para Tech&Solve | Corto plazo, mediano plazo, largo plazo — priorizadas | CEO: impacto estrategico. CTO: soluciones. CFO: ahorro/ROI |
| 12 | opportunityHypothesis | Hipotesis de Oportunidad | Hipotesis clara, riesgo que mitiga, valor que genera | Igual para todos — accionable |
| 13 | strategicQuestions | Preguntas Estrategicas | Preguntas para abrir conversacion: estrategia/finanzas, tecnologia, datos/regulacion, futuro | Adapta preguntas al rol del interlocutor |
| 14 | painValueServiceMap | Mapa Dolor-Valor-Servicio | Tabla de 3 columnas lista para presentacion | Igual para todos — artefacto core |
| 15 | keyMessage | Mensaje Clave | Una sola frase poderosa, orientada a negocio y riesgo | CEO: vision. CTO: habilitacion. CFO: valor economico |

---

## Comportamiento esperado

1. El sistema valida que la investigacion tiene un analisis generado (SPEC-L2-analyze-results)
2. Recupera el analisis completo (hallazgos, stakeholders, oportunidades, mapa dolor-valor-servicio)
3. Invoca el PortfolioAgent (Semantic Kernel) con:
   - El analisis como input
   - El rol del interlocutor
   - El idioma
   - Instrucciones personalizadas del comercial (si las hay)
   - El catalogo de servicios y casos de exito de Tech&Solve
4. El agente genera las 15 secciones con:
   - Tono ejecutivo, comercial, directo — cero relleno
   - Cada punto genera una posible accion comercial
   - Adaptacion de contenido y enfoque segun el rol
   - Idioma seleccionado
5. Persiste el portfolio con status DRAFT
6. El comercial puede:
   - **Editar secciones** individuales (EditPortfolioSection) — el sistema marca isEdited=true
   - **Regenerar** el portfolio con diferentes parametros (RegeneratePortfolio)
   - **Aprobar** el portfolio (ApprovePortfolio) — cambia status a APPROVED
7. Al aprobar:
   - Se marca approvedAt
   - Se publica evento PortfolioApproved
   - El portfolio se ingesta automaticamente en el RAG como APPROVED_PLAN (BR-005)

---

## Tipos de error

| Codigo de error | HTTP Status | Descripcion | Condicion de disparo |
|----------------|-------------|------------|---------------------|
| INVESTIGATION_NOT_FOUND | 404 | La investigacion no existe | investigationId no encontrado |
| ANALYSIS_NOT_FOUND | 404 | No hay analisis generado para esta investigacion | No se ha ejecutado AnalyzeAndDisplayResults |
| PORTFOLIO_NOT_FOUND | 404 | El portfolio no existe | portfolioId no encontrado |
| PORTFOLIO_GENERATION_FAILED | 500 | Error al generar el portfolio | Timeout o error del PortfolioAgent |
| INVALID_LANGUAGE | 400 | Idioma no soportado | language no es ES, EN o PT |
| INVALID_ROLE | 400 | Rol del interlocutor no reconocido | targetRole no esta en la lista permitida |
| INVALID_SECTION | 400 | Seccion no existe | sectionKey no corresponde a ninguna de las 15 secciones |
| PORTFOLIO_ALREADY_APPROVED | 409 | No se puede editar un portfolio aprobado | Se intenta editar un portfolio con status APPROVED |

---

## Dependencias permitidas

> **REGLA (ADR-004):** Solo estas dependencias pueden ser usadas en la implementacion.

| Dependencia | Proposito | Tipo de interfaz |
|------------|----------|-----------------|
| AnalysisRepository | Obtener el analisis generado de la investigacion | Puerto (IAnalysisRepository) |
| PortfolioAgent | Agente de IA (Semantic Kernel) que genera las 15 secciones del portfolio | Puerto (IPortfolioAgent) |
| ServiceCatalog | Consultar catalogo de servicios de Tech&Solve para incluir en oportunidades | Puerto (IServiceCatalog) |
| CaseStudyCatalog | Consultar casos de exito para incluir en el portfolio | Puerto (ICaseStudyCatalog) |
| PortfolioRepository | Persistir portfolios (CRUD + edicion de secciones) | Puerto (IPortfolioRepository) |
| RAGEngine | Ingestar el portfolio aprobado como documento de conocimiento | Puerto (IRAGEngine) |
| EventBus | Publicar eventos: PortfolioGenerated, PortfolioApproved | Puerto (IEventPublisher) |

---

## Requisitos de ejecucion

| Requisito | Valor | Notas |
|-----------|-------|-------|
| **Latencia maxima (generacion)** | 30000 ms (30s) | Incluye llamada al PortfolioAgent |
| **Latencia maxima (edicion)** | 500 ms (p95) | Operacion CRUD simple |
| **Latencia maxima (aprobacion)** | 2000 ms (p95) | Incluye ingesta al RAG |
| **Idempotencia** | No requerida | Cada generacion puede dar resultados ligeramente diferentes |
| **Atomicidad** | Transaccion atomica por operacion | |
| **Observabilidad** | Ver abajo | |

### Observabilidad requerida

- **Logs:** Portfolio generado (INFO), seccion editada (INFO), portfolio aprobado (INFO), ingesta en RAG completada (INFO), error de generacion (ERROR)
- **Trazas:** span por generacion completa, span por llamada al PortfolioAgent, span por ingesta en RAG
- **Metricas:** portfolio_generation_duration_ms, sections_edited_per_portfolio, portfolios_approved_total, portfolios_by_role, portfolios_by_language
- **Alertas:** generation error rate > 5% en 5 min, generation duration p95 > 45s

---

## Restricciones de arquitectura

| ADR | Titulo | Impacto en esta spec |
|-----|--------|---------------------|
| ADR-001 | Ports & Adapters | GeneratePortfolio depende de 7 puertos. PortfolioAgent es un puerto — se puede cambiar el modelo de IA sin afectar el core |
| ADR-004 | Dependency Management | Solo las 7 dependencias listadas |
| ADR-005 | Testing Strategy | Tests GWT antes de implementar. Mock del PortfolioAgent en tests |

---

## Restricciones y consideraciones

| Tipo | Descripcion |
|------|------------|
| **Limitaciones tecnicas** | La calidad del portfolio depende de la calidad del analisis previo. Semantic Kernel gestiona el agente y los plugins |
| **Integraciones necesarias** | Semantic Kernel como orquestador del PortfolioAgent. Vector store para ingesta en RAG |
| **Consideraciones de seguridad** | No incluir datos sensibles de personas sin consentimiento. Sanitizar inputs del comercial antes de pasarlos al agente |
| **Consideraciones de rendimiento** | Generacion en < 30s. Edicion de secciones es CRUD simple. Ingesta en RAG puede ser async |

---

## Criterios de aceptacion

- [ ] El portfolio tiene las 15 secciones con contenido relevante y no generico
- [ ] El tono es ejecutivo, comercial y directo — cero relleno, orientado a accion
- [ ] El contenido se adapta al rol del interlocutor (CEO vs CTO vs CFO tienen enfoque diferente)
- [ ] Las oportunidades estan cruzadas con servicios reales de Tech&Solve
- [ ] El mapa dolor-valor-servicio tiene formato de tabla listo para PPT
- [ ] El comercial puede editar secciones individuales sin regenerar todo el documento
- [ ] Al aprobar, el portfolio se ingesta automaticamente en el RAG
- [ ] El cambio de idioma regenera sin relanzar la investigacion
- [ ] El mensaje clave es una sola frase poderosa orientada a negocio y riesgo

---

## Escenarios de prueba (GWT)

> **REGLA (ADR-005):** Estos escenarios deben traducirse en tests ANTES de generar la implementacion.

| ID | Given (Dado que) | When (Cuando) | Then (Entonces) | Prioridad |
|----|-----------------|--------------|-----------------|-----------|
| TS-001 | Existe un analisis completo para "Seguros Bolivar" | Se genera portfolio con targetRole=CEO, language=ES | Portfolio con 15 secciones, tono estrategico, enfocado en vision de negocio e impacto | Alta |
| TS-002 | Existe un analisis completo para "Seguros Bolivar" | Se genera portfolio con targetRole=CTO, language=ES | Portfolio con 15 secciones, enfocado en soluciones tecnicas, legacy vs modernizacion, casos de exito tech | Alta |
| TS-003 | Existe un portfolio DRAFT con 15 secciones | Se edita la seccion "opportunities" con contenido personalizado | La seccion se actualiza, isEdited=true, demas secciones intactas | Alta |
| TS-004 | Existe un portfolio DRAFT editado | Se aprueba el portfolio | Status cambia a APPROVED, approvedAt se registra, evento PortfolioApproved publicado, portfolio ingestado en RAG | Alta |
| TS-005 | Existe un analisis completo | Se genera portfolio con language=ES (default) | Todas las secciones en espanol | Media |
| TS-006 | Existe un portfolio en espanol | Se regenera con language=EN | Portfolio regenerado en ingles, investigacion no se relanza, solo el documento cambia | Alta |
| TS-007 | Existe un analisis con pocos hallazgos (solo web, sin LinkedIn) | Se genera portfolio | Portfolio generado con las 15 secciones, marcando donde la informacion es limitada e infiriendo razonablemente | Media |
| TS-008 | El RAG tiene 3 casos de exito del sector seguros | Se genera portfolio para una aseguradora | Las secciones de oportunidades e hipotesis incluyen referencias a casos de exito relevantes | Alta |
| TS-009 | Existe un portfolio con status APPROVED | Se intenta editar una seccion | Error PORTFOLIO_ALREADY_APPROVED | Media |
| TS-010 | No existe analisis para la investigacion | Se intenta generar portfolio | Error ANALYSIS_NOT_FOUND | Alta |

---

## Referencias

- SPEC-L1-account-planning v2.0.0 — Dominio (BR-004, BR-005, BR-007)
- SPEC-L2-analyze-results v1.0.0 — Analisis que sirve de input
- context/Promejemplo.txt — Estructura de 15 puntos y estilo esperado
- context/scope.md — Seccion "Generacion del portafolio personalizado"
- context/TranscriptNegocio.txt — Como los comerciales usan el portafolio hoy

---

## Especificaciones relacionadas

| Spec ID | Nivel | Version | Relacion |
|---------|-------|---------|----------|
| SPEC-L1-account-planning | L1 | 2.0.0 | Parent — dominio |
| SPEC-L2-analyze-results | L2 | 1.0.0 | Upstream — provee el analisis |
| SPEC-L2-manage-knowledge-base | L2 | 1.0.0 | Downstream — portfolio aprobado alimenta RAG |
| SPEC-L3-language-switch | L3 | 1.0.0 | Child — cambio de idioma sin re-investigar |

---

## Historial de cambios

| Version | Fecha | Autor | Descripcion del cambio |
|---------|-------|-------|----------------------|
| 1.0.0 | 2026-04-09 | Juan Gabriel Colorado | Version inicial: generacion de portfolio de 15 secciones adaptado por rol |
