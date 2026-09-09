#############################################################
# CollectionHub
#############################################################

> "Las funcionalidades cambian. Los principios permanecen."

# CORE_PRINCIPLES.md

Documento: CORE_PRINCIPLES.md

Versión: 1.0.0

Estado: DRAFT (Pending Constitutional Approval)

Nivel: FOUNDATION

Prioridad: MÁXIMA

Owner: Chief Software Architect

Última actualización: 2026-07-31

---

# Índice

1. Preámbulo
2. Objetivo
3. Cómo interpretar esta Constitución
4. Jerarquía documental
5. Definiciones
6. Valores Fundamentales
7. Regla Suprema
8. Principios Fundamentales
9. Principios Estratégicos
10. Principios Económicos
11. Principios de Riesgo
12. Principios de Recomendación
13. Principios de Evolución
14. Axiomas
15. Interpretación Constitucional
16. Constitutional Compliance
17. Procedimiento de modificación
18. Relación con el resto de documentos
19. Historial de versiones

---

# 1. Preámbulo

CollectionHub nace con un propósito claro:

Ayudar a los coleccionistas a tomar mejores decisiones.

No pretende aumentar el número de compras.

No pretende incentivar el consumo.

No pretende maximizar el gasto.

CollectionHub existe para maximizar el valor estratégico de una colección a largo plazo mediante decisiones racionales, transparentes y sostenibles.

Toda decisión tomada durante el desarrollo del producto deberá perseguir ese objetivo.

La presente Constitución define los principios que gobiernan CollectionHub.

Estos principios constituyen la máxima autoridad funcional y conceptual del proyecto.

Ningún documento podrá contradecir esta Constitución.

Ningún algoritmo podrá ignorarla.

Ninguna funcionalidad podrá implementarse si entra en conflicto con alguno de sus artículos.

---

# 2. Objetivo

Esta Constitución establece:

- la identidad del producto;
- la filosofía que guía todas las decisiones;
- los principios irrenunciables;
- la forma en la que CollectionHub interpreta el coleccionismo;
- las reglas que deberán respetar todos los componentes del sistema.

Este documento constituye la fuente de verdad (Source of Truth) para cualquier decisión de diseño, negocio o desarrollo.

---

# 3. Cómo interpretar esta Constitución

Esta Constitución utiliza terminología normativa inspirada en RFC 2119.

Las palabras siguientes tienen significado específico:

| Término | Significado |
|----------|-------------|
| MUST | Obligatorio. No admite excepciones. |
| MUST NOT | Prohibido. Nunca podrá realizarse. |
| SHOULD | Altamente recomendado. Solo podrá incumplirse con justificación documentada. |
| MAY | Opcional. Depende del contexto. |

Cuando exista conflicto entre dos decisiones, prevalecerá siempre la interpretación que mejor respete los principios definidos en esta Constitución.

---

# 4. Jerarquía documental

Toda la documentación oficial de CollectionHub seguirá la siguiente jerarquía normativa.

```

PRODUCT_MANIFESTO.md

↓

CORE_PRINCIPLES.md

↓

BUSINESS_DOMAIN.md

↓

DECISION_ENGINES.md

↓

DATA_MODEL.md

↓

ARCHITECTURE.md

↓

CODING_STANDARDS.md

↓

IMPLEMENTACIÓN

```

Cada documento deberá respetar íntegramente todos los documentos situados por encima.

Un documento de nivel inferior nunca podrá redefinir un concepto establecido por uno superior.

---

# 5. Definiciones

## Colección

Conjunto organizado de piezas que persiguen un objetivo común definido por el coleccionista.

Una colección constituye un proyecto a largo plazo.

---

## Patrimonio Coleccionable

Conjunto formado por:

- las piezas adquiridas;
- las piezas pendientes;
- el presupuesto disponible;
- el conocimiento acumulado;
- la estrategia vigente.

CollectionHub analiza el patrimonio completo, no únicamente las piezas.

---

## Pieza

Elemento individual perteneciente a una colección.

Cada pieza posee un valor estratégico independiente de su precio de mercado.

---

## Riesgo

Probabilidad de que una futura decisión reduzca la capacidad del usuario para completar una colección de forma eficiente.

El riesgo constituye uno de los conceptos centrales de CollectionHub.

---

## Oportunidad

Situación temporal que puede justificar modificar la planificación prevista.

Una oportunidad nunca invalida la estrategia.

---

## Estrategia

Conjunto ordenado de decisiones destinadas a maximizar el valor futuro del patrimonio.

La estrategia prevalece siempre sobre las oportunidades aisladas.

---

# 6. Valores Fundamentales

Los valores representan la identidad permanente de CollectionHub.

Nunca podrán modificarse como consecuencia de una funcionalidad concreta.

## VF-001 — Estrategia

Toda decisión deberá perseguir un objetivo claramente definido.

La ausencia de planificación nunca podrá considerarse una estrategia válida.

---

## VF-002 — Disciplina

La disciplina constituye la principal herramienta para construir una colección sostenible.

Las decisiones impulsivas deberán minimizarse.

---

## VF-003 — Paciencia

Esperar forma parte del proceso de coleccionar.

No comprar también constituye una decisión válida.

CollectionHub nunca penalizará la paciencia.

---

## VF-004 — Transparencia

Toda recomendación generada por CollectionHub deberá poder explicarse.

No se admitirán recomendaciones opacas.

Los motores de decisión deberán justificar siempre sus conclusiones.

---

## VF-005 — Sostenibilidad

Una colección deberá poder mantenerse durante años sin comprometer la estabilidad económica del usuario.

CollectionHub nunca incentivará estrategias financieramente irresponsables.

---

## VF-006 — Mejora Continua

CollectionHub deberá evolucionar constantemente.

No obstante, ninguna evolución podrá contradecir esta Constitución.

---
# 7. Regla Suprema

---

# Artículo I — CP-000

## La Regla Suprema

### Identificador

CP-000

### Nombre

Incrementar el Valor Estratégico del Patrimonio

### Regla

**Toda decisión, recomendación, algoritmo, métrica o funcionalidad implementada en CollectionHub MUST incrementar el valor estratégico del patrimonio del usuario sin comprometer su sostenibilidad futura.**

Esta regla prevalece sobre cualquier otro principio definido en esta Constitución.

Cuando dos artículos entren en conflicto, siempre prevalecerá la interpretación que mejor respete este principio.

---

### Motivación

CollectionHub no existe para aumentar el número de compras.

No existe para mostrar estadísticas.

No existe para gestionar inventarios.

CollectionHub existe para ayudar al usuario a tomar mejores decisiones.

El éxito del producto nunca se medirá por el número de compras realizadas.

Se medirá por la calidad de las decisiones tomadas.

---

### Consecuencias

Toda funcionalidad deberá justificar cómo incrementa el valor estratégico del patrimonio.

Toda recomendación deberá poder explicarse.

Toda decisión deberá ser compatible con la sostenibilidad económica del usuario.

Ninguna funcionalidad podrá incentivar decisiones impulsivas.

---

### Documentos afectados

PRODUCT_MANIFESTO.md

BUSINESS_DOMAIN.md

DECISION_ENGINES.md

DATA_MODEL.md

ARCHITECTURE.md

---

### Estado

APPROVED

---

# 8. Principios Fundamentales

Los principios fundamentales representan las reglas permanentes sobre las que se construye CollectionHub.

Todos los motores de decisión deberán respetarlos.

Todos los algoritmos deberán implementarlos.

Toda funcionalidad deberá justificarse utilizando al menos uno de estos principios.

---

# Artículo II — CP-101

## La Estrategia prevalece sobre la Oportunidad

### Regla

CollectionHub MUST priorizar la estrategia global frente a cualquier oportunidad aislada.

---

### Motivación

Una oportunidad solamente tiene sentido dentro de una estrategia.

Comprar algo únicamente porque está barato no constituye una buena decisión.

Una buena oportunidad fuera de estrategia puede convertirse en una mala compra.

---

### Ejemplo

El usuario está ahorrando para adquirir una pieza crítica de 110 €.

Aparece un videojuego con un 40 % de descuento.

Aunque el descuento sea elevado, CollectionHub NO deberá recomendar dicha compra si compromete el objetivo principal.

---

### Consecuencias

Opportunity Engine.

Recommendation Engine.

Budget Engine.

Dashboard.

---

### Estado

APPROVED

---

# Artículo III — CP-102

## Reducir el Riesgo tiene prioridad sobre aumentar el Porcentaje

### Regla

CollectionHub MUST priorizar la reducción del riesgo estratégico antes que incrementar el porcentaje completado de una colección.

---

### Motivación

Dos colecciones pueden presentar exactamente el mismo porcentaje de progreso.

Sin embargo, una puede encontrarse prácticamente asegurada mientras la otra depende todavía de varias piezas extremadamente difíciles.

El porcentaje visual no representa el riesgo real.

---

### Ejemplo

Colección A

80 %

Solo faltan piezas comunes.

Colección B

80 %

Faltan las tres piezas más difíciles.

CollectionHub considerará que la Colección A presenta un progreso estratégico superior.

---

### Consecuencias

Nacimiento de:

Strategic Progress

Collection Risk

Acquisition Risk

Future Risk

---

### Estado

APPROVED

---

# Artículo IV — CP-103

## Priorizar las Piezas Críticas

### Regla

Las piezas clasificadas como críticas SHOULD adquirirse antes que las piezas comunes cuando el presupuesto disponible lo permita.

---

### Definición de Pieza Crítica

Una pieza será considerada crítica cuando presente uno o varios de los siguientes factores:

- alta dificultad de adquisición;
- baja disponibilidad;
- tendencia alcista de precio;
- riesgo elevado de descatalogación;
- importancia estratégica para completar una colección.

---

### Motivación

Reducir el número de piezas críticas pendientes disminuye significativamente el riesgo futuro de la colección.

---

### Inspiración

Este principio nace directamente de la estrategia real utilizada por el creador de CollectionHub.

Primero asegurar las piezas difíciles.

Después completar el resto de la colección sin presión.

---

### Ejemplo

Comprar Soundwave antes que tres juegos comunes de Nintendo Switch.

Comprar un casco LEGO difícil antes que varios sets fácilmente localizables.

---

### Consecuencias

Recommendation Engine.

Priority Engine.

Collection Risk Engine.

---

### Estado

APPROVED

---

# Artículo V — CP-104

## Toda Compra tiene un Coste de Oportunidad

### Regla

Toda compra reduce temporalmente la capacidad del usuario para realizar futuras adquisiciones.

CollectionHub MUST considerar siempre dicho coste antes de emitir una recomendación.

---

### Motivación

El presupuesto constituye un recurso limitado.

Cada euro utilizado deja de estar disponible para futuras oportunidades.

---

### Ejemplo

Comprar tres juegos de 30 € puede impedir adquirir una pieza crítica de 90 € la semana siguiente.

---

### Consecuencias

Budget Engine.

Recommendation Engine.

Opportunity Engine.

---

### Estado

APPROVED

---

# Artículo VI — CP-105

## Esperar también constituye una Decisión Estratégica

### Regla

CollectionHub MUST considerar la ausencia de compra como una decisión completamente válida.

---

### Motivación

No comprar puede incrementar el valor estratégico del patrimonio.

Esperar puede permitir acceder a mejores oportunidades futuras.

---

### Consecuencias

El sistema nunca penalizará al usuario por no comprar.

Las recomendaciones podrán consistir simplemente en:

> Esperar.

---

### Estado

APPROVED

---

# Artículo VII — CP-106

## Cerrar una Colección tiene prioridad sobre abrir una nueva

### Regla

CollectionHub SHOULD recomendar completar colecciones ya iniciadas antes de dispersar recursos en nuevas colecciones, salvo que exista una oportunidad excepcional que no comprometa la estrategia global.

---

### Motivación

Las colecciones incompletas mantienen un riesgo estratégico superior.

Cerrar una colección reduce incertidumbre, simplifica objetivos y libera recursos mentales y económicos.

---

### Ejemplo

Completar la colección de cascos LEGO pendientes antes de comenzar una nueva línea de LEGO Architecture.

---

### Estado

APPROVED

---

# Artículo VIII — CP-107

## Cada Euro Debe Tener una Misión

### Regla

Todo presupuesto disponible MUST estar asignado a un objetivo estratégico, aunque dicho objetivo sea permanecer en reserva.

El dinero sin propósito representa una oportunidad desaprovechada.

---

### Motivación

El presupuesto forma parte del patrimonio del usuario.

No constituye simplemente un saldo económico.

Representa capacidad futura de decisión.

Por ello, CollectionHub deberá conocer siempre cuál es el destino previsto para cada euro disponible.

---

### Ejemplos

✔ Ahorrar para adquirir Soundwave.

✔ Reservar presupuesto para completar la colección de cascos LEGO.

✔ Mantener un fondo para oportunidades excepcionales.

✔ Acumular liquidez antes de iniciar una nueva colección.

---

### Consecuencias

Budget Engine.

Recommendation Engine.

Strategic Planner.

Savings Dashboard.

---

### Estado

APPROVED

---

# Artículo IX — CP-108

## El Presupuesto también forma parte del Patrimonio

### Regla

CollectionHub MUST considerar el presupuesto disponible como un activo estratégico.

Nunca será tratado como un simple dato financiero.

---

### Motivación

Dos usuarios con exactamente la misma colección pueden encontrarse en situaciones completamente diferentes si uno dispone de liquidez y el otro no.

La capacidad futura de compra forma parte del patrimonio.

---

### Consecuencias

El valor estratégico del patrimonio será calculado utilizando:

- Colecciones
- Piezas
- Liquidez
- Riesgo
- Prioridades

---

### Estado

APPROVED

---

# Artículo X — CP-109

## La Disciplina prevalece sobre la Impulsividad

### Regla

CollectionHub MUST reducir la probabilidad de compras impulsivas.

Nunca incentivará decisiones basadas únicamente en emociones momentáneas.

---

### Motivación

La mayoría de errores en una colección aparecen por decisiones impulsivas.

El objetivo del sistema es actuar como una segunda opinión racional.

---

### Ejemplos

El sistema podrá mostrar mensajes como:

> Esta compra retrasa tu objetivo principal.

o

> Si esperas aproximadamente dos meses mantendrás intacta tu estrategia.

Nunca utilizará mensajes del tipo:

> ¡Compra ahora antes de que desaparezca!

---

### Estado

APPROVED

---

# Artículo XI — CP-110

## La Colección debe evolucionar de forma Sostenible

### Regla

CollectionHub MUST recomendar un ritmo de crecimiento compatible con la capacidad económica del usuario.

---

### Motivación

Completar una colección un año antes no compensa si obliga al usuario a realizar esfuerzos financieros innecesarios.

Las colecciones exitosas se construyen durante años.

No durante semanas.

---

### Estado

APPROVED

---

# Artículo XII — CP-111

## Toda Recomendación debe poder Explicarse

### Regla

Toda recomendación generada por CollectionHub MUST incluir una justificación objetiva.

No se admitirán recomendaciones sin explicación.

---

### Motivación

El usuario debe comprender el razonamiento del sistema.

La confianza nace de la transparencia.

---

### Formato recomendado

Toda recomendación debería responder al menos a:

- ¿Por qué se recomienda?
- ¿Qué riesgo reduce?
- ¿Qué objetivo favorece?
- ¿Qué alternativas existen?
- ¿Qué consecuencias tendría no seguirla?

---

### Estado

APPROVED

---

# Artículo XIII — CP-112

## CollectionHub Recomienda. Nunca Decide.

### Regla

La decisión final MUST pertenecer siempre al usuario.

---

### Motivación

CollectionHub es un sistema de apoyo a la decisión.

No sustituye el criterio del coleccionista.

---

### Consecuencias

Nunca existirán:

- compras automáticas;
- prioridades bloqueadas;
- decisiones obligatorias.

El usuario podrá ignorar cualquier recomendación.

---

### Estado

APPROVED

---

# Artículo XIV — CP-113

## El Descuento nunca Justifica por sí solo una Compra

### Regla

CollectionHub MUST NOT recomendar una compra únicamente por su porcentaje de descuento.

---

### Motivación

Un gran descuento sobre una pieza irrelevante puede resultar peor decisión que pagar el precio habitual por una pieza estratégica.

---

### Ejemplo

Juego A

50 % descuento.

No pertenece a ninguna prioridad.

Juego B

Sin descuento.

Completa una colección prioritaria.

CollectionHub favorecerá el Juego B.

---

### Estado

APPROVED

---

# Artículo XV — CP-114

## El Progreso Estratégico tiene más Valor que el Progreso Visual

### Regla

Los indicadores estratégicos MUST prevalecer sobre los porcentajes de completitud.

---

### Motivación

Completar piezas fáciles puede aumentar el porcentaje visual sin reducir el riesgo real.

CollectionHub medirá el progreso utilizando múltiples dimensiones.

---

### Métricas futuras

- Strategic Progress
- Collection Risk
- Acquisition Difficulty
- Opportunity Readiness
- Budget Health
- Priority Score

---

### Estado

APPROVED

---

# Artículo XVI — CP-115

## El Valor Estratégico prevalece sobre el Valor Económico

### Regla

CollectionHub MUST distinguir entre precio y valor estratégico.

Nunca asumirá que una pieza cara es automáticamente más importante.

---

### Motivación

Existen piezas económicas cuya ausencia bloquea una colección completa.

También existen piezas muy costosas cuya adquisición puede esperar.

El precio representa únicamente una dimensión de análisis.

---

### Estado

APPROVED

# 9. Principios Estratégicos

Los siguientes principios gobiernan la planificación estratégica del patrimonio coleccionable.

Estos principios deberán ser utilizados por todos los motores de recomendación, planificación y análisis.

---

# Artículo XVII — CP-201

## Reducir Incertidumbre siempre genera Valor

### Regla

CollectionHub SHOULD priorizar decisiones que reduzcan la incertidumbre futura del patrimonio.

---

### Motivación

Toda colección contiene incertidumbre:

- futuras subidas de precio;
- descatalogaciones;
- baja disponibilidad;
- aparición de oportunidades inesperadas.

Reducir esa incertidumbre incrementa el valor estratégico de la colección.

---

### Ejemplo

Comprar una pieza difícil hoy puede eliminar años de incertidumbre.

Aunque el ahorro económico sea pequeño, el beneficio estratégico puede ser enorme.

---

### Estado

APPROVED

---

# Artículo XVIII — CP-202

## El Riesgo debe Distribuirse

### Regla

CollectionHub SHOULD evitar concentrar múltiples riesgos importantes dentro de una misma colección.

---

### Motivación

Una colección que depende de varias piezas extremadamente difíciles presenta un riesgo muy elevado.

El objetivo es repartir ese riesgo a lo largo del tiempo.

---

### Ejemplo

Incorrecto

Pendientes:

- Darth Vader
- Jango Fett
- Soundwave
- Torna
- Bayonetta 1

Correcto

Asegurar primero una o dos piezas críticas.

Después continuar con el resto.

---

### Estado

APPROVED

---

# Artículo XIX — CP-203

## El Riesgo futuro es más importante que el Riesgo presente

### Regla

CollectionHub MUST analizar el impacto futuro de una decisión antes que su beneficio inmediato.

---

### Motivación

Una compra aparentemente buena puede impedir una oportunidad mucho mejor dentro de unas semanas.

El sistema siempre evaluará el coste futuro de cada decisión.

---

### Estado

APPROVED

---

# Artículo XX — CP-204

## Toda Colección posee una Ruta Crítica

### Regla

CollectionHub MUST identificar las piezas que condicionan la finalización de cada colección.

---

### Definición

La Ruta Crítica representa el conjunto mínimo de adquisiciones necesarias para asegurar el éxito futuro de una colección.

---

### Consecuencias

Cada colección tendrá:

- Ruta Crítica
- Prioridad
- Riesgo
- Tiempo estimado de cierre

---

### Estado

APPROVED

---

# Artículo XXI — CP-205

## La Prioridad pertenece a la Colección, no a la Pieza

### Regla

Las piezas heredarán su prioridad de la colección a la que pertenecen.

Una pieza secundaria puede convertirse en prioritaria si pertenece a una colección estratégica.

---

### Motivación

CollectionHub optimiza patrimonio, no objetos individuales.

---

### Estado

APPROVED

---

# Artículo XXII — CP-206

## Las Prioridades son Dinámicas

### Regla

Las prioridades MUST recalcularse automáticamente tras cualquier cambio relevante.

---

### Eventos

- Compra.
- Venta.
- Cambio de presupuesto.
- Nueva colección.
- Descatalogación.
- Cambio importante de precio.
- Cambio en la estrategia del usuario.

---

### Estado

APPROVED

---

# Artículo XXIII — CP-207

## Toda Compra modifica el Futuro

### Regla

CollectionHub MUST recalcular el estado estratégico completo después de cada adquisición.

---

### Motivación

Ninguna compra afecta únicamente a una colección.

Puede modificar:

- presupuesto;
- liquidez;
- oportunidades;
- prioridades;
- riesgo;
- planificación futura.

---

### Estado

APPROVED

---

# Artículo XXIV — CP-208

## El Capital Estratégico es el principal activo del usuario

### Regla

CollectionHub MUST calcular y proteger el Capital Estratégico del usuario.

---

### Definición

El Capital Estratégico representa la capacidad real del usuario para tomar buenas decisiones futuras.

No depende únicamente del dinero.

Incluye múltiples dimensiones.

---

### Componentes

- Liquidez disponible.
- Prioridades activas.
- Riesgo del patrimonio.
- Número de oportunidades futuras.
- Tiempo disponible.
- Conocimiento acumulado.
- Colecciones abiertas.
- Colecciones cerradas.

---

### Objetivo

Toda recomendación deberá intentar aumentar este indicador.

---

### Estado

APPROVED

---

# Artículo XXV — CP-209

## Las Colecciones deben tender al Equilibrio

### Regla

CollectionHub SHOULD evitar diferencias excesivas de progreso entre colecciones prioritarias.

---

### Motivación

Una colección completamente abandonada consume recursos mentales y aumenta la sensación de proyecto inacabado.

---

### Estado

APPROVED

---

# Artículo XXVI — CP-210

## El Tiempo también es un Recurso Estratégico

### Regla

CollectionHub MUST considerar el tiempo como parte del patrimonio del usuario.

---

### Motivación

Poder esperar constituye una ventaja competitiva.

El tiempo permite:

- negociar;
- encontrar oportunidades;
- reducir precios;
- disminuir riesgo.

---

### Consecuencias

El sistema nunca penalizará la paciencia.

De hecho, podrá recomendar explícitamente:

"Es mejor esperar."

---

### Estado

APPROVED

---

# Artículo XXVII — CP-211

## La Estrategia debe ser Personal

### Regla

CollectionHub MUST adaptar todas las recomendaciones al comportamiento real del usuario.

---

### Motivación

No existen estrategias universales.

Cada coleccionista posee:

- prioridades;
- presupuesto;
- ritmo;
- tolerancia al riesgo;
- objetivos distintos.

La personalización constituye un principio constitucional.

---

### Estado

APPROVED

# 10. Principios Económicos

Los principios económicos definen la forma en que CollectionHub interpreta el presupuesto del usuario.

El dinero nunca será considerado únicamente como un saldo disponible.

Representa capacidad estratégica.

Toda recomendación económica deberá proteger esa capacidad.

---

# Artículo XXVIII — CP-301

## El Presupuesto es Finito

### Regla

CollectionHub MUST asumir que todo presupuesto es limitado.

Toda recomendación económica deberá optimizar recursos escasos.

---

### Motivación

Las oportunidades son prácticamente infinitas.

El presupuesto nunca lo es.

Por ello toda decisión económica implica una renuncia.

---

### Consecuencias

Todo algoritmo deberá considerar el coste de oportunidad.

No podrán existir recomendaciones independientes del presupuesto disponible.

---

### Estado

APPROVED

---

# Artículo XXIX — CP-302

## Toda Compra Consume Capital Estratégico

### Regla

Toda adquisición reduce temporalmente el Capital Estratégico disponible.

CollectionHub MUST recalcular dicho capital inmediatamente después de cualquier compra.

---

### Motivación

Comprar una pieza no solo reduce dinero.

También modifica:

- la capacidad futura;
- las oportunidades disponibles;
- la flexibilidad del usuario.

---

### Estado

APPROVED

---

# Artículo XXX — CP-303

## La Liquidez tiene Valor Propio

### Regla

Mantener liquidez MAY ser más valioso que realizar una compra inmediata.

---

### Motivación

La liquidez proporciona capacidad de reacción.

Permite aprovechar oportunidades inesperadas.

Reduce el riesgo.

Aumenta la flexibilidad.

---

### Ejemplo

Usuario A

0 €

Usuario B

150 €

Misma colección.

CollectionHub considerará que el Usuario B posee una mejor posición estratégica.

---

### Estado

APPROVED

---

# Artículo XXXI — CP-304

## El Presupuesto debe estar Asignado

### Regla

Todo presupuesto disponible SHOULD encontrarse asociado a un objetivo.

---

### Tipos de asignación

- Pieza concreta.
- Colección.
- Fondo de oportunidades.
- Reserva estratégica.
- Ahorro planificado.

---

### Motivación

El dinero sin propósito suele terminar utilizándose de forma impulsiva.

---

### Estado

APPROVED

---

# Artículo XXXII — CP-305

## Las Compras deben competir entre sí

### Regla

CollectionHub MUST comparar todas las compras potenciales antes de emitir una recomendación.

Nunca evaluará una compra de forma aislada.

---

### Motivación

La pregunta correcta nunca es:

"¿Es buena compra?"

La pregunta correcta es:

"¿Es la mejor compra posible para este presupuesto?"

---

### Estado

APPROVED

---

# Artículo XXXIII — CP-306

## La Mejor Compra puede ser No Comprar

### Regla

El sistema MAY concluir que ninguna compra actual mejora el patrimonio estratégico.

En ese caso deberá recomendar mantener el presupuesto.

---

### Motivación

No toda oportunidad merece ser aprovechada.

Esperar también genera valor.

---

### Estado

APPROVED

---

# Artículo XXXIV — CP-307

## El Precio Ideal es Dinámico

### Regla

CollectionHub MUST distinguir entre:

- Precio de mercado.
- Precio objetivo.
- Precio máximo.
- Precio excepcional.

---

### Definiciones

Precio de mercado

Valor medio habitual.

---

Precio objetivo

Precio al que CollectionHub recomienda comprar.

---

Precio máximo

Precio que todavía puede considerarse aceptable.

---

Precio excepcional

Precio muy inferior al esperado que justifica revisar prioridades.

---

### Consecuencias

Todas las piezas podrán almacenar estos valores.

---

### Estado

APPROVED

---

# Artículo XXXV — CP-308

## Las Oportunidades tienen Caducidad

### Regla

Toda oportunidad deberá evaluarse considerando su duración estimada.

---

### Motivación

No todas las oportunidades desaparecerán mañana.

CollectionHub evitará generar sensación artificial de urgencia.

---

### Estado

APPROVED

---

# Artículo XXXVI — CP-309

## Las Compras deben maximizar el Valor Marginal

### Regla

CollectionHub SHOULD recomendar aquellas adquisiciones cuyo beneficio estratégico por euro invertido sea mayor.

---

### Concepto

Valor Marginal Estratégico

=

Incremento del patrimonio estratégico

/

Capital invertido

---

### Ejemplo

Compra A

40 €

Incrementa mucho la seguridad de una colección.

Compra B

40 €

Solo aumenta el porcentaje visual.

CollectionHub favorecerá la Compra A.

---

### Estado

APPROVED

---

# Artículo XXXVII — CP-310

## Toda Compra modifica la Planificación Financiera

### Regla

Después de cada compra deberán recalcularse:

- presupuesto disponible;
- ahorro previsto;
- próximos objetivos;
- fecha estimada de compra;
- capital estratégico.

---

### Estado

APPROVED

---

# Artículo XXXVIII — CP-311

## Nunca comprometer la Estrategia Global

### Regla

CollectionHub MUST rechazar cualquier recomendación que comprometa la estrategia global del usuario.

Aunque la compra parezca buena individualmente.

---

### Motivación

El patrimonio debe evolucionar como un sistema.

No como una suma de compras independientes.

---

### Estado

APPROVED

# 11. Principios de Recomendación

Los siguientes principios regulan el comportamiento de todos los motores de recomendación presentes y futuros.

Toda recomendación generada por CollectionHub deberá cumplir simultáneamente estos principios.

---

# Artículo XXXIX — CP-401

## Toda recomendación debe aportar Valor Estratégico

### Regla

Toda recomendación MUST incrementar el Valor Estratégico del Patrimonio.

Si una recomendación no mejora objetivamente el patrimonio, no deberá emitirse.

---

### Motivación

CollectionHub no recomienda compras.

Recomienda mejoras del patrimonio.

---

### Estado

APPROVED

---

# Artículo XL — CP-402

## Las recomendaciones deben ser Explicables

### Regla

Toda recomendación deberá incluir una explicación comprensible para el usuario.

Nunca existirá una recomendación basada únicamente en una puntuación interna.

---

### Información mínima

Cada recomendación deberá explicar:

- por qué aparece;
- qué riesgo reduce;
- qué objetivo favorece;
- qué principios constitucionales utiliza;
- qué consecuencias tendría ignorarla.

---

### Ejemplo

Recomendación

Comprar Soundwave.

Motivos

✔ Reduce el riesgo de la colección Transformers.

✔ Permite cerrar la colección.

✔ Precio dentro del rango objetivo.

✔ Mantiene suficiente liquidez.

✔ Cumple CP-103, CP-203 y CP-305.

---

### Estado

APPROVED

---

# Artículo XLI — CP-403

## Las recomendaciones deben competir entre sí

### Regla

CollectionHub MUST ordenar todas las recomendaciones disponibles.

Nunca mostrará una única opción sin contexto.

---

### Motivación

El usuario debe conocer cuál es la mejor decisión y cuáles son las alternativas.

---

### Resultado esperado

Ranking estratégico.

1.

Soundwave

SIS 97

2.

Luke Helmet

SIS 93

3.

Metroid Dread

SIS 71

4.

Rubik Batman

SIS 58

---

### Estado

APPROVED

---

# Artículo XLII — CP-404

## CollectionHub debe recomendar Esperar cuando sea la mejor opción

### Regla

El sistema MAY recomendar explícitamente no realizar ninguna compra.

---

### Motivación

Esperar puede producir un beneficio estratégico superior.

---

### Ejemplo

"No existe actualmente ninguna oportunidad capaz de mejorar tu patrimonio.

La recomendación es mantener liquidez."

---

### Estado

APPROVED

---

# Artículo XLIII — CP-405

## Las recomendaciones deben ser Personalizadas

### Regla

Toda recomendación MUST adaptarse al comportamiento real del usuario.

---

### Variables

- prioridades
- presupuesto
- liquidez
- velocidad de ahorro
- perfil de riesgo
- colecciones activas
- estrategia vigente

---

### Motivación

No existen recomendaciones universales.

---

### Estado

APPROVED

---

# Artículo XLIV — CP-406

## Nunca recomendar Compras Impulsivas

### Regla

CollectionHub MUST NOT incentivar compras basadas exclusivamente en:

- ofertas;
- miedo a perder la oportunidad;
- presión temporal;
- emociones.

---

### Motivación

Las decisiones impulsivas deterioran el patrimonio estratégico.

---

### Estado

APPROVED

---

# Artículo XLV — CP-407

## Toda recomendación tiene un Coste de Oportunidad

### Regla

El sistema deberá informar siempre del coste de oportunidad asociado.

---

### Ejemplo

Comprar hoy:

Luke Helmet.

Consecuencia:

Retrasa aproximadamente 3 semanas la compra de Soundwave.

---

### Estado

APPROVED

---

# Artículo XLVI — CP-408

## Las recomendaciones deben ser Revisables

### Regla

Toda recomendación deberá recalcularse automáticamente cuando cambie el contexto.

---

### Eventos

- nueva compra;
- nueva venta;
- cambio de presupuesto;
- nueva oportunidad;
- actualización de precios;
- modificación de prioridades.

---

### Estado

APPROVED

---

# Artículo XLVII — CP-409

## Nunca ocultar Incertidumbre

### Regla

CollectionHub MUST indicar el nivel de confianza asociado a cada recomendación.

---

### Ejemplo

Confianza

96 %

Precio muy estable.

---

Confianza

58 %

Mercado muy volátil.

---

### Motivación

El usuario debe conocer cuándo una recomendación es sólida y cuándo depende de variables inciertas.

---

### Estado

APPROVED

---

# Artículo XLVIII — CP-410

## La IA asesora. El Usuario decide.

### Regla

La Inteligencia Artificial constituye un asistente estratégico.

Nunca sustituirá el criterio del usuario.

---

### Consecuencias

La IA podrá:

✔ explicar;

✔ comparar;

✔ simular escenarios;

✔ detectar oportunidades;

✔ calcular riesgos.

La IA nunca podrá:

✘ comprar;

✘ modificar prioridades sin autorización;

✘ alterar presupuestos;

✘ iniciar colecciones automáticamente.

---

### Estado

APPROVED

---

# Artículo XLIX — CP-411

## Toda recomendación debe ser Trazable

### Regla

Cada recomendación deberá indicar explícitamente los artículos constitucionales utilizados para generarla.

---

### Ejemplo

Recomendación:

Comprar Soundwave.

Fundamentación Constitucional

✔ CP-103

✔ CP-203

✔ CP-305

✔ CP-401

---

### Estado

APPROVED

---

# Artículo L — CP-412

## La recomendación perfecta no existe

### Regla

CollectionHub MUST reconocer que toda recomendación es una estimación basada en la información disponible.

---

### Motivación

El mercado cambia.

Las personas cambian.

Las prioridades cambian.

CollectionHub recomendará siempre la mejor decisión posible con los datos actuales.

Nunca afirmará poseer la verdad absoluta.

---

### Estado

APPROVED

# 12. Principios de Integridad del Sistema

Los siguientes principios establecen los límites inquebrantables de CollectionHub.

Su objetivo es preservar la identidad del producto incluso cuando evolucione durante años.

Estos principios tienen el mismo nivel jerárquico que el resto de artículos constitucionales.

---

# Artículo LI — CP-501

## CollectionHub nunca fomentará el consumismo

### Regla

CollectionHub MUST NOT incentivar la compra de productos cuyo único argumento sea aumentar el número de adquisiciones realizadas.

---

### Motivación

CollectionHub existe para construir patrimonio.

No para aumentar el consumo.

El éxito del usuario no se mide por cuánto compra.

Se mide por la calidad de las decisiones tomadas.

---

### Consecuencias

Nunca existirán métricas como:

• Número de compras este mes.

• Objetivo de compras mensuales.

• Rachas de compra.

• Logros por comprar.

El sistema premiará únicamente decisiones estratégicas.

---

### Estado

APPROVED

---

# Artículo LII — CP-502

## CollectionHub nunca manipulará al usuario

### Regla

CollectionHub MUST NOT utilizar patrones de diseño destinados a provocar decisiones impulsivas.

---

### Quedan prohibidos

• Contadores falsos.

• Ofertas engañosas.

• Mensajes de urgencia artificial.

• Colores diseñados para inducir compras.

• Notificaciones manipulativas.

---

### Motivación

La confianza del usuario constituye un activo superior a cualquier venta.

---

### Estado

APPROVED

---

# Artículo LIII — CP-503

## La transparencia prevalece sobre la complejidad

### Regla

Todo cálculo importante deberá poder explicarse.

---

### Motivación

Un algoritmo complejo no aporta valor si el usuario no comprende por qué recomienda una determinada acción.

---

### Ejemplo

Incorrecto

"Puntuación 92."

Correcto

"Puntuación 92 porque reduce el riesgo de la colección, mantiene liquidez suficiente y aprovecha una oportunidad dentro de tu precio objetivo."

---

### Estado

APPROVED

---

# Artículo LIV — CP-504

## CollectionHub protege la estrategia del usuario

### Regla

Cuando una acción contradiga claramente la estrategia definida, el sistema deberá advertirlo.

La advertencia nunca bloqueará la decisión.

---

### Ejemplo

"Esta compra retrasará aproximadamente dos meses el cierre de tu colección prioritaria."

---

### Motivación

Advertir no significa prohibir.

El usuario conserva siempre el control.

---

### Estado

APPROVED

---

# Artículo LV — CP-505

## El error forma parte del aprendizaje

### Regla

CollectionHub MAY registrar decisiones que no siguieron las recomendaciones para mejorar el conocimiento del usuario y del sistema.

---

### Motivación

Una recomendación ignorada también aporta información.

El objetivo es aprender.

No juzgar.

---

### Estado

APPROVED

---

# Artículo LVI — CP-506

## La evolución nunca romperá la identidad

### Regla

Toda nueva funcionalidad deberá respetar esta Constitución.

Si una nueva característica entra en conflicto con cualquiera de los artículos aquí definidos, deberá modificarse o rechazarse.

---

### Motivación

El crecimiento sin coherencia destruye los productos.

La identidad debe permanecer estable.

---

### Estado

APPROVED

---

# Artículo LVII — CP-507

## Los principios prevalecen sobre las funcionalidades

### Regla

Cuando exista conflicto entre una funcionalidad y un principio constitucional, prevalecerá siempre el principio.

---

### Ejemplo

Una funcionalidad muy demandada que incentive compras impulsivas nunca será implementada.

Aunque resulte rentable.

---

### Estado

APPROVED

---

# Artículo LVIII — CP-508

## La calidad prevalece sobre la velocidad

### Regla

CollectionHub SHOULD retrasar una funcionalidad antes que implementarla de forma incoherente.

---

### Motivación

El usuario olvidará una funcionalidad que llegó un mes tarde.

Nunca olvidará una mala decisión del producto.

---

### Estado

APPROVED

---

# Artículo LIX — CP-509

## Toda decisión debe ser reversible

### Regla

Siempre que sea técnicamente posible, las decisiones tomadas por el usuario deberán poder modificarse.

---

### Ejemplos

✔ Cambiar prioridad de una colección.

✔ Cambiar precio objetivo.

✔ Cambiar estrategia.

✔ Modificar presupuesto.

✔ Reabrir una colección.

---

### Motivación

Las personas evolucionan.

Su estrategia también.

CollectionHub debe adaptarse.

---

### Estado

APPROVED

---

# Artículo LX — CP-510

## CollectionHub protege el tiempo del usuario

### Regla

Toda funcionalidad deberá justificar el tiempo que requiere.

---

### Motivación

El tiempo constituye parte del Capital Estratégico.

Una funcionalidad que exige demasiado esfuerzo sin aportar valor estratégico deberá simplificarse o eliminarse.

---

### Estado

APPROVED

# DECLARACIÓN DE PROPÓSITO

## ¿Por qué existe CollectionHub?

CollectionHub nace de una idea muy sencilla.

Coleccionar no consiste en comprar.

Consiste en decidir.

Cada compra representa una decisión.

Cada decisión modifica el futuro de una colección.

Cada colección forma parte de un patrimonio construido durante años.

Sin embargo, la mayoría de herramientas existentes únicamente permiten registrar qué posee un coleccionista.

CollectionHub nace para responder preguntas mucho más importantes.

¿Qué debería comprar ahora?

¿Es el mejor momento?

¿Qué oportunidad debería dejar pasar?

¿Cuánto riesgo tiene mi colección?

¿Cuál es el siguiente paso más inteligente?

CollectionHub no pretende sustituir el criterio del usuario.

Pretende ayudarle a tomar mejores decisiones.

---

## Nuestra misión

Ayudar a cada coleccionista a construir un patrimonio sostenible mediante decisiones estratégicas.

---

## Nuestra visión

Convertirse en el sistema de inteligencia estratégica para coleccionistas más avanzado del mundo.

No por la cantidad de datos.

Sino por la calidad de sus recomendaciones.

---

## Nuestros valores

La estrategia antes que la impulsividad.

La paciencia antes que la prisa.

La transparencia antes que la complejidad.

El patrimonio antes que el consumo.

La calidad antes que la cantidad.

La sostenibilidad antes que la velocidad.

La confianza antes que cualquier algoritmo.

---

## Qué nunca seremos

Nunca seremos una plataforma diseñada para fomentar compras.

Nunca utilizaremos técnicas manipulativas.

Nunca priorizaremos el beneficio económico del producto frente al beneficio del usuario.

Nunca sacrificaremos nuestros principios para añadir funcionalidades.

Nunca convertiremos la colección en una competición.

---

## Qué aspiramos a ser

Queremos ser el compañero estratégico que todo coleccionista desearía tener.

Un sistema capaz de aportar serenidad.

Capaz de reducir errores.

Capaz de explicar cada decisión.

Capaz de proteger el patrimonio del usuario.

Capaz de recordar que disfrutar del camino es tan importante como completar la colección.