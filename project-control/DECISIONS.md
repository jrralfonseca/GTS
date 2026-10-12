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