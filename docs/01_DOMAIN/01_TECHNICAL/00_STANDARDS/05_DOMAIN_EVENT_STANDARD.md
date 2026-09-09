# Domain Event Standard

Version: 1.0

Status: APPROVED

---

# Objetivo

Este documento define el estándar oficial para modelar Domain Events dentro del Dominio Técnico de CollectionHub.

Los Domain Events representan hechos relevantes que ya han ocurrido dentro del dominio.

Su propósito no es ejecutar acciones.

Su propósito es registrar conocimiento.

Cada evento representa un cambio significativo en el estado del dominio y constituye una pieza de la memoria histórica del sistema.

---

# Definición

Un Domain Event representa un hecho del negocio que ha ocurrido y no puede deshacerse conceptualmente.

Los eventos describen el pasado.

Nunca representan órdenes.

Nunca representan intenciones.

Nunca representan procesos pendientes.

---

# Principios

Todo Domain Event debe cumplir los siguientes principios.

- Representa un hecho consumado.
- Es inmutable.
- Utiliza lenguaje ubicuo.
- Describe un cambio relevante del negocio.
- Puede ser comprendido sin conocer la implementación.
- Forma parte de la memoria del dominio.
- Es independiente de la infraestructura.

---

# Cuándo crear un Domain Event

Debe existir un evento cuando:

- Se modifica el estado del dominio.
- Se toma una decisión relevante.
- Cambia el Strategic Capital.
- Se alcanza un objetivo.
- Se completa una colección.
- Se registra una compra o una venta.
- Se genera o acepta una recomendación.

---

# Cuándo NO crear un Domain Event

No crear eventos para:

- Operaciones técnicas.
- Consultas.
- Acciones sin impacto en el dominio.
- Cambios internos sin significado para el negocio.

---

# Estructura Obligatoria

Todo Domain Event deberá documentarse utilizando exactamente la siguiente estructura.

```text
# Domain Event Name

## Purpose

## Trigger

## Business Meaning

## Payload

## Consequences

## Related Principles

## Strategic Impact

## Consumers

## Examples

## Future Evolution
```

---

# 1. Purpose

Explica por qué existe el evento.

Debe responder únicamente a una pregunta.

> ¿Qué hecho relevante representa?

---

# 2. Trigger

Describe qué comportamiento del dominio provoca el evento.

Siempre debe existir una causa claramente identificable.

---

# 3. Business Meaning

Explica el significado del evento para el negocio.

Debe poder entenderse sin conocer el código.

---

# 4. Payload

Describe la información conceptual que acompaña al evento.

No se documentan propiedades técnicas.

Solo información relevante para el dominio.

---

# 5. Consequences

Describe qué cambios provoca este hecho en el dominio.

No son acciones técnicas.

Son consecuencias conceptuales.

---

# 6. Related Principles

Indica qué principios de Foundation y del Dominio Conceptual justifican el evento.

Todo evento debe poder trazarse hasta los principios.

---

# 7. Strategic Impact

Describe cómo afecta el evento a:

- Strategic Capital
- Strategic Debt
- Portfolio
- Objetivos
- Riesgo
- Liquidez

Si no existe impacto estratégico, debe justificarse.

---

# 8. Consumers

Indica qué partes del dominio pueden reaccionar al evento.

No hablar de colas, brokers ni infraestructura.

Solo consumidores conceptuales.

---

# 9. Examples

Incluye ejemplos reales del dominio.

Nunca ejemplos de código.

---

# 10. Future Evolution

Describe posibles ampliaciones futuras del evento.

---

# Inmutabilidad

Todo Domain Event es inmutable.

Una vez ocurrido, forma parte de la historia del dominio.

Nunca debe modificarse.

---

# Temporalidad

Todo evento pertenece al pasado.

Su nombre debe expresarse siempre en pasado.

Ejemplos correctos:

- PurchaseRegistered
- SaleCompleted
- RecommendationAccepted
- CollectionCompleted
- GoalAchieved
- BudgetReserved

Ejemplos incorrectos:

- RegisterPurchase
- CompleteSale
- GenerateRecommendation
- UpdateBudget

---

# Memoria Estratégica

CollectionHub considera los Domain Events como la memoria oficial del dominio.

El histórico de eventos permitirá:

- Analizar decisiones.
- Medir aprendizaje.
- Explicar recomendaciones.
- Detectar patrones.
- Evaluar la evolución del Portfolio.

---

# Calidad Esperada

Antes de aprobar un Domain Event deberán responderse afirmativamente las siguientes preguntas.

□ ¿Representa un hecho ya ocurrido?

□ ¿Tiene significado para el negocio?

□ ¿Es inmutable?

□ ¿Puede entenderse sin leer código?

□ ¿Tiene impacto estratégico o justificación?

□ ¿Utiliza el lenguaje ubicuo?

□ ¿Respeta la Constitución?

□ ¿Respeta los Core Principles?

□ ¿Puede formar parte de la historia del dominio?

□ ¿No depende de infraestructura?

---

# Anti-Patrones

Un Domain Event nunca debe convertirse en:

- Un comando.
- Una orden.
- Una llamada técnica.
- Un mensaje de integración.
- Un DTO.
- Un mecanismo de persistencia.

Si el evento expresa algo que todavía no ha ocurrido, no es un Domain Event.

---

# Candidatos Iniciales en CollectionHub

- PurchaseRegistered
- SaleCompleted
- RecommendationGenerated
- RecommendationAccepted
- CollectionCompleted
- GoalAchieved
- BudgetReserved
- StrategicCapitalUpdated
- StrategicDebtReduced
- OpportunityDetected

Esta lista evolucionará junto con el dominio.

---

# Regla de Oro

Todo Domain Event debe responder a una única pregunta:

> ¿Qué hecho relevante del negocio acaba de ocurrir?

Si no puede responderse claramente, el evento está mal definido.

---

# Objetivo Final

Construir una memoria histórica rica y coherente del dominio, donde cada evento represente un aprendizaje, una decisión o un cambio significativo que permita explicar la evolución del Portfolio y mejorar continuamente la calidad de las recomendaciones futuras.