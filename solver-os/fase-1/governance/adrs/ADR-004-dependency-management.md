# ADR-004: Solo dependencias explicitamente autorizadas en la especificacion

| Campo | Valor |
|-------|-------|
| **ADR ID** | ADR-004 |
| **Estado** | ACCEPTED |
| **Fecha** | 2026-04-07 |
| **Autor** | Equipo de Arquitectura |
| **PR** | Pendiente |
| **Reemplaza** | N/A |
| **Reemplazado por** | N/A |

## Contexto

Uno de los riesgos mas criticos al generar codigo con IA es la **alucinacion de dependencias**: la IA puede inventar servicios, librerias o integraciones que no existen o no estan aprobadas. Esto genera codigo que compila pero que introduce dependencias no autorizadas, potencialmente inseguras o incompatibles con la arquitectura.

## Decision

Solo las dependencias explicitamente listadas en la especificacion L2 (seccion "Dependencias permitidas") pueden ser utilizadas en la implementacion. Cualquier dependencia no listada es considerada **no autorizada** y debe ser rechazada en revision.

El prompt estructurado debe incluir la instruccion: "No inventes servicios, reglas ni dependencias fuera del spec."

## Alternativas evaluadas

| Opcion | Descripcion | Pros | Contras |
|--------|------------|------|---------|
| **Dependencias explicitas en spec** (elegida) | Solo se pueden usar las dependencias listadas en la especificacion L2 | Control total, previene alucinaciones, trazable | Requiere actualizar spec si se necesita nueva dependencia |
| **Allowlist a nivel organizacional** | Lista global de dependencias permitidas, sin control por spec | Menor overhead por spec, cobertura amplia | No controla que dependencias son relevantes para cada caso, demasiado permisivo |
| **Uso libre con revision post-generacion** | La IA usa lo que considere necesario, se revisa despues | Sin restricciones a la generacion | Alto riesgo de alucinaciones, mayor carga de revision, dificil de detectar dependencias fantasma |

## Consecuencias

### Positivas
- Elimina el riesgo de alucinacion de dependencias
- Cada dependencia es trazable a una decision explicita
- Facilita el analisis SCA (se sabe exactamente que librerias se deben usar)
- La IA genera codigo rapido pero dentro de limites controlados

### Negativas
- Si se necesita una nueva dependencia, hay que actualizar la spec (L3)
- Puede ralentizar levemente el inicio de la generacion
- Requiere que las specs L2 sean completas en su seccion de dependencias

## Cumplimiento

- En especificaciones L2: la seccion "Dependencias permitidas" es obligatoria
- En el prompt: incluir la regla explicita de no inventar dependencias
- En Gate 2: SCA valida que solo se usan librerias conocidas
- En Gate 3: el reviewer verifica que las imports coinciden con las dependencias autorizadas

## ADRs relacionados

- ADR-001: Ports and Adapters — las dependencias deben ser puertos, no implementaciones
- ADR-005: Testing Strategy — los tests deben mockear solo dependencias autorizadas
