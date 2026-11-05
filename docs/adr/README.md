# Decisiones de arquitectura (ADR)

Un archivo por decisión, numerado: `NNNN-titulo-corto.md`. Un ADR no se reescribe: si una
decisión cambia, se escribe uno nuevo que la sustituye y se marca el antiguo como
"Sustituido por NNNN".

| Nº | Decisión | Estado |
|---|---|---|
| [0001](0001-idp-central-oidc.md) | IdP central con OIDC para todas las apps | Aceptado |

## Plantilla

```markdown
# NNNN. Título

- Fecha: AAAA-MM-DD
- Estado: Propuesto | Aceptado | Sustituido por NNNN

## Contexto
Qué problema hay y qué condiciona la decisión.

## Decisión
Qué se hace.

## Alternativas descartadas
Qué otras opciones había y por qué no.

## Consecuencias
Qué se gana, qué se pierde y qué obliga a hacer.
```
