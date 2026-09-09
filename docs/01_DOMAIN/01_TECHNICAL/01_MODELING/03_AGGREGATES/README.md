# Aggregates

Version: 1.0

Status: APPROVED

---

# Purpose

La carpeta `03_AGGREGATES` contiene la definición de los Aggregates que forman parte del dominio de CollectionHub.

Un Aggregate representa un límite de consistencia del dominio.

Define qué conceptos deben mantenerse coherentes como una única unidad y establece el punto de entrada mediante el cual puede modificarse su estado.

Un Aggregate no representa una agrupación de tablas ni una agrupación técnica de objetos.

---

# Scope

Esta carpeta contiene exclusivamente documentos correspondientes a Aggregates identificados y aprobados dentro del modelo de dominio.

Cada Aggregate deberá disponer de su propio documento.

---

# Relationship with Standards

Todo Aggregate documentado en esta carpeta deberá cumplir:

`00_STANDARDS/03_AGGREGATE_STANDARD.md`

El estándar técnico constituye la autoridad para determinar cómo debe documentarse y modelarse un Aggregate.

---

# Relationship with Other Concepts

Un Aggregate puede contener:

- Un Aggregate Root.
- Entities.
- Value Objects.
- Domain Events.
- Policies.
- Specifications.

También puede colaborar con otros Aggregates y Domain Services, siempre respetando sus límites de consistencia.

---

# Aggregate Root

Todo Aggregate debe tener exactamente un Aggregate Root.

El Aggregate Root constituye el único punto de entrada para modificar el estado interno del Aggregate.

Las entidades internas no deberán ser modificadas directamente desde el exterior.

---

# Consistency Boundary

El límite del Aggregate debe estar determinado por las invariantes del negocio.

Los conceptos que deban mantenerse consistentes dentro de una misma operación deberán evaluarse como candidatos a pertenecer al mismo Aggregate.

La estructura de persistencia nunca deberá determinar por sí sola los límites del Aggregate.

---

# Modeling Rules

Todo Aggregate deberá:

- Tener exactamente un Aggregate Root.
- Tener un propósito de negocio claramente definido.
- Proteger invariantes reales.
- Definir explícitamente sus límites.
- Mantener cohesión interna.
- Minimizar dependencias externas.
- Utilizar el lenguaje ubicuo.
- Ser independiente de la infraestructura.
- Respetar Foundation.
- Respetar el Domain Conceptual.
- Respetar `03_AGGREGATE_STANDARD.md`.

---

# Transactional Boundary

El Aggregate representa una unidad natural de consistencia.

Las operaciones que modifiquen su estado deberán garantizar que el Aggregate permanezca en un estado válido.

No deberán diseñarse Aggregates únicamente para facilitar transacciones técnicas.

La consistencia del negocio tiene prioridad sobre la comodidad de persistencia.

---

# Aggregate Relationships

Los Aggregates deberán mantener referencias conceptuales mínimas hacia otros Aggregates.

Cuando una operación requiera coordinar varios Aggregates, deberá evaluarse si la responsabilidad corresponde a un Domain Service o a otro mecanismo definido por el modelo.

No se deberá convertir un Aggregate en propietario del estado de otros Aggregates.

---

# Events

Un Aggregate podrá generar Domain Events cuando se produzca un hecho relevante del negocio.

Los eventos deberán representar hechos ocurridos, no órdenes ni acciones futuras.

---

# Document Lifecycle

Un Aggregate seguirá el siguiente ciclo:

```text
Candidate
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

Ningún Aggregate deberá considerarse parte oficial del modelo hasta alcanzar el estado `Approved`.

---

# Quality Gate

Antes de aprobar un Aggregate deberá verificarse:

- [ ] Existe un único Aggregate Root.
- [ ] Su límite está justificado por invariantes del negocio.
- [ ] Su propósito está claramente definido.
- [ ] Las Entities internas están justificadas.
- [ ] Los Value Objects están justificados.
- [ ] Sus responsabilidades están claramente delimitadas.
- [ ] Sus relaciones externas son mínimas y explícitas.
- [ ] No está definido por la estructura de la base de datos.
- [ ] Respeta `03_AGGREGATE_STANDARD.md`.
- [ ] Respeta Foundation y Core Principles.

---

# Anti-Patterns

No deberán crearse Aggregates:

- Para agrupar tablas relacionadas.
- Para contener todas las Entities de un contexto.
- Para facilitar consultas.
- Para centralizar toda la lógica del dominio.
- Para evitar diseñar correctamente los límites de consistencia.
- Porque dos conceptos estén relacionados visualmente.
- Para reproducir la estructura de la base de datos.

Un Aggregate excesivamente grande deberá considerarse una señal de revisión del modelo.

---

# Golden Rule

> Un Aggregate existe para proteger una frontera de consistencia del negocio.

Si no existe una invariante que justifique su límite, deberá cuestionarse la necesidad de crear el Aggregate.

---

# Expected Outcome

Esta carpeta debe proporcionar una representación clara de las unidades de consistencia del dominio, identificando sus Roots, límites, invariantes, responsabilidades y relaciones con el resto del modelo.

El objetivo es construir Aggregates pequeños, cohesionados y orientados al comportamiento del negocio.