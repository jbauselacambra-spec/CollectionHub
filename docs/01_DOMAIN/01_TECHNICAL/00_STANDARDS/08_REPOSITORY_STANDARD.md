# Repository Standard

Version: 1.0

Status: APPROVED

---

# Objetivo

Este documento define el estándar oficial para modelar Repositories dentro del Dominio Técnico de CollectionHub.

Los Repositories representan contratos mediante los cuales el dominio obtiene y persiste Aggregates.

No representan acceso a bases de datos.

No representan consultas SQL.

No representan Entity Framework.

Representan la persistencia conceptual del dominio.

---

# Definición

Un Repository actúa como la puerta de entrada al almacenamiento de un Aggregate.

Su responsabilidad consiste únicamente en recuperar y persistir Aggregates preservando la independencia del dominio respecto a cualquier tecnología de almacenamiento.

---

# Principios

Todo Repository debe cumplir los siguientes principios.

- Trabaja exclusivamente con Aggregates.
- Es independiente de la infraestructura.
- Es independiente de la base de datos.
- No contiene lógica de negocio.
- No modifica el dominio.
- No construye DTOs.
- No realiza transformaciones de presentación.
- Expresa el lenguaje ubicuo.

---

# Cuándo utilizar un Repository

Debe existir un Repository cuando:

- Un Aggregate necesita recuperarse.
- Un Aggregate necesita persistirse.
- El dominio requiere abstraer el mecanismo de almacenamiento.

---

# Cuándo NO utilizar un Repository

No utilizar un Repository cuando:

- Se necesitan consultas analíticas.
- Se generan informes.
- Se realizan búsquedas específicas para UI.
- Se construyen View Models.
- Se ejecutan estadísticas.
- Se implementa lógica del negocio.

Estas responsabilidades pertenecen a otras capas.

---

# Estructura Obligatoria

Todo Repository deberá documentarse utilizando exactamente la siguiente estructura.

```text
# Repository Name

## Purpose

## Managed Aggregate

## Responsibilities

## Operations

## Consistency Guarantees

## Dependencies

## Examples

## Future Evolution
```

---

# 1. Purpose

¿Por qué existe este Repository?

Debe responder únicamente a una pregunta.

> ¿Qué Aggregate administra?

---

# 2. Managed Aggregate

Debe identificar claramente el Aggregate Root gestionado.

Un Repository administra un único Aggregate.

---

# 3. Responsibilities

Lista exclusivamente responsabilidades relacionadas con la persistencia conceptual.

Nunca responsabilidades del negocio.

---

# 4. Operations

Describe conceptualmente las operaciones soportadas.

Ejemplos.

- Obtener un Portfolio.
- Guardar un Portfolio.
- Recuperar una Collection Aggregate.

No describir métodos técnicos ni tecnologías concretas.

---

# 5. Consistency Guarantees

Describe las garantías esperadas durante la recuperación y persistencia del Aggregate.

Debe respetar los límites definidos por el Aggregate.

---

# 6. Dependencies

Únicamente dependencias conceptuales.

Nunca:

- SQL Server
- Entity Framework
- MongoDB
- Redis
- APIs

---

# 7. Examples

Ejemplos reales del dominio.

Nunca ejemplos de implementación.

---

# 8. Future Evolution

Posibles ampliaciones compatibles con el dominio.

---

# Calidad Esperada

Antes de aprobar un Repository deberán responderse afirmativamente las siguientes preguntas.

□ ¿Gestiona exactamente un Aggregate?

□ ¿No contiene lógica del negocio?

□ ¿Es independiente de la infraestructura?

□ ¿Respeta los límites del Aggregate?

□ ¿Puede entenderse sin leer código?

□ ¿Utiliza el lenguaje ubicuo?

□ ¿Respeta Foundation?

□ ¿Respeta el Dominio Conceptual?

□ ¿Respeta los Core Principles?

□ ¿No conoce tecnologías concretas?

---

# Anti-Patrones

Un Repository nunca debe convertirse en:

- DAO
- CRUD Service
- Query Service
- Reporting Service
- Entity Framework Wrapper
- Helper

Si contiene reglas del negocio, está mal diseñado.

---

# Candidatos Iniciales en CollectionHub

- PortfolioRepository
- CollectionRepository

La incorporación de nuevos Repositories deberá justificarse por la aparición de nuevos Aggregates.

---

# Regla de Oro

Los Repositories existen para persistir Aggregates.

No para consultar la base de datos.

---

# Objetivo Final

Construir un dominio completamente independiente de la tecnología de persistencia, donde los Aggregates puedan evolucionar sin conocer cómo ni dónde se almacenan.