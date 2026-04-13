# Especificacion de Sistema (Nivel 2)

| Campo | Valor |
|-------|-------|
| **Spec ID** | SPEC-L2-manage-knowledge-base |
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

Modulo de gestion de la base de conocimiento interna (RAG). Permite subir documentos manualmente (casos de exito, catalogo de servicios, otros) e ingesta automaticamente los planes aprobados. El vector store indexa los documentos para que el Research Engine y el Portfolio Agent los consulten semanticamente.

---

## Sistema

**Nombre:** Knowledge Base Management

## Caso de uso

**Nombre:** ManageKnowledgeBase

**Descripcion:** Los comerciales pueden subir documentos relevantes (casos de exito, catalogo de servicios, informes) que se indexan en un vector store local. Los portfolios aprobados se ingresan automaticamente. El sistema permite consultar, listar y eliminar documentos de la base de conocimiento.

---

## Contexto

- Tech&Solve tiene un centro de experiencias con casos de exito (algunos maquetados, otros solo con datos basicos)
- Existe un portafolio base de ~25 slides con ~12 casos de exito maquetados
- Los documentos relevantes hoy estan en SharePoint sin indexacion semantica
- El RAG es la base que permite al sistema generar oportunidades informadas y portfolios contextualizados
- En MVP se usa un vector store local (sin cloud)
- Los portfolios aprobados son la fuente mas valiosa para el RAG — representan conocimiento validado por el equipo comercial

---

## Contrato tecnico

### Input — UploadDocument

| Campo | Tipo | Requerido | Descripcion |
|-------|------|-----------|------------|
| documentType | string | Si | CASE_STUDY, SERVICE_CATALOG, APPROVED_PLAN, FINANCIAL_REPORT, OTHER |
| title | string | Si | Titulo del documento |
| content | string | Si | Contenido del documento (texto plano o markdown) |
| description | string | No | Descripcion breve del documento |
| tags | string[] | No | Etiquetas para clasificacion (ej: "seguros", "automatizacion", "IA") |
| industry | string | No | Industria a la que aplica (ej: "Seguros", "Banca") |

### Input — ListDocuments

| Campo | Tipo | Requerido | Descripcion |
|-------|------|-----------|------------|
| documentType | string | No | Filtrar por tipo |
| industry | string | No | Filtrar por industria |
| search | string | No | Busqueda por titulo o contenido |
| page | integer | No | Pagina (default: 1) |
| pageSize | integer | No | Tamano de pagina (default: 20) |

### Input — DeleteDocument

| Campo | Tipo | Requerido | Descripcion |
|-------|------|-----------|------------|
| documentId | string (UUID) | Si | ID del documento a eliminar |

### Output — DocumentResponse

| Campo | Tipo | Descripcion |
|-------|------|------------|
| documentId | string (UUID) | ID unico del documento |
| documentType | string | Tipo del documento |
| title | string | Titulo |
| description | string | Descripcion |
| tags | string[] | Etiquetas |
| industry | string | Industria |
| status | string | INDEXING, INDEXED, FAILED |
| indexedAt | string (ISO 8601) | Timestamp de indexacion (null si INDEXING o FAILED) |
| createdAt | string (ISO 8601) | Fecha de creacion |

### Output — DocumentListResponse

| Campo | Tipo | Descripcion |
|-------|------|------------|
| documents | DocumentResponse[] | Lista de documentos |
| total | integer | Total de documentos que cumplen el filtro |
| page | integer | Pagina actual |
| pageSize | integer | Tamano de pagina |

---

## Comportamiento esperado

### UploadDocument
1. El sistema valida los datos de entrada (titulo requerido, contenido no vacio)
2. Crea el registro del documento con status INDEXING
3. Lanza la indexacion asincrona en el vector store:
   a. Divide el contenido en chunks semanticos
   b. Genera embeddings para cada chunk
   c. Almacena en el vector store con metadata (tipo, industria, tags)
4. Al completar: status pasa a INDEXED
5. Si falla: status pasa a FAILED con detalle del error
6. Publica evento DocumentIndexed

### Ingesta automatica de portfolios aprobados
1. Al recibir evento PortfolioApproved:
   a. Recupera el contenido completo del portfolio
   b. Crea un documento de tipo APPROVED_PLAN
   c. Agrega como tags: industria del cliente, rol del interlocutor
   d. Ejecuta el mismo flujo de indexacion

### ListDocuments
1. Aplica filtros si se proporcionan (tipo, industria, busqueda)
2. Retorna lista paginada ordenada por fecha de creacion desc

### DeleteDocument
1. Valida que el documento existe
2. Elimina del vector store (embeddings + metadata)
3. Elimina el registro
4. Publica evento DocumentDeleted

---

## Tipos de error

| Codigo de error | HTTP Status | Descripcion | Condicion de disparo |
|----------------|-------------|------------|---------------------|
| DOCUMENT_NOT_FOUND | 404 | El documento no existe | documentId no encontrado |
| DOCUMENT_TOO_LARGE | 400 | El contenido excede el tamano maximo | content > 100KB |
| INVALID_DOCUMENT_TYPE | 400 | Tipo de documento no reconocido | documentType no esta en la lista |
| INDEXING_FAILED | 500 | Error al indexar en el vector store | Vector store no responde o error de procesamiento |
| EMPTY_CONTENT | 400 | El contenido esta vacio | content vacio o solo espacios |

---

## Dependencias permitidas

> **REGLA (ADR-004):** Solo estas dependencias pueden ser usadas en la implementacion.

| Dependencia | Proposito | Tipo de interfaz |
|------------|----------|-----------------|
| DocumentRepository | Persistir metadata de documentos | Puerto (IDocumentRepository) |
| VectorStore | Indexar, buscar y eliminar embeddings | Puerto (IVectorStore) |
| EmbeddingGenerator | Generar embeddings de texto para el vector store | Puerto (IEmbeddingGenerator) |
| PortfolioRepository | Obtener portfolio aprobado para ingesta automatica | Puerto (IPortfolioRepository) |
| EventBus | Escuchar PortfolioApproved, publicar DocumentIndexed/DocumentDeleted | Puerto (IEventPublisher) |

---

## Requisitos de ejecucion

| Requisito | Valor | Notas |
|-----------|-------|-------|
| **Latencia maxima (upload)** | 1000 ms (p95) | Solo persistir metadata — indexacion es async |
| **Latencia maxima (indexacion)** | 30000 ms (30s) | Generacion de embeddings es costosa |
| **Latencia maxima (list/delete)** | 500 ms (p95) | Operaciones CRUD |
| **Idempotencia** | No requerida | |
| **Atomicidad** | Eventual consistency | Metadata y embeddings se persisten en fases |
| **Observabilidad** | Ver abajo | |

### Observabilidad requerida

- **Logs:** Documento subido (INFO), indexacion completada (INFO), indexacion fallida (ERROR), portfolio aprobado ingestado (INFO), documento eliminado (INFO)
- **Trazas:** span por upload + indexacion, span por generacion de embeddings
- **Metricas:** documents_total (por tipo), indexation_duration_ms, documents_indexed_total, rag_search_hit_rate
- **Alertas:** indexing failure rate > 10%, vector store no disponible

---

## Restricciones de arquitectura

| ADR | Titulo | Impacto en esta spec |
|-----|--------|---------------------|
| ADR-001 | Ports & Adapters | ManageKnowledgeBase depende de 5 puertos. VectorStore es un puerto — se puede cambiar la implementacion (FAISS, Chroma, etc.) sin afectar el core |
| ADR-004 | Dependency Management | Solo las 5 dependencias listadas |
| ADR-005 | Testing Strategy | Tests GWT antes de implementar. Mock del VectorStore en tests |

---

## Restricciones y consideraciones

| Tipo | Descripcion |
|------|------------|
| **Limitaciones tecnicas** | Vector store local en MVP (no managed service). Limite de 100KB por documento |
| **Integraciones necesarias** | Vector store local (FAISS, Chroma o similar), modelo de embeddings |
| **Consideraciones de seguridad** | No indexar documentos con PII de terceros. Validar que el contenido subido no contenga malware o scripts |
| **Consideraciones de rendimiento** | < 200 documentos en MVP. Indexacion async no bloquea al usuario |

---

## Criterios de aceptacion

- [ ] Se puede subir un caso de exito y queda indexado en el vector store
- [ ] Se puede subir el catalogo de servicios y queda disponible para busquedas
- [ ] Los portfolios aprobados se ingresan automaticamente sin accion del usuario
- [ ] Se puede buscar documentos por tipo, industria o texto libre
- [ ] Se puede eliminar un documento y se remueve del vector store
- [ ] La indexacion es asincrona — el usuario no espera a que termine
- [ ] El status del documento refleja el estado real de la indexacion

---

## Escenarios de prueba (GWT)

> **REGLA (ADR-005):** Estos escenarios deben traducirse en tests ANTES de generar la implementacion.

| ID | Given (Dado que) | When (Cuando) | Then (Entonces) | Prioridad |
|----|-----------------|--------------|-----------------|-----------|
| TS-001 | No hay documentos en el sistema | Se sube un caso de exito con titulo, contenido, tags e industria "Seguros" | Documento creado con status INDEXING, luego pasa a INDEXED | Alta |
| TS-002 | Un portfolio para "Seguros Bolivar" fue aprobado | El evento PortfolioApproved se dispara | Se crea automaticamente un documento tipo APPROVED_PLAN con el contenido del portfolio, tags incluyen industria y rol | Alta |
| TS-003 | Existen 10 documentos: 3 de tipo CASE_STUDY y 7 de tipo APPROVED_PLAN | Se busca con documentType=CASE_STUDY | Retorna solo los 3 casos de exito | Media |
| TS-004 | Existe un documento indexado | Se elimina el documento | El documento se remueve de la BD y del vector store, status ya no existe | Media |
| TS-005 | Se intenta subir un documento con contenido > 100KB | Se ejecuta el upload | Error DOCUMENT_TOO_LARGE | Media |
| TS-006 | Se intenta subir un documento con contenido vacio | Se ejecuta el upload | Error EMPTY_CONTENT | Alta |

---

## Referencias

- SPEC-L1-account-planning v2.0.0 — Dominio (BR-005)
- SPEC-L2-generate-portfolio v1.0.0 — Genera portfolios que se ingresan al RAG
- SPEC-L2-execute-research v1.0.0 — Consume documentos del RAG durante la investigacion
- context/scope.md — Seccion "Fuentes de investigacion" (RAG)
- context/TranscriptNegocio.txt — Centro de experiencias y casos de exito existentes

---

## Especificaciones relacionadas

| Spec ID | Nivel | Version | Relacion |
|---------|-------|---------|----------|
| SPEC-L1-account-planning | L1 | 2.0.0 | Parent — dominio |
| SPEC-L2-execute-research | L2 | 1.0.0 | Consumer — consulta RAG durante investigacion |
| SPEC-L2-generate-portfolio | L2 | 1.0.0 | Producer — portfolios aprobados alimentan RAG |

---

## Historial de cambios

| Version | Fecha | Autor | Descripcion del cambio |
|---------|-------|-------|----------------------|
| 1.0.0 | 2026-04-09 | Juan Gabriel Colorado | Version inicial: gestion de base de conocimiento RAG con ingesta automatica de planes aprobados |
