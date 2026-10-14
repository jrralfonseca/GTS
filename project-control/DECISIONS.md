# GTS — Decisiones de arquitectura

Este fichero recoge las decisiones que **condicionan el código** y que alguien
encontraría más tarde y leería como errores. Cada una dice qué se decidió, por qué,
y qué se paga a cambio.

Las decisiones ya escritas en el README (D-015 a D-020) y en `memory-graph.jsonl`
**no se duplican aquí**: van referenciadas. Este fichero solo recoge lo que
decide esta auditoría.

---

## D-021 — El mapa maestro vive en `project-control/`, y no duplica el grafo

**Decisión:** el inventario estructurado de componentes, problemas, evidencias y
roadmap vive en `project-control/`, no en `memory-graph.jsonl`.

**Por qué:** el grafo de conocimiento guarda *hechos sobre el sistema* —qué es un
bug, qué se aprendió, qué se decidió— en prosa con observaciones. Es bueno
contando historias y malo como inventario: no tiene campos, no se puede consultar
por estado, y una entrada de 4 000 caracteres no dice si un componente está
validado.

**Qué se paga:** hay dos lugares donde vive el conocimiento. La mitigación es
explícita:

| Artefacto | Es la fuente de verdad DE | Nunca contiene |
|---|---|---|
| `memory-graph.jsonl` | Los hechos narrativos: bugs, lecciones, decisiones, experimentos | Estados de componentes, inventarios |
| `project-control/COMPONENTS.yaml` | Estado, validación, operación, rutas, tests por componente | Narrativa, contexto histórico |
| `README.md` | El índice legible para una persona | Estados estructurados |
| `SYSTEM_MAP.md` | La arquitectura y las divergencias | Detalle por componente |

**Cómo se sostiene:** `EVIDENCE.yaml` es la única capa que se regenera con un
comando. Si un componente cambia de estado, se actualiza `COMPONENTS.yaml` y su
`evidencias`; la narrativa se actualiza en el grafo. Nunca al revés.

---

## D-022 — Los tres estados se mantienen separados, y "No evaluado" es una respuesta válida

**Decisión:** cada componente tiene `implementacion`, `validacion` y `operacion`
como campos independientes, y `No evaluado` es un valor permitido.

**Por qué:** el defecto que hace un mapa inútil es tratar "el código existe" como
"esto funciona". Un motor de backtesting puede estar implementado, validado por
tests, y nunca haberse ejecutado sobre un resultado real. Los tres son preguntas
distintas con tres respuestas distintas, y confundirlas convierte un inventario
en una lista de deseos.

**Ejemplo medido hoy:** `VAL-004` (Monte Carlo) tiene código, no tiene test, y
tiene `operacion: No evaluado` — aunque los ficheros en disco traigan
`mc_metrics`. El bloque existe, lo que no significa que se haya ejecutado con
contenido real. Sin esta separación, ese componente se habría marcado "hecho".

**Consecuencia asumida:** el mapa tiene más entradas "No evaluado" de las que
gustarían. Es el precio de no mentir.

---

## D-023 — La auditoría re-verifica antes de repetir

**Decisión:** la auditoría del 2026-10-09 (`audit/`, borrada del árbol pero
recuperable en `HEAD`) se usó como **hipótesis**, no como resultado. Cada hallazgo
se comprobó contra el código actual antes de escribirlo.

**Por qué:** entre la auditoría anterior (base `32b6f8e`) y hoy han entrado dos
commits (`44f8591`, `e801d4e`) y 36 ficheros de trabajo sin commitear. Copiar un
informe caduco es peor que no tener informe: da una confianza que no se ha ganado.

**Lo que cambió al re-verificar:**

| Hallazgo de 2026-10-09 | Estado hoy |
|---|---|
| H1 — la suite del bot no corre (2 imports, 4 firmas) | **CERRADO.** Los 2 módulos coleccionan (16 tests) |
| H2 — `sweep-H4.json` no existe | **SIGUE ABIERTO.** Peor: ahora hay un barrido H4 *vacío* en disco |
| H3 — no existe tramo fuera de muestra | **SIGUE ABIERTO.** Confirmado por el propio fichero |
| H4 — el panel puede apagar los validadores | **SIGUE ABIERTO** (R-002) |
| H5 — `--dry-run` no es un interruptor | **PARCIAL.** `dry_run` ahora significa "qué libro" y se deriva de `live.enabled` |
| H6 — `signal_value` no existe | **OBSOLETO.** El campo existe hoy. Lo que queda por verificar es si los topes se aplican |
| H7 — el test de conformidad no ejerce el motor | **SIGUE ABIERTO** (R-007) |
| H8 — métricas sin pintar, sin alertas | **SIGUE ABIERTO** (R-012, R-013) |

**Qué se paga:** la auditoría de hoy repite trabajo que la anterior ya había
hecho. Se acepta: el coste de un informe caducado es mayor que el de volver a
mirar.

---

## D-024 — Las cifras se reportan con su N, o no se reportan

**Decisión:** ningún número de este mapa aparece sin su denominador. Un Sharpe sin
el número de folds que lo produjo, o un p-valor sin el tamaño del universo, no se
escribe.

**Por qué:** el sistema ya pago por esto. `README.md:531` documenta por qué un
`sharpe_mean` sin N es ininterpretable: con N ensayos, el máximo esperado es
`sqrt(2 ln N)/T` aunque todas las estrategias valgan cero en población. El p-valor
de 0,998 que este mapa cita no significa "98 % de probabilidad de ser malo":
significa que el máximo de 25 configuraciones no se distingue del ruido.

**Ejemplo aplicado hoy:** el Sharpe fuera de muestra del modelo se reporta como
`-0,064 ± 1,363 sobre 28 folds`, no como "-0,064". La segunda forma se lee como
"casi cero"; la primera se lee como "no lo sé", que es lo cierto.

---

## D-025 — Un resultado negativo es un resultado, no un fallo

**Decisión:** los tres p-valores (0,6988 / 0,9980 / 0,2715), el Sharpe OOS de
−0,064, y la confirmación refutada de MA_Crossover se registran como **evidencia
válida**, no como trabajo pendiente.

**Por qué:** el criterio preinscrito del experimento `reality-check` dice, en el
propio README: *"p-valor del Reality Check de White por debajo de 0,05. Por encima
de 0,05 es un resultado legítimo y publicable."* El sistema escribió esa frase antes
de correr el experimento, y el experimento salió por el otro lado.

Marcar esto como "fallo" sería confundir el estado del código con el estado del
mundo. El código funciona: corrige, y la corrección dice que no hay ventaja. Lo
que falta es encontrar una que la sobreviva.

**Consecuencia asumida:** `ECO-001` tiene `implementacion: existe`,
`validacion: probada`, `operacion: "MEDIDA: NEGATIVA"`. Es la única entrada del
mapa con ese estado, y es correcta.

---

## D-026 — `project-control/` es un registro, no un sitio donde Meter

**Decisión:** estos ficheros se actualizan **después** de cada tarea, no durante.
El mapa describe un estado; un estado a medio cambiar no es un estado.

**Por qué:** la auditoría anterior sufrió exactamente este problema. Su propio
informe abre con una tabla de mtimes: *"durante esta auditoría el propietario
estaba editando el repositorio en paralelo"*, y marks media la mitad de sus cifras
como `PROVISIONAL` por eso. Documentar un sistema mientras se modifica produce un
mapa que miente con la mayor precisión posible.

**Cómo se aplica en el trabajo diario:**
1. Antes de empezar: leer `ISSUES.yaml` (lo abierto) y `ROADMAP.yaml` (lo siguiente).
2. Durante: implementar y probar. El mapa no se toca.
3. Después: actualizar el componente afectado en `COMPONENTS.yaml`, la evidencia
   nueva en `EVIDENCE.yaml`, el problema cerrado en `ISSUES.yaml`.
4. Si hubo una decisión que condiciona código: añadirla aquí.

---

## D-027 — `audit/` no se restauró sin preguntar

**Decisión:** los 220 ficheros de la auditoría del 2026-10-09 están borrados del
árbol (preparados para commit) y esta auditoría **no los restauró**.

**Por qué:** restaurarlos resucitaría conclusiones caducadas junto con las
vigentes — H6 dice que el optimizador de cartera está roto por un campo que hoy
existe. Quien lea ese informe sin saber que es de septiembre tomará una decisión
equivocada. La alternativa honesta es dejar la evidencia en el historial de git
(`git show HEAD:audit/<dominio>/<fichero>.md`) y que este mapa sea el vigente.

**Qué se paga:** si el dueno quería conservar esos informes como documento de
trabajo, hay que restaurarlos con una marca de "revisado el 2026-10-10". La
decisión es suya (R-016).

**Aviso relacionado:** **no ejecutar `git clean` ni `git checkout` en este árbol
sin revisar.** Hay 220 eliminaciones preparadas y 96 líneas más
(`brokers/exness.yaml`) que un checkout destruiría de forma irreversible.

---

## D-028 — `gts/` y `descarga/` no son fuente de verdad

**Decisión:** el bundle de entrenamiento en `gts/` se trata como **artefacto
construido**, no como código fuente. No se corrigió durante esta auditoría.

**Por qué:** está en `.gitignore:100` (`/gts/`), contiene un `BUNDLE_VERSION.txt`
que declara su commit de origen (`44f8591`), y sus ficheros `.py` tienen mtime
09:31 frente a los 15:55 de la raíz. Es una copia de distribución para Kaggle y
Colab. Sus 13 ficheros divergentes no son un conflicto de merge: son la foto de un
commit anterior.

**Por qué importa igual:** esa copia es la causa de los 3 tests rojos (ISS-001).
Que no sea código fuente no la exime de contaminar `sys.path` cuando un test la
extrae. Por eso R-001 es tarea de prioridad 1 pese a ser "solo un arnés".

---

## D-029 — El mapa no declara ninguna estrategia apta para producción

**Decisión:** ningún componente de `COMPONENTS.yaml` tiene estado "apto para
producción", y ninguna tarea del roadmap propone promover una regla o un modelo.

**Por qué:** no hay evidencia que lo sostenga. El registro de experimentos tiene
**cero pares confirmados**; los tres barridos dan p > 0,05; la única
confirmación ejecutada fue refutada; y el sistema **no puede colocar órdenes
reales** (`live.enabled: false`), de modo que ninguna cifra de este repositorio
es una cifra de dinero.

**Consecuencia asumida, y es la buena:** el sistema está parado por su propia
regla —`evidencia_confirmada()` devuelve vacío y el bot lanza `RuntimeError`— y eso
es exactamente lo que un sistema que se respeta a sí mismo debería hacer cuando
no ha encontrado nada que operar. El mapa documenta ese estado con precisión en
lugar de disimularlo.

---

## D-030 — La prioridad no la da el número de tests

**Decisión:** la cobertura numérica (1384 tests) **no** se trata como evidencia
de fiabilidad. `ROADMAP.yaml` ordena por dependencia y por riesgo, no por
cantidad.

**Por qué:** de los 1384 tests, cinco son guardianes de la evidencia que **no
corren nunca** (ISS-006), el único test que valida la conformidad de costes no
ejercita ninguno de los dos motores (ISS-007), y `portfolio_optimizer` —activo
en el YAML— no tiene ninguno (ISS-008). El único componente sin vallas es
precisamente el único que nadie ha mirado.

**Consecuencia asumida:** el roadmap empieza por un arnés de tests (R-001), no
por lógica de trading, porque sin él no se puede medir nada de lo demás.

---

## D-031 — El panel no ejecuta análisis del bot: lee sus artefactos o se retira

**Decisión:** los endpoints `/api/learning` no construyen un `LearningLoop`
del bot dentro del proceso del panel. `GET /api/learning/report` lee
`data/learning/report.json` —el artefacto que escribe el bot cuando corre
SU análisis— y responde 404 con el motivo mientras no exista. `POST
/api/learning/run` se retira con 501: si algún día hace falta arrancar el
análisis desde la interfaz, el camino es un trabajo de la cola del bot, no
un import cruzado.

**Por qué:** durante semanas esos endpoints hicieron `from app.learning
import LearningLoop`, que en el panel resuelve a un módulo inexistente: un
500 permanente que ningún test pisaba (ISS-028). Y de haber resuelto, habría
sido el panel ejecutando la lógica del bot — exactamente la excepción que
D-017 existe para impedir. La valla es AST (mira nodos de importación, no
texto): la primera versión del test se disparó a sí misma con el docstring
que explicaba el problema.

**Consecuencia asumida:** el panel no puede refrescar un reporte de
aprendizaje por su cuenta. El bot escribe el artefacto cuando corre su
análisis; mientras no lo corra, no hay reporte y el 404 lo dice con nombre
y apellido. La capacidad no se pierde: se muda al lugar donde debe
ejecutarse.

---

## D-032 — Una sola puerta de lectura de precios, y exacta: round_trip

**Decisión:** todo camino de MEDIDA lee las velas por `research/io.read_frame`
(con `float_precision="round_trip"`): el entrenamiento ML
(`training/train_universal.py`), el walk-forward (`scripts/walkforward_eval.py`)
y el holdout (`research/holdout_eval.py`). El feed del runtime
(`adapters/market_data/csv_feed.py`) lleva la misma bandera con el motivo
escrito, sin importar el paquete de research.

**Por qué:** el parser por defecto de pandas puede releer un double UN ULP
por debajo de lo escrito — medido: 25.650 valores del EURUSD_H1.csv cambiaban
un ulp, la cadena `1.2794699999999999` volvía como `1.27947` (otro double), y
el `data_hash` del fichero dejaba de cuadrar aunque el fichero fuera
exactamente el descargado (ISS-048). Por la puerta pasaban el hash y el
replay, pero el entrenamiento y el walk-forward leían por su cuenta: un
modelo ajustado sobre precios que nadie más puede releer iguales es una
medida irreproducible por construcción.

**Qué se paga:** `csv_feed` duplica la bandera en vez de importar la puerta.
Es deliberado: un adaptador del bot no debe depender del paquete de research
(la dirección de dependencia no se abre por ahorrar una línea). La
conformidad entre ambas lecturas la vigila la valla del hash y la
equivalencia serie/paralelo; si la puerta cambia de criterio, el
`test_cost_conformity` del mismo precio es donde se verá.

---

## D-033 — `dry-run` es un modo de ejecución, no un perfil

**Decisión:** `execution.profile` y `--dry-run` son **dos ejes distintos**.
El perfil (`research · backtest · paper · shadow · demo · live`) decide *qué
puerto de órdenes* existe; `--dry-run` decide *si ese puerto manda*. Un
`--dry-run` sobre `live` no llama nunca a `mt5.order_send()`. El perfil no
baja por el hecho de que el modo esté activo.

**Por qué:** durante días seouleurs "demo" y "dry-run" como sinónimos, y no
lo son. Un perfil `demo` **con** envío es una cuenta de práctica con dinero
virtual en riesgo; un perfil `live` **sin** envío es el rehearsal exacto de lo
que se hará mañana. Confundirlos produce dos errores opuestos y ambos caros:
creer que se puede gastar cuando no, o creer que no se puede gastar cuando sí.

**Qué se paga:** dos banderas donde bastaba una, y el panel tiene que explicar
las dos. Se acepta: el botón del panel es siempre `--dry-run`, y el perfil se
dice al lado.

---

## D-034 — LIVE exige dos llaves, y las dos a la vez

**Decisión:** el envío real exige `execution.live.enabled: true` **y**
`execution.live.allow_funded: true` **y** `account_info().trade_mode != DEMO`.
Si falta cualquiera de las tres, no hay puerto de órdenes.

**Por qué:** `live.enabled` solo se puede decir que sí por accidente — un
`git checkout` de una rama con el flag puesto, un YAML editado a deshora, un
`deep_merge` que sobrescribe lo que el fichero decía. `allow_funded` es una
afirmación deliberada y explícita de que hay dinero detrás, y `trade_mode` es
la comprobación que **el broker** hace, no el YAML. Dos afirmaciones locales y
una externa: si las tres se cumplen, ninguna puede fallar sola.

**Qué se paga:** `scripts/go_live.py --profile live --allow-funded` no basta
por sí mismo; el pre-flight se encarga de decir qué falta, con nombre. Es el
motivo de que el pre-flight devuelva un código de salida y no un log.

---

## D-035 — La fuente de señal es una, no una mezcla

**Decisión:** `app/signal.py::resolve()` devuelve **exactamente una** fuente:
`rules` **o** `model`, según `signal.source`. No hay fusión, ni ponderación,
ni *fallback* de una a otra.

**Por qué:** se creía que las reglas y el modelo actuaban a la vez. No: una
fusión sin pesos publicados no es auditable — cuando las dos señales discrepan,
no hay forma de saber cuál Weighed más, y por tanto no hay forma de explicar
una operación. Y un *fallback* silencioso es peor: si el modelo falla al
cargar, el sistema empezaría a operar con reglas sin que nadie lo supiera. Hoy
`resolve()` falla de forma explícita, que es lo correcto.

**Qué se paga:** dos sistemas que mantener para producir una decisión. Es
caro y es honesto. La alternativa —una fusión— sería más barata de mantener y
más cara de explicar.

---

## D-036 — El refresco lo hornea el servidor; el HTML nunca lo fija

**Decisión:** el intervalo de refresco vive en la cookie `gts-refresco`, la
validan `_REFRESCOS_VALIDOS = (1, 2, 5, 10, 30)`, y **el servidor lo pone en
el HTML** en el momento de servirlo. Un valor desconocido o manipulado se
ignora y se cae al defecto (2 s).

**Por qué:** los elementos que el operador está rellenando —el `<select>` de
"Fuente de señal", el campo de motivo, el `<details>` que los contiene— viven
dentro de `#bot-vivo`, que se re-pide cada N segundos. Dos salidas: subir el
intervalo a 30 s, o marcar esos elementos con `hx-preserve="true"`. Se han
hecho **las dos**, y se explican juntas: `hx-preserve` para lo que el
operador está usando ahora mismo, el intervalo largo para lo que no.

**Qué se paga:** una cookie de preferencia para una preferencia que casi
nadie cambia. Y una regla general que conviene tener escrita, porque es
fácil de romper sin querer: **un fragmento que se refresca no puede
contener un mensaje**. Los POST devuelven la pieza y el aviso por separado.

---

## D-037 — El camino a live se construye entero y se deja cerrado

**Decisión:** el camino a live está completo —perfiles, doble llave, gateway
MT5, pre-flight, aplanado, reconciliación, runbook— y `live.enabled` sigue en
`false`. El techo operativo es **demo**. La decisión la tomó el dueño el
2026-10-14 y es la correcta: el sistema no tiene ninguna evidencia
confirmada (`ECO-001`, resultado medido **negativo**), y poner dinero detrás
de un sistema que no ha encontrado nada que operar seríaография de vanidad.

**Por qué:** tener el camino listo y no recorrerlo son dos estados
compatibles. Lo que no es aceptable es un camino *a medio construir* que
parezca disponible. `scripts/go_live.py --profile live --allow-funded` está
ahí, y si alguien lo ejecuta el pre-flight le dirá exactamente qué falta.

**Qué se paga:** plumbing que hoy no ejecuta nada. Es el precio de que,
cuando llegue el momento —y el registro confirme una regla—, no haya que
inventar la infraestructura con el mercado abierto.

---

## D-038 — Un runbook con comandos que no existen es peor que ningún runbook

**Decisión:** las divergencias entre `RUNBOOK_LIVE.md` y el código **se
documentan como defectos** (`ISS-` en `ISSUES.yaml`), no se corrigen en
silencio ni se anotan como "pendiente de pulir". El documento queda
marcado **DRAFT** y la lista de las siete divergencias está en
`INFORME_LIVE_Y_MONITORING.md` §5.7.

**Por qué:** un runbook se sigue *durante* una sesión real, con prisa y con
dinero en la cuenta. `python -m gts flatten --force` no existe; quien lo
copie pierde minutos justo cuando no puede perderlos. Un documento que
promete comandos inexistentes no es un borrador: es una trampa con la
apariencia de un procedimiento.

**Qué se paga:** el runbook queda declarado no ejecutable, y `preflight_live`
—que sí existe y sí funciona— es de momento la única puerta real. Es un
estado peor que "todo bien" y mucho mejor que "todo bien, según el
documento".