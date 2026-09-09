# Value Object Standard

Version: 1.0

Status: APPROVED

---

# Objetivo

Este documento define el estándar oficial para modelar Value Objects dentro del Dominio Técnico de CollectionHub.

Los Value Objects representan conceptos del negocio que poseen significado pero no identidad propia.

Su misión es encapsular conocimiento, reglas e invariantes del dominio de forma segura, expresiva e inmutable.

No representan primitivas enriquecidas.

No representan estructuras auxiliares.

Representan conceptos reales del negocio.

---

# Definición

Un Value Object es un objeto definido exclusivamente por sus atributos y comportamiento.

Dos Value Objects son iguales cuando representan exactamente el mismo valor dentro del dominio.

Su identidad no importa.

Su significado sí.

---

# Principios

Todo Value Object debe cumplir los siguientes principios.

- No posee identidad.
- Es inmutable.
- Representa un concepto del negocio.
- Encapsula comportamiento.
- Protege sus propias invariantes.
- Expresa el lenguaje ubicuo.
- Es independiente de la persistencia.
- Es independiente de la infraestructura.
- Es reutilizable dentro del dominio.

---

# Cuándo utilizar un Value Object

Un concepto del dominio debe modelarse como Value Object cuando:

- No necesita identidad propia.
- No requiere un ciclo de vida independiente.
- Puede reemplazarse completamente por otro con el mismo valor.
- Su igualdad depende exclusivamente de sus atributos.
- Su significado permanece aunque cambie el objeto que lo contiene.

---

# Cuándo NO utilizar un Value Object

No debe utilizarse cuando el concepto:

- Tiene identidad propia.
- Posee historial independiente.
- Puede existir por sí mismo.
- Necesita ser referenciado por otros elementos del dominio.
- Tiene un ciclo de vida propio.

En esos casos deberá modelarse como Entity.

---

# Estructura Obligatoria

Todo Value Object deberá documentarse utilizando exactamente la siguiente estructura.

```text
# Value Object Name

## Purpose

## Concept

## Value Definition

## Invariants

## Behaviors

## Equality

## Relationships

## Examples

## Future Evolution
```

Todas las secciones son obligatorias.

---

# 1. Purpose

Explica qué representa el Value Object dentro del negocio.

Debe responder únicamente a una pregunta.

> ¿Qué significado aporta este concepto al dominio?

---

# 2. Concept

Describe el concepto desde el punto de vista del negocio.

No debe hablar de implementación.

Debe utilizar exclusivamente el lenguaje ubicuo.

---

# 3. Value Definition

Define qué atributos forman el valor.

El valor completo determina su igualdad.

Nunca debe existir información irrelevante.

---

# 4. Invariants

Todo Value Object protege sus propias reglas.

Ejemplos.

Money

- Nunca puede representar una cantidad inválida.

CompletionPercentage

- Siempre debe estar comprendido entre 0 y 100.

Priority

- Debe pertenecer a los niveles definidos por el dominio.

Las invariantes nunca pueden romperse.

---

# 5. Behaviors

Los Value Objects contienen comportamiento.

No son simples contenedores de datos.

Ejemplos.

Money

- Add()
- Subtract()
- Multiply()

CompletionPercentage

- Increase()
- IsComplete()

StrategicCapital

- Increase()
- Decrease()
- Compare()

El comportamiento siempre debe preservar las invariantes.

---

# 6. Equality

Todo Value Object debe definir claramente cuándo dos instancias representan el mismo valor.

La igualdad depende únicamente del valor.

Nunca de la referencia.

---

# 7. Relationships

Describe con qué entidades o Value Objects suele colaborar.

No representan dependencias técnicas.

Solo relaciones conceptuales.

---

# 8. Examples

Incluye ejemplos reales del dominio.

No ejemplos de código.

Los ejemplos deben ayudar a comprender el significado del Value Object.

---

# 9. Future Evolution

Describe posibles ampliaciones futuras compatibles con el modelo conceptual.

---

# Inmutabilidad

Todos los Value Objects de CollectionHub deben ser inmutables.

Una modificación nunca altera el objeto existente.

Siempre genera una nueva instancia.

La inmutabilidad garantiza:

- Consistencia.
- Seguridad.
- Predictibilidad.
- Facilidad de razonamiento.
- Menor acoplamiento.

---

# Igualdad

La igualdad de un Value Object depende exclusivamente de su valor.

Dos objetos que representan el mismo concepto deben considerarse equivalentes.

Ejemplo conceptual.

Money(100 €)

es igual a

Money(100 €)

aunque sean instancias distintas.

---

# Comportamiento

Los Value Objects contienen lógica del dominio.

Nunca deben limitarse a almacenar información.

Su comportamiento debe proteger siempre las invariantes.

---

# Calidad Esperada

Antes de aprobar un Value Object deben responderse afirmativamente las siguientes preguntas.

□ ¿Representa un concepto del negocio?

□ ¿No necesita identidad?

□ ¿Es completamente inmutable?

□ ¿Protege sus propias invariantes?

□ ¿Encapsula comportamiento?

□ ¿Puede reutilizarse en distintas entidades?

□ ¿Puede entenderse sin leer código?

□ ¿Utiliza el lenguaje ubicuo?

□ ¿Respeta la Constitución?

□ ¿Respeta los Core Principles?

---

# Anti-Patrones

Un Value Object nunca debe convertirse en:

- Una Entity sin identificador.
- Un DTO.
- Una estructura de persistencia.
- Un conjunto de propiedades públicas.
- Un contenedor pasivo de datos.
- Un helper.

Si un Value Object no posee comportamiento probablemente está mal modelado.

---

# Ejemplos de Value Objects en CollectionHub

Los siguientes conceptos se consideran candidatos naturales a Value Object.

- Money
- StrategicCapital
- StrategicDebt
- Liquidity
- MarketValue
- TargetPrice
- CompletionPercentage
- Priority
- Scarcity
- RiskLevel
- Condition
- OpportunityScore
- ExpectedReturn
- CollectionProgress

Esta lista evolucionará conforme lo haga el dominio.

---

# Regla de Oro

Siempre que exista duda entre Entity y Value Object, deberá asumirse inicialmente que el concepto es un Value Object.

Solo se modelará como Entity cuando exista una necesidad real de identidad y ciclo de vida propio.

---

# Objetivo Final

Construir un dominio rico en conceptos, donde el conocimiento del negocio se exprese mediante objetos pequeños, inmutables y altamente cohesionados.

El éxito del modelo dependerá de que los Value Objects concentren el significado del dominio y permitan que las entidades se centren únicamente en coordinar comportamiento y proteger identidad.