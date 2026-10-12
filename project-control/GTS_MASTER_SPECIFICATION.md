# GTS — DEFINICIÓN MAESTRA DEL SISTEMA
## Propósito, arquitectura, comportamiento operativo y objetivos de desarrollo

**Proyecto:** GTS — Algorithmic Trading System  
**Objetivo:** Construir un sistema profesional de trading algorítmico, modular, verificable, seguro y preparado para evolucionar desde investigación y simulación hasta ejecución automatizada mediante MetaTrader 5.

---

# 1. DEFINICIÓN DEL SISTEMA

GTS es una plataforma de trading algorítmico diseñada para recibir datos del mercado, analizarlos mediante estrategias y modelos de decisión, evaluar el riesgo, gestionar operaciones y supervisar su ciclo de vida de principio a fin.

El sistema debe permitir desarrollar, evaluar y ejecutar estrategias de trading mediante una arquitectura compartida, en la que la lógica central de decisión y gestión de operaciones no dependa de un broker, de la interfaz web ni del entorno concreto de ejecución.

GTS debe evolucionar hacia un sistema capaz de operar con estrategias basadas en reglas, modelos de machine learning y combinaciones de ambos, siempre dentro de los permisos del perfil activo y sometido a controles de riesgo obligatorios.

No es simplemente un bot que genera señales y envía órdenes. Es un sistema completo que debe gestionar el proceso operativo desde la recepción de una barra de mercado hasta la apertura, mantenimiento, modificación, cierre y reconciliación de una posición.

Su arquitectura debe permitir investigar y desarrollar estrategias nuevas sin tener que reconstruir el motor de ejecución, y sustituir o actualizar componentes sin duplicar el ciclo de vida de una operación.

# 2. OBJETIVO PRINCIPAL

El objetivo principal de GTS es disponer de un sistema de trading algorítmico que:

1. Reciba y valide datos de mercado.
2. Calcule las features necesarias para la toma de decisiones.
3. Ejecute estrategias y modelos de decisión intercambiables.
4. Determine cuándo abrir, mantener, modificar o cerrar una posición.
5. Calcule el volumen apropiado según el capital, las condiciones del mercado y las reglas de riesgo.
6. Aplique controles obligatorios antes de autorizar una operación.
7. Gestione la ejecución, los fills, los rechazos, las órdenes pendientes y las posiciones.
8. Mantenga coherencia entre el estado interno y el estado del broker.
9. Registre los eventos operativos necesarios para reconstruir lo ocurrido.
10. Gestione errores, reconexiones, reinicios y situaciones inconsistentes de forma segura.
11. Permita probar nuevas estrategias y modelos sin poner en riesgo las operaciones reales.
12. Ofrezca supervisión y configuración mediante una interfaz de administración que no pueda eludir los controles críticos.

El criterio de éxito no es la cantidad de módulos, clases, tests o informes que produce el proyecto.

**El criterio de éxito es que GTS complete correctamente el ciclo operativo, respete los controles de riesgo, mantenga el estado coherente y permita verificar su comportamiento mediante ejecuciones reproducibles.**

La rentabilidad es un objetivo de investigación y evaluación posterior. No debe confundirse la corrección técnica del sistema con la demostración de una ventaja estadística en el mercado.

# 3. PRINCIPIOS FUNDAMENTALES

## 3.1. Un solo ciclo de vida operativo

GTS debe tener una única implementación principal del flujo de decisión y gestión de posiciones.

El recorrido normal del bot, la simulación, las pruebas de integración y los diferentes perfiles deben reutilizar los mismos casos de uso cuando corresponda.

No deben existir dos motores independientes que implementen la misma lógica de trading y evolucionen de forma diferente.

Los adaptadores pueden comportarse de manera distinta según el entorno, pero los contratos, las reglas de negocio y las responsabilidades deben estar claramente definidos.

## 3.2. Separación de responsabilidades

Cada módulo debe tener una función concreta.

La lógica de trading no debe depender directamente de la interfaz web, de una base de datos, de la biblioteca de MT5 ni de archivos particulares de investigación.

Los detalles externos se conectan mediante interfaces y adaptadores.

## 3.3. Seguridad por defecto

El sistema debe iniciar en un estado seguro.

Si faltan configuraciones obligatorias, no se puede verificar el estado del broker o existe una inconsistencia crítica, GTS debe impedir nuevas exposiciones en lugar de asumir que todo está correcto.

No debe habilitar el envío de órdenes como consecuencia de una configuración predeterminada, una variable ausente, una recuperación automática o un cambio de arquitectura.

Una orden ya ejecutada en el broker no desaparece porque el proceso se haya detenido. Por eso, bloquear nuevas órdenes no puede sustituir a la reconciliación de las posiciones y órdenes que ya existen.

## 3.4. Simplicidad funcional

Se debe mantener un monolito modular, no dividir el proyecto en microservicios sin una necesidad real.

No crear abstracciones, subsistemas, agentes de IA, capas o frameworks internos solamente porque parezcan profesionales.

Cada componente nuevo debe resolver una necesidad concreta del sistema.

## 3.5. Verificación proporcional

Las pruebas deben demostrar comportamientos reales.

Se prefieren pruebas pequeñas, deterministas y enfocadas en el flujo afectado. Las pruebas extensas de entrenamiento, optimización y backtesting no deben ejecutarse automáticamente después de cada cambio de arquitectura.

Cuando una prueba descubre un defecto, se corrige el defecto y se añade una comprobación proporcionada al riesgo.

# 4. ARQUITECTURA OBJETIVO

GTS debe organizarse en módulos con responsabilidades explícitas y dependencias unidireccionales.

## 4.1. Dominio y núcleo

Responsable de los conceptos fundamentales del sistema:

- Barras y datos de mercado normalizados.
- Símbolos e instrumentos.
- Especificaciones de los instrumentos.
- Señales y decisiones.
- Órdenes, ejecuciones y posiciones.
- Riesgo y autorización.
- Estados y transiciones del ciclo de vida.
- Costes y reglas de trading.

El núcleo no debe depender de MT5, del panel web, del almacenamiento concreto ni de componentes de investigación.

## 4.2. Puertos e interfaces

Definir contratos claros entre la lógica central y los servicios externos.

Entre los principales contratos se encuentran:

- `MarketDataPort`: obtención de datos de mercado.
- `OrderPort`: envío y seguimiento de órdenes.
- `PositionBookPort`: acceso y actualización del estado de posiciones.
- `JournalPort`: registro y consulta de eventos operativos.
- Interfaces para reloj, especificaciones de instrumentos y otros servicios externos cuando realmente sean necesarios.

Los contratos deben precisar tipos de datos, unidades, timestamps, errores posibles e invariantes.

## 4.3. Casos de uso

Los casos de uso coordinan el trabajo del sistema.

Como mínimo:

- `ProcessBar`: coordina el procesamiento de una barra de mercado.
- `ManagePosition`: administra la posición y sus condiciones de salida.
- `AuthorizeTrade`: aplica los controles de autorización y riesgo.
- `Reconcile`: compara el estado esperado con el estado observado.

Estos casos de uso deben utilizarse en el recorrido operativo real. No deben quedarse como clases aisladas que solo ejecutan las demostraciones.

## 4.4. Adaptadores

Implementan las interfaces para cada entorno.

Entre ellos:

- Feed de mercado MT5.
- Adaptador de ejecución MT5 cuando esté implementado y validado.
- Adaptador de simulación.
- Libro de posiciones en memoria o persistente, según el entorno.
- Adaptadores de almacenamiento y journal.
- Reloj controlado y feeds deterministas para pruebas.

El núcleo no debe necesitar conocer la implementación concreta para ejecutar sus reglas de negocio.

## 4.5. Runtime y orquestación

El runtime administra el proceso operativo:

- Arranque y verificaciones previas.
- Construcción y conexión de dependencias.
- Lectura de las barras.
- Invocación del ciclo de vida.
- Control del perfil activo.
- Gestión de errores y reconexiones.
- Reconciliación periódica.
- Pausa, parada y recuperación.
- Supervisión y apagado seguro.

El runtime debe invocar los casos de uso centrales, no reimplementar las reglas de decisión.

## 4.6. Estrategias y modelos

Los componentes de decisión deben ser intercambiables.

GTS debe poder trabajar con estrategias basadas en reglas, modelos de machine learning o mecanismos de combinación de señales, siempre que estén implementados y configurados de manera explícita.

El motor de ejecución no debe depender de una estrategia específica ni de un framework de entrenamiento particular.

## 4.7. Riesgo

La gestión del riesgo debe constituir una capa independiente, obligatoria y verificable.

Debe evaluar, según corresponda:

- Tamaño de posición.
- Capital y exposición.
- Límites de operaciones y posiciones.
- Volatilidad y condiciones de mercado.
- Distancia y validez de los stops.
- Riesgo acumulado.
- Restricciones del instrumento y del broker.
- Estado operativo, pausas y límites de seguridad.

Los validadores obligatorios no pueden desactivarse desde el panel ni mediante una estrategia, un modelo o una configuración ordinaria.

## 4.8. Panel web y API

La interfaz web debe permitir supervisar y administrar el sistema, consultar estados, visualizar decisiones, revisar operaciones y modificar las configuraciones autorizadas.

El panel no debe contener una segunda implementación del motor de trading ni tener autoridad implícita para modificar el estado de ejecución, eliminar restricciones obligatorias o enviar órdenes por caminos alternativos.

La API debe validar permisos y configuraciones en el servidor. No se debe depender de que la interfaz simplemente oculte un botón.

# 5. CICLO DE VIDA COMPLETO DE UNA OPERACIÓN

Este flujo es uno de los requisitos más importantes de GTS.

## Fase 1. Recepción de datos

El feed obtiene los datos de mercado y verifica que sean válidos.

Debe controlar, según corresponda:

- Instrumento y temporalidad.
- Barra cerrada frente a barra en formación.
- Orden cronológico.
- Duplicados y retrocesos temporales.
- Valores inválidos o incoherentes.
- Desconexiones y errores de lectura.

Los timestamps deben tener una semántica inequívoca. Si el proveedor informa la hora de apertura y GTS utiliza la hora de cierre, ambas deben distinguirse y convertirse correctamente.

## Fase 2. Preparación de features

El sistema calcula las features requeridas por la estrategia o el modelo.

Debe evitar datos futuros, desplazamientos incorrectos, uso accidental de una barra todavía abierta y diferencias inexplicadas entre entrenamiento e inferencia.

Las transformaciones de datos deben respetar el contrato de cada modelo.

## Fase 3. Evaluación de estrategias

Las estrategias y modelos evalúan las condiciones de mercado y generan señales.

Una señal no equivale a una orden. Es una entrada para el proceso de decisión.

Las estrategias deben poder sustituirse o combinarse sin alterar el ciclo operativo común.

## Fase 4. Decisión

El sistema considera la señal, la posición actual, las condiciones de mercado y las reglas de transición para decidir si debe:

- Mantenerse sin cambios.
- Abrir una posición.
- Cerrar una posición.
- Revertir una posición, si está permitido.
- Modificar una posición o sus protecciones.
- Rechazar la acción.

La decisión debe poder explicarse mediante datos y motivos registrados.

## Fase 5. Gestión de riesgo y volumen

Se calcula el volumen inicial según las reglas aplicables.

Después se verifica el volumen definitivo y se ejecutan los controles obligatorios.

El volumen no puede incrementarse después de la autorización sin volver a validar el riesgo.

La existencia de una señal válida nunca debe permitir saltarse los controles de seguridad.

## Fase 6. Autorización

El sistema decide si la acción puede continuar.

La autorización debe considerar el perfil activo, los permisos de envío, la configuración, los controles de riesgo y el estado operativo.

Una decisión aprobada por el motor no implica automáticamente que el sistema tenga permiso para enviarla al broker.

## Fase 7. Ejecución

Cuando el perfil y el entorno permitan enviar órdenes, el adaptador de ejecución deberá gestionar las respuestas del broker.

Debe contemplar:

- Aceptación.
- Rechazo.
- Orden pendiente.
- Ejecución completa.
- Ejecución parcial.
- Cancelación.
- Error o pérdida de conexión.
- Respuesta incierta después de un timeout.

La idempotencia debe impedir que una retransmisión causada por un error o una reconexión genere órdenes duplicadas.

## Fase 8. Gestión de la posición

El sistema debe mantener una representación coherente de la posición.

Debe controlar las condiciones de salida, las modificaciones, los stops, los objetivos, el volumen restante y las transiciones de estado.

No debe asumir que una solicitud de cierre equivale a un cierre confirmado.

## Fase 9. Reconciliación

El estado interno debe compararse regularmente con el estado observado en el broker.

No basta con hacerlo una vez al arrancar.

Si existen órdenes o posiciones desconocidas, diferencias de volumen, ejecuciones parciales o estados incompatibles, el sistema debe entrar en un estado seguro, impedir nuevas exposiciones cuando corresponda y resolver la discrepancia mediante el procedimiento definido.

## Fase 10. Registro operativo

El sistema debe registrar los eventos nuevos necesarios para reconstruir el recorrido de una decisión y una operación.

El registro debe distinguir señales, decisiones, autorizaciones, órdenes, ejecuciones, modificaciones, cierres, rechazos, errores y reconciliaciones.

El registro inicial puede estar vacío. No debe exigir auditorías históricas ni resultados antiguos para permitir el arranque.

## Fase 11. Parada y recuperación

Una parada o un reinicio deben tratarse como transiciones operativas explícitas.

GTS debe recuperar o volver a verificar su estado antes de permitir nuevas operaciones.

Si el proceso desconoce si una orden pudo ejecutarse antes de una desconexión, no debe retransmitirla ciegamente.

La pausa del operador y los bloqueos críticos deben conservarse o restaurarse mediante un procedimiento seguro y definido.

# 6. PERFILES DE EJECUCIÓN

Los perfiles representan capacidades operativas, no simples nombres que cambian la interfaz.

## RESEARCH

Permite investigar estrategias, preparar datos y ejecutar experimentos sin acceso a la ejecución real.

## BACKTEST

Permite simular estrategias sobre datos históricos, respetando las reglas temporales y los costes modelados.

Los resultados de los backtests no deben confundirse con resultados reales de ejecución.

## PAPER

Permite practicar el flujo operativo mediante ejecución simulada. No tiene permiso para enviar órdenes externas.

## SHADOW

Consume datos reales del mercado, calcula features, genera decisiones, evalúa riesgo y registra resultados, pero no envía órdenes.

Debe existir una barrera efectiva contra el envío, no únicamente una condición que el bucle espera respetar.

## DEMO

Representa la integración operativa con una cuenta de demostración del broker.

Solo puede habilitarse cuando existan el adaptador de órdenes y los controles necesarios, y cuando la configuración identifique explícitamente el entorno de demostración.

## LIVE

Representa la ejecución en una cuenta con dinero real.

Debe permanecer deshabilitado hasta validar el adaptador, los permisos, la reconciliación, los límites de riesgo, las recuperaciones y las condiciones de parada.

La activación debe ser explícita y no puede ocurrir por defecto.

## Reglas comunes

Los perfiles deben tener capacidades inequívocas.

El tipo de perfil debe ayudar a impedir combinaciones peligrosas, pero la seguridad no debe depender exclusivamente de comprobaciones de tipos durante la construcción del objeto: las barreras críticas también deben proteger el camino de ejecución efectivo.

Los modos `ENFORCE`, `OBSERVE` y `BYPASS`, cuando estén implementados, deben respetar los permisos del perfil. `BYPASS` no puede hacer que un perfil sin capacidad de envío obtenga esa capacidad, ni permitir que DEMO o LIVE evadan controles obligatorios.

El modo activo y los permisos efectivos deben poder inspeccionarse y comprobarse desde el propio sistema.

# 7. DATOS, INVESTIGACIÓN Y MACHINE LEARNING

GTS debe permitir investigar estrategias y modelos, pero esa capacidad no debe bloquear ni contaminar el ciclo operativo básico.

## Datos

El sistema debe distinguir entre datos de mercado, fixtures de pruebas, datos sintéticos, resultados de experimentos y registros operativos.

Los datos sintéticos utilizados para verificar el funcionamiento no deben presentarse como datos reales ni alimentar accidentalmente una ejecución ordinaria.

## Investigación

Debe ser posible comparar estrategias, evaluar hipótesis y reproducir experimentos cuando se trabaje expresamente en investigación.

Los resultados de investigación deben tener procedencia y contexto suficientes para saber qué código, configuración y datos los generaron.

## Machine learning

La arquitectura debe permitir integrar entrenamiento, evaluación e inferencia de modelos sin acoplar el runtime a una implementación concreta.

El uso de un modelo requiere una configuración válida, los artefactos realmente necesarios y una estrategia de inferencia implementada.

Si una fuente de señales, como `model`, no está soportada, el sistema debe declararlo expresamente y no simular que esa capacidad funciona.

La investigación y el entrenamiento deben poder evolucionar por separado de la gestión de órdenes y posiciones.

## Evaluación

Una arquitectura funcional no demuestra que la estrategia tenga rentabilidad.

La rentabilidad debe evaluarse posteriormente mediante un proceso adecuado, separado de las pruebas de integración, y con atención a la fuga temporal, la selección de modelos, los costes y la generalización fuera de muestra.

# 8. LIMPIEZA Y COMIENZO DESDE CERO

La migración debe dejar GTS preparado para iniciar una nueva etapa de pruebas sin arrastrar resultados generados anteriormente.

No se deben conservar por defecto:

- Auditorías históricas.
- Journals antiguos.
- Logs de ejecuciones anteriores.
- Historiales de barras generados para experimentos previos.
- Resultados de backtesting, replay, barridos o walk-forward.
- Checkpoints y artefactos de entrenamientos antiguos que no se hayan autorizado expresamente.
- Archivos temporales, salidas de demostraciones y datos residuales.

No dedicar la migración a clasificar o recuperar el pasado.

La limpieza no implica eliminar código funcional, dependencias, configuraciones esenciales, esquemas o artefactos necesarios para la ejecución vigente.

Los fixtures nuevos deben ser pequeños y reproducibles. Los registros operativos nuevos deben comenzar limpios y acumular únicamente eventos de las nuevas ejecuciones.

El sistema no debe exigir evidencia histórica de experimentos para poder arrancar. Sí debe exigir las configuraciones y dependencias operativas que sean necesarias para el perfil activo.

# 9. PRUEBAS Y CRITERIOS DE CALIDAD

Las pruebas tienen un propósito concreto: verificar que GTS hace lo que debe hacer.

Las comprobaciones deben concentrarse en:

- Arranque y validación de configuración.
- Contratos de datos.
- Procesamiento de una barra.
- Decisión y autorización.
- Gestión del riesgo.
- Ejecución simulada.
- Fills completos y parciales.
- Gestión de posiciones.
- Reconciliación.
- Recuperación y pausa segura.
- Permisos de cada perfil.
- Ausencia de rutas de ejecución no autorizadas.
- Integración entre runtime, API y panel cuando corresponda.

El objetivo no es maximizar el número de pruebas ni ejecutar siempre la suite completa.

Las pruebas largas deben reservarse para situaciones en las que aporten una verificación adicional justificada, por ejemplo antes de una integración importante.

Los errores de infraestructura de pruebas, como conflictos de directorios temporales, deben identificarse como tales y no presentarse automáticamente como defectos del sistema.

No se deben ejecutar múltiples procesos de pruebas que compartan un mismo `--basetemp`.

Cada entrega debe distinguir:

- Implementado.
- Ejecutado.
- Verificado.
- Pendiente.
- No verificable en el entorno disponible.

No declarar completada una funcionalidad porque exista una interfaz, una clase o una prueba aislada.

# 10. PANEL Y CONTROL OPERATIVO

El panel es una herramienta de supervisión y administración, no una autoridad paralela sobre el motor de trading.

Debe poder mostrar, según las funcionalidades realmente implementadas:

- Estado general del runtime.
- Perfil activo y capacidades.
- Conexión y estado de los datos.
- Señales y decisiones recientes.
- Resultado de las evaluaciones de riesgo.
- Órdenes y posiciones.
- Errores, pausas y reconciliaciones.
- Métricas operativas y resultados.

La interfaz no debe modificar silenciosamente límites críticos ni desactivar validadores obligatorios.

Los controles administrativos deben tener autorización adecuada, validación en el servidor y trazabilidad.

La aplicación no debe ofrecer controles que aparenten estar operativos cuando todavía no lo están. Las funcionalidades incompletas deben identificarse con claridad.

# 11. MÉTODO DE DESARROLLO

La IA encargada del proyecto debe actuar como responsable técnico de la implementación, no como un generador indiscriminado de código.

Antes de modificar una parte del sistema:

1. Inspeccionar su implementación actual y sus consumidores.
2. Identificar la causa concreta del problema.
3. Determinar el cambio mínimo que resuelve el problema.
4. Verificar las dependencias y los contratos afectados.
5. Implementar el cambio.
6. Ejecutar las comprobaciones pertinentes.
7. Revisar que no se haya creado una segunda implementación ni roto un camino operativo existente.
8. Informar de los resultados reales.

No reconstruir módulos que ya funcionen solo para ajustarlos a una nueva estructura teórica.

No eliminar tests que detectan defectos reales para conseguir una suite verde. Cuando se retire un componente, se debe revisar si sus protecciones siguen cubriendo el comportamiento correspondiente en la nueva arquitectura.

No iniciar fases adicionales antes de terminar la tarea actual y exponer el resultado.

No hacer modificaciones destructivas o commits sin la autorización correspondiente.

# 12. ESTADO ACTUAL REPORTADO Y TRABAJO PENDIENTE

Esta sección refleja la última información comunicada sobre el proyecto; la IA debe comprobar su vigencia antes de usarla como diagnóstico definitivo.

> **Actualizada el 2026-10-11**, tras verificación directa del árbol en `ffbe39f` más el trabajo sin commitear: suites ejecutadas, cableado leído y barreras probadas. Lo marcado como verificado se comprobó en esa fecha; lo marcado como reportado proviene de sesiones anteriores y no se ha vuelto a ejecutar.

## Avances verificados el 2026-10-11

- La arquitectura de puertos, casos de uso y adaptadores está implementada y conectada al `TradingLoop` de producción: `ProcessBar`, `AuthorizeTrade`, `ManagePosition` y `Reconcile` operan por barra y por símbolo, con reconciliación antes de aumentar exposición.
- Las suites del bot y del panel están en verde con corrida propia: bot 1461 coleccionados, 1382 pasan, 0 fallan, 79 saltos con motivo declarado, exit 0; panel 411/411, exit 0. El orden adverso que reproducía los fallos históricos del arnés sale limpio.
- El procesamiento determinista de una barra cerrada por el recorrido normal (`run_once` → `feed.poll`) está cubierto y en verde: una barra nueva llega una vez y solo una, la ya procesada no se reentrega, y una reconexión que sirve la misma vela no produce una segunda decisión. SHADOW no envía en ese recorrido.
- La coherencia entre el perfil declarado y el libro real es obligatoria en el cargador (ISS-032, resuelto): un perfil con capacidad de envío exige `live.enabled: true` y viceversa, `--dry-run` sobre una configuración live se niega a arrancar en `run` y en `once`, y el arranque registra los permisos efectivos completos.
- La barrera de envío es triple y verificada: solo `live/terminal.py` puede escribir en el broker (valla por AST), la fábrica de puertos construye vía solo para DEMO y se niega para LIVE, y el bucle exige capacidad Y puerto; todo lo demás cablea `NoSendBroker`, que lanza `PermissionError`.
- La limpieza de la nueva etapa está en vigor y verificada: `data/research/` vacío, journal nuevo a cero, bundle de entrenamiento `gts/` eliminado; los tests que usaban esos artefactos saltan con motivo declarado.

## Estado del adaptador de órdenes MT5 (corrige la versión anterior)

- La afirmación anterior —"el adaptador de órdenes MT5 todavía no está implementado"— **estaba desactualizada**. `MT5OrderPort` existe y está cableado en la raíz de composición solo para el perfil DEMO, con idempotencia por `client_id` y protocolo de `comment` ante envíos ambiguos; LIVE se rechaza en la fábrica con mensaje explícito.
- Su estado correcto es: **implementado y cubierto por tests unitarios; NO verificado contra un broker real**. La primera orden DEMO la envía el operador, con `project-control/RUNBOOK_DEMO.md` delante.
- La receta de activación de DEMO son dos líneas: `execution.profile: demo` + `execution.live.enabled: true` — con lo que entran la puerta de evidencia, el libro del broker, la lista de símbolos permitidos y el presupuesto de live (0,25 %).
- El envío de órdenes permanece deshabilitado en la configuración distribuida (`research` + `enabled: false`), y LIVE no obtiene vía de envío.

## Pendientes identificados (actualizados)

1. ~~Confirmar el resultado definitivo de la regresión del cableado~~ **Resuelto el 2026-10-11**: suite del bot en verde con corrida propia, incluido el orden adverso que reproducía los fallos históricos.
2. ~~Probar el procesamiento normal de una nueva barra cerrada mediante el polling del runtime~~ **Resuelto en su parte determinista** (tests en verde); queda ejecutar `scripts/polling_mt5.py` contra MT5 real con el mercado abierto.
3. Ejecutar `scripts/shadow_mt5.py` contra el terminal real y repetir la verificación SHADOW con registro propio. La verificación SHADOW previa es un reporte de una sesión anterior, no una corrida de esta etapa.
4. Mantener separados los datos sintéticos, los históricos y los registros operativos — **verificado como estado vigente**; sigue siendo un invariant que conservar, no una tarea puntual.
5. ~~Revisar la configuración por perfiles y asegurar que sus permisos coincidan con las capacidades reales~~ **Resuelto el 2026-10-11** (ISS-032: coherencia obligatoria en el cargador, `--dry-run` honesto, permisos inspeccionables al arranque).
6. Verificar el adaptador de órdenes MT5 contra una cuenta demo real, en etapa controlada y con el runbook: la implementación existe; falta la verificación operativa, y la primera orden es del operador.
7. Verificar entrenamiento, evaluación y barridos con datos apropiados antes de sacar conclusiones estadísticas. Nota de esta etapa: `data/research/` está vacío por diseño, así que cualquier cifra de investigación previa queda sin soporte en disco y debe re-emitirse antes de citarse.

No debe afirmarse que ningún punto está resuelto sin comprobar el estado actual del repositorio.

## Alcance global e inventario de capacidades (evaluación del propietario, 2026-10-11)

Esta tabla es el **alcance global acordado para la etapa**. La clasificación es la del propietario, hecha a partir de los informes verificados de la sesión del 2026-10-11, y manda como estado de referencia hasta que una medición nueva la sustituya:

- **Verificado** — el código existe y las pruebas reportadas confirman su comportamiento dentro del alcance comprobado.
- **Parcial** — está implementado, pero falta verificarlo en el entorno real o completar partes del flujo.
- **Pendiente** — la capacidad no está implementada o falta una parte fundamental para considerarla completa.
- **Sin verificar** — no hay evidencia suficiente después de la limpieza para afirmar que funciona de extremo a extremo.
- **No demostrada** — afirmación que requiere un proceso de investigación que todavía no existe.

| # | Área de GTS | Estado | Qué falta |
|---|---|---|---|
| 1 | Arquitectura central | Verificado | No requiere otro rediseño general. Mantener la separación y evitar duplicar lógica. |
| 2 | Runtime y ciclo de vida | Verificado en pruebas | Ya se conecta `TradingLoop` con `ProcessBar`. Falta validar más escenarios externos de extremo a extremo. |
| 3 | Datos de mercado MT5 | Parcial | Probar la llegada de una nueva barra H1 cerrada en tiempo real. La lectura de históricos ya se comprobó. |
| 4 | Features e indicadores | Verificado en el camino de reglas | Confirmar el comportamiento con el flujo completo y las configuraciones definitivas de cada estrategia. |
| 5 | Estrategias basadas en reglas | Verificado en pruebas | No se ha demostrado que las reglas actuales sean rentables. |
| 6 | Decisiones de trading | Verificado en pruebas | Completar la validación en escenarios operativos externos y ante estados inesperados del broker. |
| 7 | Gestión del riesgo | Verificado en pruebas | Falta validar el comportamiento combinado con órdenes y especificaciones reales del broker. |
| 8 | Dimensionado de posiciones | Verificado en pruebas | Comprobar en DEMO volumen, valor del contrato, stops y restricciones del instrumento. |
| 9 | Ejecución de órdenes MT5 | Parcial | `MT5OrderPort` tiene pruebas unitarias, pero falta la ejecución controlada en DEMO. |
| 10 | Gestión de posiciones | Parcial | La simulación se ha probado; falta verificar modificaciones, salidas y cierres mediante el broker real de demostración. |
| 11 | Reconciliación | Parcial | Probar con posiciones externas reales y después de desconectar y reiniciar el proceso. |
| 12 | Journal y persistencia | Parcial | Se comprobó una cadena temporal de 60 entradas, pero falta validar el registro operativo persistente durante una sesión y su recuperación. |
| 13 | Parada y recuperación | Parcial | Las pruebas unitarias pasan; falta el ensayo real de detener y reiniciar GTS con una posición abierta en DEMO. |
| 14 | Perfiles de ejecución | Parcial | Se verifican restricciones de perfiles; debe completarse la integración operativa de DEMO y conservarse LIVE bloqueado. |
| 15 | API y panel web | Pruebas reportadas en verde | Las pruebas del panel pasan, pero siguen pendientes las métricas y alertas que se habían identificado. |
| 16 | Machine learning: inferencia | Pendiente | `signal.source: model` no está implementado en el camino operativo reportado. |
| 17 | Entrenamiento de modelos | Sin verificar | Ejecutar y comprobar el flujo desde cero con datos y artefactos nuevos; los tests dependientes de artefactos anteriores se omiten. |
| 18 | Investigación y barridos | Sin verificar | Volver a verificar el proceso de investigación desde cero. Los resultados y artefactos históricos fueron retirados deliberadamente. |
| 19 | Backtesting y walk-forward | Sin verificar completamente | Confirmar que el flujo completo se ejecuta con nuevos datos, costes y separación temporal válida. |
| 20 | Rentabilidad de las estrategias | No demostrada | Investigación y evaluación estadística nuevas, sin fuga de información ni selección sobre el conjunto de prueba. |

Nota de la misma sesión: la **reestructuración del panel web**, para adaptarlo a esta etapa, es el siguiente paso acordado después de este alcance.

# 13. OBJETIVO FINAL DE GTS

GTS debe convertirse en una plataforma de trading algorítmico donde investigación, decisión, riesgo, ejecución y supervisión formen partes de un mismo sistema coherente.

La misma arquitectura debe poder utilizarse para probar estrategias, realizar backtests, procesar datos reales en SHADOW y, tras las verificaciones correspondientes, ejecutar órdenes mediante MT5 en DEMO y eventualmente LIVE.

Los módulos deben ser sustituibles, las dependencias deben ser controladas, las operaciones deben tener un ciclo de vida completo y el estado debe reconciliarse con la realidad del broker.

El sistema debe comenzar limpio cuando se inicia una nueva etapa, sin depender de resultados experimentales antiguos, y debe poder demostrar su comportamiento mediante comprobaciones reproducibles.

GTS no debe considerar una operación completada porque produjo una señal o porque una orden fue aceptada. Debe saber qué decidió, qué autorizó, qué se envió, qué se ejecutó, qué posición existe y si su estado interno coincide con el estado observado externamente.

**La meta final es un sistema de trading funcional, coherente, seguro y verificable. La rentabilidad se investiga y se demuestra por separado; nunca se presupone por el mero hecho de que la arquitectura funcione.**