# Domain Service Standard

Version: 1.0

Status: APPROVED

---

# Objetivo

Este documento define el estándar oficial para modelar Domain Services dentro del Dominio Técnico de CollectionHub.

Los Domain Services representan comportamiento del dominio que no pertenece de forma natural a una Entity, Value Object o Aggregate.

Su objetivo es encapsular procesos de negocio complejos manteniendo un modelo de dominio cohesionado y expresivo.

No representan servicios técnicos.

No representan casos de uso.

No representan servicios de aplicación.

Representan comportamiento del negocio.

---

# Definición

Un Domain Service encapsula una operación del dominio que requiere coordinar varios elementos del modelo sin romper sus responsabilidades.

El servicio existe porque ningún objeto individual posee toda la información o responsabilidad necesaria para ejecutar correctamente dicha operación.

---

# Principios

Todo Domain Service debe cumplir los siguientes principios.

- Representa comportamiento del negocio.
- No posee estado persistente.
- Es conceptualmente stateless.
- Utiliza el lenguaje ubicuo.
- Coordina, no domina.
- Respeta los límites de los Aggregates.
- Es independiente de la infraestructura.
- Es independiente de la persistencia.

---

# Cuándo utilizar un Domain Service

Debe existir un Domain Service cuando:

- Una operación involucra varios Aggregates.
- Ninguna Entity puede asumir la responsabilidad completa.
- El comportamiento pertenece claramente al dominio.
- La lógica sería artificial si se colocara dentro de una Entity.

---

# Cuándo NO utilizar un Domain Service

No utilizar un Domain Service cuando:

- La lógica pertenece claramente a una Entity.
- La lógica pertenece a un Value Object.
- Se trata de un caso de uso de la aplicación.
- Solo realiza operaciones CRUD.
- Únicamente encapsula llamadas técnicas.

---

# Estructura Obligatoria

Todo Domain Service deberá documentarse utilizando exactamente la siguiente estructura.

```text
# Domain Service Name

## Purpose

## Responsibilities

## Collaborators

## Inputs

## Outputs

## Behaviors

## Business Rules

## Generated Events

## Dependencies

## Examples

## Future Evolution
```

---

# 1. Purpose

Explica qué proceso del negocio representa el servicio.

Debe responder únicamente a una pregunta.

> ¿Qué proceso del dominio coordina este servicio?

---

# 2. Responsibilities

Lista únicamente responsabilidades relacionadas con el negocio.

Si aparecen responsabilidades técnicas, el diseño debe revisarse.

---

# 3. Collaborators

Indica qué Aggregates, Entities o Value Objects participan en el proceso.

---

# 4. Inputs

Describe qué información necesita el servicio para realizar su trabajo.

No hablar de DTOs ni contratos de API.

---

# 5. Outputs

Describe el resultado conceptual producido.

Puede ser una decisión, una recomendación, una evaluación o un evento.

---

# 6. Behaviors

Lista las operaciones principales del servicio utilizando lenguaje ubicuo.

Ejemplos:

- AnalyzePortfolio()
- GenerateRecommendation()
- EvaluateOpportunity()
- AllocateBudget()

---

# 7. Business Rules

Documenta las reglas del negocio que aplica el servicio.

Las reglas deben proceder del Dominio Conceptual y de las Policies.

---

# 8. Generated Events

Enumera los Domain Events que el servicio puede provocar.

Siempre representan hechos ya ocurridos.

---

# 9. Dependencies

Solo dependencias conceptuales.

Nunca:

- Entity Framework
- SQL Server
- ASP.NET
- HTTP
- JSON

---

# 10. Examples

Describe ejemplos reales del dominio.

Nunca ejemplos de código.

---

# 11. Future Evolution

Describe posibles ampliaciones futuras coherentes con el modelo.

---

# Calidad Esperada

Antes de aprobar un Domain Service deberán responderse afirmativamente las siguientes preguntas.

□ ¿Representa un proceso del negocio?

□ ¿No pertenece claramente a una Entity?

□ ¿No pertenece claramente a un Value Object?

□ ¿Coordina varios elementos del dominio?

□ ¿Respeta los límites de los Aggregates?

□ ¿No contiene lógica técnica?

□ ¿Puede entenderse sin leer código?

□ ¿Respeta la Constitución?

□ ¿Respeta los Core Principles?

□ ¿Utiliza el lenguaje ubicuo?

---

# Anti-Patrones

Un Domain Service nunca debe convertirse en:

- Un Application Service.
- Un CRUD Service.
- Un Helper.
- Un Utility.
- Un Facade técnico.
- Un contenedor de lógica que debería vivir en las entidades.

Si el servicio solo existe para "hacer cosas", probablemente está mal modelado.

---

# Candidatos Iniciales en CollectionHub

Los siguientes servicios se consideran candidatos naturales:

- Recommendation Service
- Portfolio Analysis Service
- Opportunity Detection Service
- Budget Allocation Service
- Market Valuation Service

La incorporación de nuevos servicios deberá justificarse por necesidades reales del dominio.

---

# Regla de Oro

Un Domain Service solo debe existir cuando ninguna Entity, Value Object o Aggregate pueda asumir legítimamente la responsabilidad del comportamiento.

Si existe un lugar natural para esa lógica, el servicio no debe crearse.

---

# Objetivo Final

Construir un dominio donde los Domain Services sean pocos, altamente cohesionados y dedicados exclusivamente a coordinar procesos complejos del negocio, manteniendo el conocimiento distribuido en el lugar donde realmente pertenece.