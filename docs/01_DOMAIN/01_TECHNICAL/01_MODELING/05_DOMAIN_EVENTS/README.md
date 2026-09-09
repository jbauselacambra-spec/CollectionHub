# Domain Events

Version: 1.0

Status: APPROVED

---

# Purpose

La carpeta `05_DOMAIN_EVENTS` contiene la definición de los Domain Events que forman parte del dominio de CollectionHub.

Un Domain Event representa un hecho relevante del negocio que ya ha ocurrido.

Los eventos constituyen una representación de la evolución histórica del dominio y permiten mantener trazabilidad sobre decisiones, cambios y acontecimientos significativos.

No representan comandos.

No representan órdenes.

No representan mecanismos técnicos de comunicación.

---

# Scope

Esta carpeta contiene exclusivamente documentos correspondientes a Domain Events identificados y aprobados dentro del modelo de dominio.

Cada Domain Event deberá disponer de su propio documento.

---

# Relationship with Standards

Todo Domain Event documentado en esta carpeta deberá cumplir:

`00_STANDARDS/05_DOMAIN_EVENT_STANDARD.md`

El estándar técnico constituye la autoridad para determinar cómo debe documentarse y modelarse un Domain Event.

---

# Event Meaning

Todo Domain Event debe representar un hecho consumado.

Los nombres deberán expresarse en pasado.

Ejemplos conceptuales:

```text
PurchaseRegistered
CollectionCompleted
GoalAchieved
RecommendationAccepted
SaleCompleted
```

No deberán utilizarse nombres que representen acciones pendientes.

---

# Immutability

Los Domain Events son inmutables.

Una vez que un hecho del dominio ha ocurrido, su representación histórica no deberá modificarse.

Si el dominio evoluciona posteriormente, deberá registrarse un nuevo hecho.

---

# Domain Memory

Los Domain Events constituyen una parte fundamental de la memoria del dominio.

Permiten conservar información sobre:

- Decisiones.
- Compras.
- Ventas.
- Cambios relevantes.
- Objetivos alcanzados.
- Recomendaciones.
- Evolución estratégica.

El histórico de eventos deberá permitir reconstruir la evolución significativa del dominio.

---

# Strategic Traceability

Cuando un evento tenga relevancia estratégica deberá poder relacionarse con:

- Core Principles.
- Policies.
- Specifications.
- Decisions.
- Strategic Capital.
- Strategic Debt.
- Portfolio evolution.

La trazabilidad deberá permitir comprender no solo qué ocurrió, sino también el contexto estratégico que rodeó el acontecimiento.

---

# Relationship with Aggregates

Los Domain Events pueden ser generados como consecuencia del comportamiento de un Aggregate.

El evento deberá representar un hecho derivado de una transición válida del dominio.

Los eventos no deberán utilizarse para modificar directamente el estado interno de un Aggregate desde el exterior.

---

# Relationship with Policies and Specifications

Una Policy puede contribuir a determinar una decisión que posteriormente provoque un Domain Event.

Una Specification puede contribuir a evaluar las condiciones necesarias para que dicha decisión sea posible.

La relación conceptual deberá mantenerse explícita cuando sea relevante.

---

# Event Consumers

Los consumidores de un evento deberán estar definidos desde el punto de vista del dominio.

La documentación no deberá introducir tecnologías concretas como:

- Message Brokers.
- Queues.
- Kafka.
- RabbitMQ.
- Azure Service Bus.

La infraestructura será responsabilidad de otras capas.

---

# Modeling Rules

Todo Domain Event deberá:

- Representar un hecho ocurrido.
- Tener significado para el negocio.
- Ser inmutable.
- Utilizar lenguaje ubicuo.
- Tener una causa identificable.
- Poder relacionarse con el comportamiento que lo produjo.
- Mantener independencia tecnológica.
- Respetar Foundation.
- Respetar el Domain Conceptual.
- Respetar `05_DOMAIN_EVENT_STANDARD.md`.

---

# Event Lifecycle

Un Domain Event seguirá conceptualmente el siguiente ciclo:

```text
Defined
    ↓
Proposed
    ↓
Reviewed
    ↓
Approved
    ↓
Evolved
    ↓
Deprecated
```

Una vez que un evento haya sido utilizado como parte de la historia del dominio, su significado no deberá modificarse de forma incompatible.

---

# Versioning

La evolución de un Domain Event deberá preservar su significado histórico.

Los cambios que alteren significativamente su semántica deberán considerarse como un nuevo concepto de evento en lugar de modificar silenciosamente el significado anterior.

La historia del dominio debe permanecer interpretable.

---

# Quality Gate

Antes de aprobar un Domain Event deberá verificarse:

- [ ] Representa un hecho ya ocurrido.
- [ ] Tiene significado real para el negocio.
- [ ] Su nombre está expresado en pasado.
- [ ] Es inmutable.
- [ ] Su causa está identificada.
- [ ] Sus consecuencias están documentadas.
- [ ] Su relevancia estratégica está evaluada.
- [ ] Puede relacionarse con los principios correspondientes.
- [ ] No depende de infraestructura.
- [ ] Respeta `05_DOMAIN_EVENT_STANDARD.md`.

---

# Anti-Patterns

No deberán crearse Domain Events:

- Para representar comandos.
- Para representar acciones pendientes.
- Para comunicar operaciones técnicas.
- Para sustituir llamadas entre servicios.
- Para registrar cambios irrelevantes.
- Para convertir cualquier modificación de datos en un evento.

No todo cambio técnico constituye un hecho relevante del dominio.

---

# Golden Rule

> Un Domain Event representa algo que el negocio reconoce como un hecho que ha ocurrido.

Si el hecho no tiene significado para el negocio, no debe convertirse en Domain Event.

---

# Expected Outcome

Esta carpeta debe proporcionar una memoria formal y coherente de los acontecimientos relevantes del dominio.

Los Domain Events deberán permitir comprender la evolución del Portfolio, explicar decisiones importantes y proporcionar una base sólida para la futura capacidad analítica e inteligente de CollectionHub.