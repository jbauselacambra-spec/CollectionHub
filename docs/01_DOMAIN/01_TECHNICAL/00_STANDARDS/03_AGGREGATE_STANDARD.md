# Aggregate Standard

Version: 1.0

Status: APPROVED

---

# Objetivo

Este documento define el estándar oficial para modelar Aggregates dentro del Dominio Técnico de CollectionHub.

Los Aggregates representan los límites de consistencia del dominio.

Su misión es proteger las reglas de negocio, garantizar que las invariantes nunca puedan romperse y controlar cómo evoluciona el estado del dominio.

No representan agrupaciones de clases.

No representan agrupaciones de tablas.

Representan límites transaccionales del negocio.

---

# Definición

Un Aggregate es un conjunto de objetos del dominio tratados como una única unidad de consistencia.

Todo acceso al Aggregate se realiza exclusivamente a través de su Aggregate Root.

El Aggregate Root es el único responsable de proteger las invariantes del Aggregate.

---

# Principios

Todo Aggregate debe cumplir los siguientes principios.

- Tiene exactamente un Aggregate Root.
- Protege invariantes del negocio.
- Define un límite de consistencia.
- Controla las modificaciones internas.
- Mantiene la coherencia del dominio.
- Minimiza el acoplamiento con otros Aggregates.
- Expresa el lenguaje ubicuo.
- Es independiente de la infraestructura.

---

# Cuándo utilizar un Aggregate

Debe existir un Aggregate cuando:

- Varias entidades comparten invariantes.
- Existen reglas que afectan a varios objetos simultáneamente.
- Se necesita garantizar consistencia transaccional.
- El negocio considera esos objetos una unidad lógica.

---

# Cuándo NO utilizar un Aggregate

No crear un Aggregate cuando:

- Solo agrupa datos.
- No protege reglas del negocio.
- Su único propósito es facilitar consultas.
- Se utiliza únicamente por razones técnicas.

---

# Estructura Obligatoria

Todo Aggregate deberá documentarse utilizando exactamente la siguiente estructura.

```text
# Aggregate Name

## Purpose

## Aggregate Root

## Boundary

## Responsibilities

## Invariants

## Internal Entities

## Value Objects

## Behaviors

## Generated Events

## Collaborators

## Consistency Rules

## Examples

## Future Evolution
```

Todas las secciones son obligatorias.

---

# 1. Purpose

Describe qué problema del negocio resuelve el Aggregate.

Debe responder únicamente a una pregunta.

> ¿Por qué existe este Aggregate?

---

# 2. Aggregate Root

Define cuál es el único punto de entrada del Aggregate.

Toda modificación deberá pasar obligatoriamente por él.

Nunca se accederá directamente a las entidades internas.

---

# 3. Boundary

Describe qué elementos pertenecen al Aggregate.

Debe indicar claramente qué queda dentro y qué queda fuera.

El límite del Aggregate representa un límite de consistencia.

---

# 4. Responsibilities

Lista las responsabilidades del Aggregate.

Cada responsabilidad debe estar relacionada con la protección de reglas del negocio.

---

# 5. Invariants

Enumera todas las reglas que nunca pueden romperse.

Las invariantes son la principal razón de existir del Aggregate.

Ejemplos.

- Un Portfolio nunca puede contener dos Collections con el mismo identificador.
- Una Collection nunca puede superar el número máximo de Collectibles definidos.
- El presupuesto reservado nunca puede ser negativo.

---

# 6. Internal Entities

Describe qué entidades forman parte del Aggregate.

Las entidades internas no pueden ser modificadas desde el exterior.

---

# 7. Value Objects

Lista los Value Objects utilizados por el Aggregate.

Estos encapsulan el conocimiento del dominio.

---

# 8. Behaviors

Describe el comportamiento público expuesto por el Aggregate Root.

Ejemplos.

- AddCollection()
- RegisterPurchase()
- CompleteCollection()
- ReserveBudget()
- GenerateRecommendation()

El comportamiento siempre debe proteger las invariantes.

---

# 9. Generated Events

Enumera los Domain Events que el Aggregate puede producir.

Ejemplos.

- PurchaseRegistered
- RecommendationGenerated
- CollectionCompleted
- BudgetReserved

Los eventos representan hechos ya ocurridos.

---

# 10. Collaborators

Describe con qué otros Aggregates o Domain Services colabora.

La colaboración nunca debe romper los límites del Aggregate.

---

# 11. Consistency Rules

Define qué reglas deben cumplirse antes y después de cada modificación.

Estas reglas representan el contrato del Aggregate.

---

# 12. Examples

Incluye ejemplos reales del dominio.

No ejemplos de código.

---

# 13. Future Evolution

Describe posibles ampliaciones futuras compatibles con el modelo conceptual.

---

# Aggregate Root

Todo Aggregate posee exactamente un Aggregate Root.

El Root es responsable de:

- controlar el acceso;
- proteger las invariantes;
- coordinar entidades internas;
- generar eventos del dominio.

Ningún otro objeto puede modificar directamente el estado interno del Aggregate.

---

# Consistencia

El Aggregate constituye el límite de consistencia del dominio.

Toda operación que modifique el Aggregate debe dejarlo en un estado válido.

No pueden existir estados intermedios inválidos.

---

# Transacciones

Las transacciones del dominio deben coincidir, siempre que sea posible, con los límites del Aggregate.

Cada Aggregate representa una unidad natural de consistencia.

---

# Calidad Esperada

Antes de aprobar un Aggregate deberán responderse afirmativamente las siguientes preguntas.

□ ¿Existe un único Aggregate Root?

□ ¿Protege invariantes reales del negocio?

□ ¿Define claramente su límite?

□ ¿Evita dependencias innecesarias?

□ ¿Expone únicamente comportamiento?

□ ¿Oculta sus entidades internas?

□ ¿Genera eventos del dominio cuando corresponde?

□ ¿Respeta la Constitución?

□ ¿Respeta los Core Principles?

□ ¿Puede entenderse sin leer código?

---

# Anti-Patrones

Un Aggregate nunca debe convertirse en:

- Un contenedor de entidades.
- Un conjunto de tablas.
- Un objeto gigantesco.
- Un servicio disfrazado.
- Un DTO complejo.
- Un modelo de persistencia.

Si el Aggregate existe únicamente para agrupar datos probablemente está mal diseñado.

---

# Candidatos Iniciales en CollectionHub

En la versión inicial del dominio se consideran candidatos naturales:

- Portfolio Aggregate
- Collection Aggregate

La incorporación de nuevos Aggregates deberá justificarse mediante nuevas necesidades del negocio.

---

# Regla de Oro

Los límites del Aggregate deben estar definidos por las reglas del negocio y nunca por restricciones técnicas o de infraestructura.

El Aggregate protege la consistencia del dominio.

No la estructura de la base de datos.

---

# Objetivo Final

Construir un dominio compuesto por Aggregates pequeños, altamente cohesionados y responsables de mantener la consistencia del negocio.

El éxito del modelo dependerá de que cada Aggregate tenga una única responsabilidad claramente definida y que sus límites sean estables a lo largo de la evolución del sistema.