# Especificacion de Dominio (Nivel 1)

| Campo | Valor |
|-------|-------|
| **Spec ID** | SPEC-L1-account-planning |
| **Nivel** | L1 — Dominio |
| **Version** | 2.0.0 |
| **Estado** | DRAFT |
| **Creado por** | @juan.gabriel.colorado |
| **Fecha de creacion** | 2026-04-09 |
| **Aprobado por** | Pendiente |
| **Fecha de aprobacion** | Pendiente |
| **PR** | Pendiente |

---

## Descripcion general

Sistema inteligente de perfilamiento de clientes potenciales para el equipo comercial de Tech&Solve. Automatiza la creacion de account plannings estrategicos: el comercial describe su necesidad en lenguaje natural, el sistema investiga en multiples fuentes, analiza con IA y genera un portfolio personalizado listo para presentar.

---

## Dominio

**Nombre:** Account Planning Inteligente

## Objetivo de negocio

Reducir el tiempo de preparacion de un account planning de **2-5 horas** (proceso manual actual) a **minutos**, permitiendo que los BDOs (Business Development Officers) lleguen a cada reunion con un documento estrategico personalizado, respaldado por datos reales y cruzado con el portafolio de servicios de Tech&Solve.

### Problema que resuelve

Hoy en Tech&Solve:
- La preparacion de un account planning es **manual, lenta y depende del conocimiento individual** de cada comercial
- El 80% de la informacion se obtiene manualmente de la IA (ChatGPT/Claude) pero sin sistematizar ni persistir
- La informacion de cada cliente esta dispersa en SharePoint, correos y documentos individuales
- No hay forma de reutilizar investigaciones previas ni aprender de planes exitosos
- Cada comercial usa prompts diferentes con resultados inconsistentes
- Los portfolios se personalizan manualmente para cada reunion, duplicando esfuerzo
- No existe un proceso estandarizado: se busca en LinkedIn, prensa, pagina web, todo manual

### Valor esperado

- **Reduccion de tiempo**: de 2-5 horas a minutos por account planning
- **Consistencia**: misma estructura de 15 puntos para todos los comerciales
- **Reutilizacion**: planes aprobados alimentan el RAG para mejorar futuras investigaciones
- **Personalizacion automatica**: portfolio adaptado al interlocutor (CEO, CTO, CFO)
- **Memoria organizacional**: la informacion no se pierde cuando un comercial cambia de cuenta
- **Mejor tasa de conversion**: propuestas mas informadas y estrategicas

---

## Contexto

- El equipo comercial actualmente usa IA de forma manual con prompts propios para investigar clientes
- Existe un portafolio base de ~25 slides con ~12 casos de exito maquetados y un centro de experiencias con mas casos
- La investigacion se alimenta de: pagina web del cliente, LinkedIn (perfiles y publicaciones), prensa, estados financieros, informes de gestion
- El documento final tiene una estructura de 15 puntos definida por el equipo comercial
- Los planes se guardan hoy en SharePoint por carpetas de cliente
- La herramienta Lusha se usa para obtener emails y telefonos de contactos
- El MVP se despliega localmente sin infraestructura cloud
- El equipo usa .NET como stack principal con Semantic Kernel para agentes IA

---

## Entidades

| Entidad | Descripcion | Atributos clave |
|---------|------------|----------------|
| Client | Cliente o prospecto a investigar | clientId, name, industry, country, language, linkedinUrl, description, createdAt |
| Investigation | Sesion de investigacion sobre un cliente | investigationId, clientId, type (QUICK/DEEP), status, userQuery, startedAt, completedAt |
| Finding | Hallazgo individual descubierto durante la investigacion | findingId, investigationId, source, sourceUrl, date, confidence, content, type, isRelevant |
| Stakeholder | Persona clave identificada en la organizacion del cliente | stakeholderId, investigationId, name, role, relevance, linkedinUrl, notes |
| Opportunity | Oportunidad comercial detectada y cruzada con servicios de Tech&Solve | opportunityId, investigationId, title, description, horizon, matchedServices, priority |
| Portfolio | Documento de account planning generado y personalizable | portfolioId, investigationId, targetRole, language, sections[], status, approvedAt |
| KnowledgeDocument | Documento en la base de conocimiento RAG | documentId, type, title, content, tags, industry, indexedAt |
| User | Usuario del sistema (comercial/BDO) | userId, email, name, avatarUrl, oauthProvider |

---

## Reglas de negocio

| ID | Regla | Descripcion | Justificacion |
|----|-------|------------|---------------|
| BR-001 | Busqueda rapida primero | Toda investigacion inicia con una busqueda rapida (~60 segundos) que muestra un resumen para confirmar que el cliente es el correcto antes de lanzar la busqueda profunda | Evitar gastar recursos en investigar al cliente equivocado |
| BR-002 | Busqueda profunda asincrona | La busqueda profunda (2-5 minutos) corre en background sin bloquear al comercial. Si cierra el navegador y vuelve, puede retomar desde donde estaba | El comercial no puede quedarse esperando 5 minutos — necesita seguir trabajando |
| BR-003 | Atribucion de fuentes obligatoria | Todo hallazgo debe incluir la fuente (nombre, URL, fecha) de donde se obtuvo la informacion | El comercial debe poder verificar y validar la informacion antes de usarla |
| BR-004 | Portfolio adaptado al interlocutor | El portfolio se adapta automaticamente segun el rol del interlocutor: CEO recibe vision estrategica, CTO recibe soluciones tecnicas, CFO recibe ROI y eficiencia | Cada perfil de decision tiene intereses y lenguaje diferentes |
| BR-005 | Feedback loop al RAG | Los planes aprobados se ingresan automaticamente a la base de conocimiento (RAG) para mejorar futuras investigaciones | El sistema debe mejorar con cada uso — los planes exitosos son la mejor fuente de conocimiento |
| BR-006 | Fallback manual de LinkedIn | Si el sistema no puede obtener automaticamente el perfil de LinkedIn, el comercial puede pegar el texto del perfil manualmente | LinkedIn bloquea scraping/APIs — el sistema no puede depender de acceso automatico |
| BR-007 | Cambio de idioma sin re-investigar | El comercial puede cambiar el idioma del portfolio (ES/EN/PT) sin relanzar la investigacion — solo se regenera el documento | La investigacion es costosa en tiempo; el cambio de idioma es solo una transformacion del output |

---

## Politicas de negocio

- La informacion del cliente es confidencial y no se comparte fuera del sistema
- El sistema funciona sin LinkedIn (degradacion elegante) — la investigacion continua con web y RAG
- El portfolio siempre requiere revision humana antes de aprobarse — la IA genera, el comercial valida
- La investigacion persiste: si el comercial cierra el navegador, la busqueda sigue corriendo y puede retomar
- En MVP no hay diferenciacion de roles — todos los usuarios autenticados tienen el mismo acceso
- La autenticacion es via OAuth con GitHub en MVP
- El procesamiento pesado es completamente asincrono — el sistema nunca bloquea al usuario
- El MVP soporta 10 usuarios concurrentes y 10-20 investigaciones diarias

---

## Validaciones

| Validacion | Condicion | Error si falla |
|-----------|----------|---------------|
| Cliente existe | El clientId debe corresponder a un cliente registrado | CLIENT_NOT_FOUND |
| Investigacion completada | Para generar portfolio, la investigacion debe estar en estado COMPLETED | INVESTIGATION_NOT_COMPLETED |
| Idioma soportado | El idioma del portfolio debe ser ES, EN o PT | INVALID_LANGUAGE |
| Rol valido | El rol del interlocutor debe ser un valor reconocido (CEO, CTO, CDO, CFO, etc.) | INVALID_ROLE |
| Investigacion no duplicada | No se puede lanzar una busqueda profunda si ya hay una en progreso para el mismo cliente | INVESTIGATION_IN_PROGRESS |
| Fuentes disponibles | Al menos una fuente de datos debe estar disponible (web, LinkedIn o RAG) | NO_SOURCES_AVAILABLE |

---

## Casos de uso

### CU-01: Gestionar clientes

```
Como comercial de Tech&Solve,
quiero crear, editar y consultar clientes con su informacion basica,
para que tenga un registro centralizado de mis prospectos con historial de investigaciones.
```

**Comportamiento esperado:**

1. El comercial crea un cliente con: nombre, sector, pais, idioma, LinkedIn, descripcion
2. El sistema persiste el cliente y lo asocia al usuario
3. El comercial puede ver el historial completo de investigaciones por cliente
4. El comercial puede actualizar la informacion del cliente bajo demanda

### CU-02: Investigar un cliente

```
Como comercial de Tech&Solve,
quiero describir mi necesidad en lenguaje natural y que el sistema investigue automaticamente,
para que obtenga informacion estrategica del cliente sin buscar manualmente en multiples fuentes.
```

**Comportamiento esperado:**

1. El comercial escribe en texto libre: "Quiero preparar una reunion con Bancolombia para presentarles servicios de automatizacion"
2. El sistema hace una busqueda rapida (~60s) y muestra un resumen para confirmar el cliente
3. El comercial confirma que es el cliente correcto
4. El sistema lanza una busqueda profunda en background (2-5 min): web, LinkedIn, RAG interno
5. El frontend consulta el estado cada ~5s (polling)
6. Al completarse, muestra el dashboard de resultados

### CU-03: Revisar resultados en el dashboard

```
Como comercial de Tech&Solve,
quiero ver los hallazgos organizados en un dashboard interactivo,
para que pueda evaluar la informacion, descartar lo irrelevante y preparar mi estrategia.
```

**Comportamiento esperado:**

1. El sistema muestra: ficha del cliente, hallazgos clave, stakeholders, oportunidades, comparativa de sector, mapa dolor-valor-servicio, fuentes consultadas
2. El comercial puede marcar hallazgos como irrelevantes o agregar notas
3. Si el cliente opera en otro idioma, el sistema muestra una alerta recomendando preparar materiales en ese idioma

### CU-04: Generar portfolio personalizado

```
Como comercial de Tech&Solve,
quiero generar un documento de account planning adaptado al rol de mi interlocutor,
para que pueda presentar una propuesta estrategica personalizada en la reunion.
```

**Comportamiento esperado:**

1. El comercial define el enfoque: tipo de empresa y rol del interlocutor (CEO, CTO, CFO, etc.)
2. El sistema sugiere el enfoque mas probable basado en la investigacion
3. El PortfolioAgent genera el documento de 15 secciones adaptando tono y foco segun el rol
4. El documento se genera en texto estructurado listo para copiar y pegar en PowerPoint
5. El comercial puede editar secciones individuales antes de aprobar
6. Al aprobar, el plan se guarda y alimenta automaticamente el RAG

### CU-05: Gestionar base de conocimiento

```
Como comercial de Tech&Solve,
quiero subir documentos al sistema (casos de exito, catalogo de servicios),
para que el RAG tenga mas contexto y genere mejores recomendaciones.
```

**Comportamiento esperado:**

1. El comercial sube un documento con tipo (caso de exito, catalogo, otro) y metadata
2. El sistema lo indexa en el vector store
3. Los planes aprobados se ingresan automaticamente
4. El RAG usa estos documentos para enriquecer futuras investigaciones

---

## Criterios de aceptacion del dominio

- [ ] Un comercial puede crear un cliente y lanzar una investigacion completa en menos de 2 minutos de interaccion activa
- [ ] La busqueda rapida retorna un resumen util en menos de 60 segundos
- [ ] La busqueda profunda completa en menos de 5 minutos con hallazgos de multiples fuentes
- [ ] El portfolio generado tiene las 15 secciones con contenido relevante y personalizado al rol
- [ ] El comercial puede editar secciones individuales del portfolio antes de aprobar
- [ ] Los planes aprobados aparecen en el RAG y mejoran investigaciones posteriores
- [ ] El sistema funciona sin LinkedIn (degrada a web + RAG)
- [ ] El cambio de idioma regenera el portfolio sin relanzar la investigacion
- [ ] El sistema soporta 10 usuarios concurrentes sin degradacion

---

## Restricciones y consideraciones

| Tipo | Descripcion |
|------|------------|
| **Limitaciones de negocio** | MVP enfocado en el equipo comercial interno de Tech&Solve. Sin acceso de clientes externos. Sin roles diferenciados. Maximo 10 usuarios concurrentes, 10-20 investigaciones/dia |
| **Integraciones necesarias** | Web search (portales confiables configurables), LinkedIn (con fallback manual), OAuth GitHub |
| **Consideraciones de seguridad** | OAuth GitHub para autenticacion. Informacion de clientes es confidencial. No enviar datos sensibles del prospecto a la IA sin sanitizar. Sin PII de personas sin consentimiento |
| **Consideraciones de rendimiento** | Quick search < 60s. Deep search < 5 min (async). Portfolio generation < 30s. Polling cada 5s. Todo el procesamiento pesado es asincrono |

---

## Glosario del dominio

| Termino | Definicion |
|---------|-----------|
| Account Planning | Documento estrategico de perfilamiento de un cliente potencial para preparar reuniones comerciales |
| BDO | Business Development Officer — comercial que usa el sistema para preparar reuniones |
| Portfolio | Documento final personalizado de 15 secciones, listo para copiar a PowerPoint y presentar |
| RAG | Retrieval Augmented Generation — base de conocimiento interna que enriquece las respuestas de la IA con documentos propios (casos de exito, catalogo, planes aprobados) |
| Finding / Hallazgo | Dato relevante descubierto durante una investigacion, con fuente y nivel de confianza |
| Quick Search | Busqueda rapida (~60 segundos) para confirmar la identidad del cliente antes de invertir en busqueda profunda |
| Deep Search | Busqueda profunda asincrona (2-5 minutos) en web, LinkedIn y RAG que genera hallazgos completos |
| Pain-Value-Service Map | Tabla de tres columnas (dolor del cliente, valor que aporta Tech, servicio del portafolio) lista para usar en la presentacion |
| Semantic Kernel | Framework de Microsoft para orquestar agentes de IA en .NET |
| Vector Store | Base de datos vectorial donde se indexan los documentos del RAG para busqueda semantica |

---

## Diseno / Mockups

Ver diagrama de arquitectura MVP vs Evolutivo en `context/imagen.png`:
- **MVP (verde oscuro):** Seguridad OAuth, entrada del usuario, busqueda web + LinkedIn + RAG, analisis IA, human in the loop, documento de texto
- **Evolutivo V2 (verde claro):** Comparativos de sector, presentacion PPT automatica, notificaciones en tiempo real
- **Evolutivo V3:** Analitica de portafolio, scoring por segmento

---

## Referencias

- `context/scope.md` — Definicion de alcance del MVP
- `context/Promejemplo.txt` — Prompts reales del equipo BDO (estructura de 15 puntos)
- `context/TranscriptNegocio.txt` — Reunion con equipo comercial (David Linares, Yosheleen)
- `context/TranscriptSesiónPresentaciónAndres.txt` — Sesion tecnica con Hernan Agudelo
- `context/imagen.png` — Diagrama de arquitectura MVP vs Evolutivo
- ADR-001 (Ports & Adapters), ADR-004 (Dependency Management), ADR-005 (Testing Strategy)

---

## Especificaciones relacionadas

| Spec ID | Nivel | Version | Descripcion |
|---------|-------|---------|------------|
| SPEC-L2-manage-clients | L2 | 1.0.0 | Gestion de clientes (CRUD + historial de investigaciones) |
| SPEC-L2-execute-research | L2 | 1.0.0 | Flujo conversacional de investigacion (quick + deep search) |
| SPEC-L2-analyze-results | L2 | 1.0.0 | Analisis IA + dashboard interactivo de resultados |
| SPEC-L2-generate-portfolio | L2 | 1.0.0 | Generacion de portfolio personalizado por rol |
| SPEC-L2-manage-knowledge-base | L2 | 1.0.0 | Gestion de base de conocimiento RAG |

---

## Historial de cambios

| Version | Fecha | Autor | Descripcion del cambio |
|---------|-------|-------|----------------------|
| 1.0.0 | 2026-04-08 | Juan Gabriel Colorado | Version inicial (concepto de agentizacion — obsoleta) |
| 2.0.0 | 2026-04-09 | Juan Gabriel Colorado | Reescritura completa basada en nuevo contexto: sistema de perfilamiento de clientes para equipo comercial con investigacion automatizada, dashboard y generacion de portfolio |
