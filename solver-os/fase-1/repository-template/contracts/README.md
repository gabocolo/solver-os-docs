# /contracts — Contratos API y Eventos

Contiene los contratos formales que definen la comunicacion entre componentes. Los contratos son el **punto de convergencia** entre backend y frontend.

## Estructura

| Carpeta | Tipo | Proposito |
|---------|------|-----------|
| openapi/ | REST API | Contratos OpenAPI 3.1 YAML para cada servicio |
| events/ | Eventos de dominio | Schemas JSON de eventos publicados via EventBus |

## Contratos OpenAPI

Se generan con **SK-003** desde la seccion "Contrato tecnico" de cada spec L2.

| Archivo | Spec L2 de origen | Descripcion |
|---------|-------------------|-------------|
| {{servicio}}-api.yaml | {{SPEC-L2-xxx}} | {{descripcion del servicio}} |

> Template base: OPENAPI-CONTRACT-TEMPLATE.yaml

## Eventos de dominio

Cada evento definido en la spec L1 tiene su schema tecnico aqui:

| Archivo | Evento | Publicador | Consumidores |
|---------|--------|-----------|-------------|
| {{NombreEvento}}.json | {{descripcion}} | {{servicio}} | {{consumidores}} |

## Reglas

- Todo contrato debe ser generado desde una spec L2 aprobada (nunca inventado)
- Los contratos deben existir en **ambos repositorios** (backend y frontend)
- Cambios en contratos requieren versionado y PR
- Incluir extension `x-data-classification` para campos PII (ADR-006)
- Usar datos sinteticos en los ejemplos del contrato
