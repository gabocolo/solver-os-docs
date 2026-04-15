# /security — Modelos de Amenazas y Politicas de Seguridad

Contiene toda la documentacion de seguridad del proyecto.

## Estructura

| Carpeta | Proposito |
|---------|-----------|
| threat-models/ | Modelos de amenazas por componente o flujo critico |
| policies/ | Politicas de seguridad del proyecto |
| dlp/ | Configuracion DLP: patrones, reglas de masking, filtros |

## Clasificacion de datos (ADR-006)

| Nivel | Ejemplos | Tratamiento |
|-------|----------|-------------|
| **Publico** | Nombres de producto, precios | Sin restriccion |
| **Interno** | IDs, timestamps, estados | No exponer externamente |
| **Confidencial** | Contratos, metricas financieras | Acceso restringido, cifrado en reposo |
| **Sensible (PII)** | Email, documento, telefono | Cifrado, masking en logs, anonimizacion en no-prod |

## Herramientas de validacion

| Herramienta | Tipo | Cuando se ejecuta |
|------------|------|-------------------|
| SAST | Analisis estatico de codigo | CI/CD (pre-merge) |
| SCA | Vulnerabilidades en dependencias | CI/CD (pre-merge) |
| DLP Scan | Deteccion de PII y secretos | CI/CD (pre-merge) |
| DAST | Pruebas sobre artefacto desplegado | Post-deploy |

## Politicas transversales

- **Privacy by Design**: proteccion de datos desde la especificacion
- **Privacy by Default**: comportamiento por defecto es el mas restrictivo
- **Datos sinteticos siempre**: nunca PII real en prompts, tests o fixtures
- **Masking automatico en logs**: filtro de PII obligatorio

## Respuesta ante fuga de datos

| Tipo de fuga | Accion inmediata | Tiempo |
|-------------|-----------------|--------|
| PII en logs | Purgar logs, notificar seguridad | < 4h |
| Secretos en repo | Rotar secretos, limpiar historial git | < 1h |
| PII en respuesta API | Hotfix con masking, notificar DPO | < 4h |
