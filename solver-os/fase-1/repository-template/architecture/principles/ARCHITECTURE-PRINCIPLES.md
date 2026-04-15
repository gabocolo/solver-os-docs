# Principios de Arquitectura

| ID | Principio | Descripcion | Justificacion |
|----|-----------|------------|---------------|
| P-001 | Ports & Adapters | Todo modulo de dominio depende de puertos (interfaces), no de adaptadores concretos | Desacoplamiento, testabilidad, independencia de infraestructura |
| P-002 | Idempotencia obligatoria | Las operaciones criticas deben ser idempotentes con idempotency key | Evitar duplicados en reintentos, consistencia en operaciones distribuidas |
| P-003 | Trazabilidad | correlationId en cada operacion, cadena completa spec → commit → release | Auditoria, debugging, cumplimiento regulatorio |
| P-004 | Dependencias autorizadas | Solo las dependencias listadas en la spec pueden usarse. La IA no inventa dependencias | Evitar alucinaciones, control de acoplamiento |
| P-005 | Privacy by Design | Proteccion de datos desde la especificacion, clasificacion obligatoria en spec L1 | Cumplimiento normativo (Ley 1581, GDPR), prevencion de fugas |
| P-006 | Contexto minimo | Solo pasar a la IA el contexto necesario para la tarea, no todo el repositorio | Optimizacion de tokens, reduccion de alucinaciones |
| P-007 | Diagramas en Mermaid | Todos los diagramas en Mermaid, no ASCII ni imagenes | Optimizacion de tokens, versionable, renderizable en GitHub/Azure DevOps |
