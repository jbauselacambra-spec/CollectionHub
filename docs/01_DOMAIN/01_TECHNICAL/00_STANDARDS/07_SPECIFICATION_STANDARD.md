# Specification Standard

Version: 1.0

Status: APPROVED

---

# Objetivo

Este documento define el estándar oficial para modelar Specifications dentro del Dominio Técnico de CollectionHub.

Las Specifications representan condiciones del negocio reutilizables que permiten evaluar si una determinada situación cumple los criterios definidos por el dominio.

No representan consultas técnicas.

No representan filtros de base de datos.

Representan conocimiento reutilizable del negocio.

---

# Definición

Una Specification encapsula una condición del dominio.

Su propósito es responder a una única pregunta:

> ¿Se cumple esta condición del negocio?

Una Specification nunca ejecuta acciones.

Únicamente evalúa.

---

# Principios

Toda Specification debe cumplir los siguientes principios.

- Representa una condición del negocio.
- Es reutilizable.
- Es independiente de la infraestructura.
- Es independiente de la persistencia.
- Utiliza el lenguaje ubicuo.
- Es determinista.
- Es fácilmente componible.
- Puede ser explicada al Collector.

---

# Cuándo utilizar una Specification

Debe existir una Specification cuando:

- Una condición se reutiliza en distintos procesos.
- Una Policy necesita evaluar criterios.
- Un Aggregate necesita validar una condición externa.
- Una regla puede combinarse con otras reglas.

---

# Cuándo NO utilizar una Specification

No utilizar una Specification cuando:

- La regla pertenece exclusivamente a una Entity.
- La lógica modifica el estado del dominio.
- La operación representa un proceso.
- Se trata únicamente de una consulta técnica.

---

# Estructura Obligatoria

Toda Specification deberá documentarse utilizando exactamente la siguiente estructura.

```text
# Specification Name

## Purpose

## Business Question

## Evaluation Criteria

## Inputs

## Output

## Related Policies

## Related Principles

## Composition

## Examples

## Future Evolution
```

---

# 1. Purpose

¿Por qué existe esta Specification?

---

# 2. Business Question

Toda Specification responde exactamente a una única pregunta.

Ejemplos.

¿Existe presupuesto suficiente?

¿La oportunidad es estratégica?

¿La colección puede completarse?

¿El precio es competitivo?

---

# 3. Evaluation Criteria

Describe los criterios utilizados para evaluar la condición.

No describir algoritmos.

Describir reglas del negocio.

---

# 4. Inputs

Información conceptual necesaria.

Nunca DTOs.

Nunca modelos de infraestructura.

---

# 5. Output

El resultado siempre representa una evaluación conceptual.

Habitualmente:

- Cumple.
- No cumple.

Opcionalmente puede incluir una explicación.

---

# 6. Related Policies

Indica qué Policies utilizan esta Specification.

---

# 7. Related Principles

Relaciona la Specification con Foundation y Core Principles.

Toda Specification debe tener una justificación estratégica.

---

# 8. Composition

Describe con qué otras Specifications puede combinarse.

Ejemplos.

AND

OR

NOT

La composición debe mantener significado para el negocio.

---

# 9. Examples

Ejemplos reales del dominio.

Nunca ejemplos técnicos.

---

# 10. Future Evolution

Posibles ampliaciones compatibles con el dominio.

---

# Explicabilidad

Toda Specification debe poder explicar por qué se cumple o no.

CollectionHub prioriza decisiones transparentes.

La evaluación nunca debe convertirse en una "caja negra".

---

# Reutilización

Una Specification debe poder utilizarse desde:

- Policies
- Domain Services
- Aggregates
- Entities
- Recommendation Engine

Sin modificar su significado.

---

# Calidad Esperada

Antes de aprobar una Specification deberán responderse afirmativamente las siguientes preguntas.

□ ¿Representa una condición del negocio?

□ ¿Responde a una única pregunta?

□ ¿Es reutilizable?

□ ¿Es explicable?

□ ¿Es independiente de infraestructura?

□ ¿Puede componerse con otras Specifications?

□ ¿Utiliza el lenguaje ubicuo?

□ ¿Respeta Foundation?

□ ¿Respeta los Core Principles?

□ ¿Puede entenderse sin leer código?

---

# Anti-Patrones

Una Specification nunca debe convertirse en:

- Un Repository.
- Una consulta SQL.
- Un filtro de Entity Framework.
- Un caso de uso.
- Un Domain Service.
- Una validación técnica.

Si modifica el estado del dominio, no es una Specification.

---

# Candidatos Iniciales en CollectionHub

- HasAvailableBudgetSpecification
- IsStrategicOpportunitySpecification
- MeetsTargetPriceSpecification
- CanCompleteCollectionSpecification
- IsHighPriorityCollectibleSpecification
- HasHealthyLiquiditySpecification
- IsCollectorObjectiveAlignedSpecification

Cada nueva Specification deberá representar una condición real del negocio.

---

# Regla de Oro

Toda Specification debe responder únicamente a una pregunta del dominio.

Nunca debe ejecutar decisiones.

Nunca debe modificar el estado.

Solo evaluar.

---

# Objetivo Final

Construir un catálogo de condiciones reutilizables que permita expresar las reglas del negocio de forma clara, componible y explicable, facilitando que Policies, Aggregates y Domain Services tomen decisiones consistentes y alineadas con la estrategia del Collector.