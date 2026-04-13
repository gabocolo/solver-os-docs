# Estandares de Seguridad

Lineamientos de seguridad aplicables a todo codigo, con enfasis en los riesgos adicionales de la generacion asistida por IA.

---

## Pruebas de seguridad automatizadas

### SAST (Static Application Security Testing)
- **Obligatorio** en el pipeline (Gate 2)
- Analiza el codigo fuente en busca de vulnerabilidades
- Hallazgos criticos y altos **bloquean** el merge
- Hallazgos medios y bajos deben documentarse y tener plan de remediacion
- Herramienta: {{definir herramienta SAST del proyecto}}

### SCA (Software Composition Analysis)
- **Obligatorio** en el pipeline (Gate 2)
- Analiza dependencias y librerias de terceros en busca de vulnerabilidades conocidas (CVEs)
- Dependencias con vulnerabilidades criticas **bloquean** el merge
- Mantener dependencias actualizadas como practica continua
- Herramienta: {{definir herramienta SCA del proyecto}}

### DAST (Dynamic Application Security Testing)
- **Recomendado** post-deploy en ambiente de staging
- Pruebas de vulnerabilidades sobre el artefacto desplegado
- Requiere un ambiente disponible para ejecutar
- Herramienta: {{definir herramienta DAST del proyecto si aplica}}

---

## Gestion de dependencias

Segun ADR-004, solo las dependencias listadas en la spec L2 son autorizadas.

- **No agregar dependencias sin actualizar la spec** (crear L3 si es necesario)
- **No usar dependencias sugeridas por la IA** que no esten en la spec
- Verificar que cada dependencia:
  - Tiene mantenimiento activo
  - No tiene CVEs criticos abiertos
  - Es compatible con la licencia del proyecto

---

## Gestion de secretos

- **NUNCA** incluir secretos en el codigo fuente (passwords, API keys, tokens, certificados)
- Usar variables de entorno o gestion de secretos (Vault, AWS Secrets Manager, Azure Key Vault)
- Los archivos `.env` estan en `.gitignore`
- Si la IA genera codigo con credenciales hardcoded, **rechazar inmediatamente**

---

## Validacion de entradas

- Toda entrada del usuario o de APIs externas debe ser validada
- Validar tipo, formato, longitud y rango
- Sanitizar entradas antes de usar en queries, templates o comandos
- Protecciones especificas contra:
  - **SQL Injection**: usar parametros preparados, nunca concatenar
  - **XSS**: escapar output, usar Content Security Policy
  - **Command Injection**: nunca ejecutar comandos con input del usuario sin sanitizar
  - **Path Traversal**: validar y normalizar paths

---

## Riesgos especificos de generacion con IA

### Alucinacion de dependencias
La IA puede sugerir librerias que no existen o que son maliciosas (typosquatting). Siempre verificar que la dependencia:
- Existe en el registry oficial
- Tiene el nombre correcto (no una variante similar)
- Tiene historial de mantenimiento

### Prompt injection
Si la aplicacion procesa input de usuarios que se pasa a un LLM:
- Sanitizar el input del usuario antes de incluirlo en el prompt
- Separar claramente las instrucciones del sistema del input del usuario
- No confiar en que el LLM respetara las instrucciones si el input es malicioso

### Codigo inseguro generado por IA
La IA puede generar patrones inseguros que parecen funcionales. En Gate 3, verificar:
- [ ] No hay credenciales hardcoded
- [ ] Las queries usan parametros preparados
- [ ] El input del usuario esta validado
- [ ] No hay ejecucion de comandos con input no sanitizado
- [ ] Los errores no exponen informacion interna

---

## Prevencion de fuga de datos (DLP)

Segun ADR-006, todo sistema debe implementar controles de prevencion de fuga de datos.

### Clasificacion de datos en especificaciones

Toda entidad en una spec L1 debe clasificar sus atributos:

| Nivel | Tratamiento obligatorio |
|-------|------------------------|
| **Publico** | Sin restriccion |
| **Interno** | No exponer en APIs publicas |
| **Confidencial** | Cifrado en reposo, acceso restringido |
| **Sensible (PII)** | Cifrado, masking en logs, anonimizacion en ambientes no productivos |

### Datos sensibles en prompts y generacion con IA

- **NUNCA** incluir datos reales de tipo Sensible o Confidencial en prompts a LLMs
- Usar **datos sinteticos** para contexto, ejemplos y fixtures de test
- Antes de enviar contexto a la IA, validar que no contenga: emails reales, numeros de documento, tokens activos, datos financieros personales
- Si la aplicacion usa LLMs en runtime: implementar filtros DLP pre-prompt y post-respuesta (ver SK-007)

### Deteccion automatizada de PII en codigo

En Gate 2, el pipeline debe incluir deteccion de:
- Emails hardcoded (regex: patron de email en strings literales)
- Numeros de documento o identificacion en fixtures o constantes
- Datos financieros (numeros de tarjeta, cuentas) en cualquier archivo
- Tokens o API keys en archivos que no sean de configuracion de secretos

Herramientas recomendadas:
- `detect-secrets` para deteccion de secretos
- `presidio` (Microsoft) o regex custom para deteccion de PII
- GitHub Advanced Security (secret scanning + code scanning)

### Datos en ambientes no productivos

- Ambientes de desarrollo y staging deben usar datos **anonimizados o sinteticos**
- Prohibido copiar datos de produccion a ambientes inferiores sin anonimizacion
- Los dumps de base de datos para testing deben pasar por proceso de masking

### Respuesta ante fuga de datos

| Tipo de fuga | Accion inmediata | Tiempo de respuesta |
|-------------|-----------------|-------------------|
| PII en logs de produccion | Purgar logs afectados, notificar al equipo de seguridad | < 4h |
| Secretos en repositorio | Rotar secretos inmediatamente, limpiar historial git | < 1h |
| PII en respuesta de API | Hotfix para aplicar masking, notificar al DPO | < 4h |
| Datos reales en prompt a LLM externo | Evaluar exposicion con proveedor, documentar incidente | < 24h |

---

## Autenticacion y autorizacion

- Toda operacion debe validar la autenticacion del usuario
- Principio de minimo privilegio: solo los permisos necesarios
- Los tokens deben tener expiracion
- Las sesiones deben poder ser revocadas
- Patron recomendado: {{definir patron de auth del proyecto, ej: JWT + refresh token}}

---

## Respuesta a vulnerabilidades

| Severidad | Tiempo de respuesta | Accion |
|-----------|-------------------|--------|
| **Critica** | Inmediato (< 24h) | Bloquear merge, hotfix, notificar equipo |
| **Alta** | < 3 dias | Planificar fix en sprint actual |
| **Media** | < 2 semanas | Incluir en backlog priorizado |
| **Baja** | < 1 mes | Registrar y planificar |

---

{{Agregar configuracion especifica de herramientas SAST/SCA del proyecto aqui}}

---

## Normativas de referencia

| Normativa | Requisitos clave para el desarrollo |
|-----------|-----------------------------------|
| **Ley 1581 de 2012** (Colombia) | Consentimiento informado, finalidad limitada, circulacion restringida, medidas de seguridad tecnicas |
| **GDPR** | Privacy by Design, Privacy by Default, minimizacion de datos, derecho al olvido, notificacion de brechas |
| **ISO 27001** | Clasificacion de activos de informacion, control de acceso, gestion de incidentes |

---

*Referencia: ADR-004 (Dependency Management), ADR-006 (Data Classification & DLP), Framework AI-Driven Engineering v1.0*
