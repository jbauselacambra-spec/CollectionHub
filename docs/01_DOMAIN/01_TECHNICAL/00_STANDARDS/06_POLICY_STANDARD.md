# Policy Standard

Version: 1.0

Status: APPROVED

---

# Objetivo

Este documento define el estándar oficial para modelar Policies dentro del Dominio Técnico de CollectionHub.

Las Policies representan reglas estratégicas del negocio que gobiernan el comportamiento del dominio.

No contienen conocimiento técnico.

No representan validaciones simples.

Representan decisiones del negocio alineadas con la Constitución y los Core Principles.

---

# Definición

Una Policy es una regla estratégica que determina cómo debe comportarse el dominio en una determinada situación.

Su misión es garantizar que todas las decisiones importantes respeten los principios fundamentales de CollectionHub.

---

# Principios

Toda Policy debe cumplir los siguientes principios.

- Representa una regla del negocio.
- Está alineada con Foundation.
- Es independiente de la infraestructura.
- Utiliza el lenguaje ubicuo.
- Es reutilizable.
- Es consistente.
- Es explicable.
- Es verificable.

---

# Cuándo utilizar una Policy

Debe existir una Policy cuando:

- Una regla afecta a múltiples procesos.
- Una decisión deriva directamente de un Core Principle.
- La regla debe mantenerse consistente en todo el dominio.
- Existen criterios estratégicos reutilizables.

---

# Cuándo NO utilizar una Policy

No utilizar una Policy cuando:

- Se trata de una validación local de una Entity.
- La regla pertenece a un Value Object.
- Es una condición reutilizable simple (Specification).
- Es lógica técnica.

---

# Estructura Obligatoria

Toda Policy deberá documentarse utilizando exactamente la siguiente estructura.

```text
# Policy Name

## Purpose

## Strategic Objective

## Related Principles

## Scope

## Decision Criteria

## Business Rules

## Inputs

## Outputs

## Exceptions

## Examples

## Future Evolution
```

---

# 1. Purpose

¿Por qué existe esta Policy?

---

# 2. Strategic Objective

¿Qué objetivo estratégico protege?

---

# 3. Related Principles

¿Qué principios de Foundation y Conceptual justifican esta Policy?

Toda Policy debe poder trazarse hasta los principios.

---

# 4. Scope

¿Dónde aplica?

Debe indicar claramente su ámbito.

---

# 5. Decision Criteria

¿Qué criterios utiliza para tomar decisiones?

Los criterios deben expresarse utilizando lenguaje ubicuo.

---

# 6. Business Rules

Lista las reglas concretas aplicadas por la Policy.

---

# 7. Inputs

Información conceptual necesaria.

Nunca DTOs.

Nunca contratos técnicos.

---

# 8. Outputs

Resultado conceptual producido.

Puede ser:

- una decisión;
- una recomendación;
- una autorización;
- un rechazo;
- una priorización.

---

# 9. Exceptions

Describe las excepciones del negocio.

No excepciones técnicas.

---

# 10. Examples

Ejemplos reales del dominio.

Nunca código.

---

# 11. Future Evolution

Posibles ampliaciones compatibles con el modelo conceptual.

---

# Calidad Esperada

Antes de aprobar una Policy deberán responderse afirmativamente las siguientes preguntas.

□ ¿Representa una regla estratégica?

□ ¿Está alineada con Foundation?

□ ¿Es reutilizable?

□ ¿Es explicable?

□ ¿Es verificable?

□ ¿Puede entenderse sin leer código?

□ ¿No depende de infraestructura?

□ ¿Utiliza el lenguaje ubicuo?

□ ¿Respeta los Core Principles?

□ ¿Tiene un objetivo estratégico claro?

---

# Anti-Patrones

Una Policy nunca debe convertirse en:

- Un helper.
- Un conjunto de ifs.
- Una validación local.
- Un servicio técnico.
- Un caso de uso.

Si la regla no representa una decisión estratégica, probablemente no pertenece a una Policy.

---

# Candidatos Iniciales en CollectionHub

- Purchase Policy
- Budget Policy
- Collection Policy
- Portfolio Policy
- Recommendation Policy
- Liquidity Policy
- Opportunity Policy

Cada nueva Policy deberá justificarse por una necesidad real del dominio.

---

# Regla de Oro

Toda Policy debe responder a una única pregunta:

> ¿Qué decisión estratégica protege esta regla del negocio?

Si no existe una respuesta clara, la Policy debe revisarse.

---

# Objetivo Final

Construir un dominio donde las reglas estratégicas estén centralizadas, documentadas y alineadas con la Constitución del proyecto, garantizando que todas las decisiones importantes sean consistentes, trazables y explicables.