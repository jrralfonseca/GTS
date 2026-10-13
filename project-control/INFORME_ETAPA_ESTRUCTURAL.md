# Etapa estructural — componentes, verificación y estado honesto

**Fecha:** 2026-10-12 · **Autor:** agente de la etapa «estructura antes de
validación»
**Encargo:** completar la arquitectura, integrarla y cubrirla con pruebas
reproducibles, **sin** usar resultados históricos potencialmente contaminados
para diseñar, ajustar o aprobar nada.

**Nada de lo hecho aquí aprueba metodología, valida estrategia ni abre el
laboratorio.** Las dos metodologías siguen `implementada`, el registro no
confirma ninguna regla, y ningún barrido se ejecutó sobre período alguno.

---

## 1. El mapa de componentes: qué existe, qué se verificó

Cada componente del encargo, su estado y su valla. «Verificado» significa:
ejercitado por una prueba que se rompe si el componente miente.

| Componente | Estado | Valla / evidencia |
|---|---|---|
| Catálogo y resolución de metodologías (`research/metodologia.py`) | Integrado (sesión previa) | `test_guardia_cientifica.py` (54), `test_metodologia_criterio.py` (30): `seleccionar()` exige `aprobada`; ningún perfil lo está |
| Criterios: generación, persistencia, inmutabilidad (`research/criterios.py`, `preinscripcion.py`) | Integrado (sesión previa) | SHA-256 del contenido canónico; el criterio se escribe **antes** de mirar; `test_metodologia_criterio.py` |
| Preflight y validación de datos | Integrado + **corregido esta etapa** | El lector de CSV perdió precisión un tiempo (§2.3); la valla del hash la cazó y ya está corregida |
| Descubrimiento automático del universo | Integrado | El universo de `universal_v1` (11 reglas, 175 configuraciones) viaja en el perfil; el barrido lo cuenta todo (`test_sweep_universe.py` 19/19, sin saltos) |
| Registro y cola de trabajos (`jobs/`, `registro.py`) | Integrado (adopción `e706a3f`, config e informe ahora commiteados) | E2E real: trabajo `job-20261012-054047-d738cb` ejecutado por el worker en 0,6 s; `exigir_aislada()` separa pruebas de producción |
| Runners y pipelines (9 runners) | Integrado | `test_jobs_worker.py`, `test_contrato_del_pipeline.py`; cancelación cooperativa con puntos seguros |
| Laboratorio web | Integrado + **corregido** (§2.4: dos endpoints que petaban) | `test_learning_sin_boton.py` (4); el middleware 410 y `_no_dejar_entrar` de sesiones previas siguen verdes |
| Entrenamiento ML | Integrado + **corregido** (§2.1: `--resume`) | `test_reanudacion_entrenamiento.py` (9, veredictos invertidos al comportamiento correcto) |
| Walk-forward | Integrado, **limitación científica declarada** (§4.1) | El runner dice su límite en su propio resultado; el defecto de fondo es ISS-002, no software |
| Controles estadísticos (Reality Check de White) | Integrado + **vallas revividas** | `test_reality_check.py` (41, antes 33+saltos); el test pedagógico corre contra el barrido real |
| Costes: conformidad backtest↔simulación | **Valla con dientes esta etapa** (§2.2) | `test_cost_conformity.py`: ahora ejerce el MOTOR `PurgedWalkForwardBacktest`, no una cuenta a mano |
| Trazabilidad | Integrado + **cita muerta extirpada** (§2.5) | Valla de citas: toda ruta `data/research/*.json` citada en config o README existe en disco |
| Progreso, métricas e informes | Integrado | Progreso por configuración/época/corte en la cola; métricas del journal pintadas con su N (`test_como_va_metricas.py`) |
| Manejo de errores y producción | Integrado + **dos 500/errores corregidos** | §2.3 y §2.4; `inference` corregida (§2.6) |
| Protección de producción | Integrado | `live.enabled: false` + coherencia del loader (ISS-032 resuelta en sesión previa); `test_research_cannot_trade.py` |

## 2. Defectos corregidos en esta etapa

### 2.1 `--resume` reentrenaba la época ya hecha (ISS-036, nueva)

El checkpoint guarda el índice 0-based de la época **acabada** y el bucle
arrancaba sobre ese mismo índice. Arreglado: `start_epoch = completada + 1`,
y `best_epoch` inicial ya no es «la época que aún no ha corrido» sino el
mejor del checkpoint. Los tests que documentaban el defecto se invirtieron
al comportamiento correcto — estaban escritos para eso: «cuando se arregle,
que se vea que se ha arreglado».

### 2.2 El test de conformidad de costes no ejercía el motor (ISS-005)

`_neto_backtest_usd()` calculaba el bruto a mano y llamaba al módulo de
costes: la «conformidad entre motores» no tocaba el backtest. Ahora conduce
`PurgedWalkForwardBacktest._compute_pnl` con un fold de exactamente un
trade (lunes→miércoles, mid-a-mid 1.1000→1.1100), exige que el motor
produzca ese trade y solo ese, y compara el neto con la cartera simulada
con la tolerancia de 2¢ documentada.

### 2.3 El lector de CSV perdía un ulp por valor (ISS-037, nueva)

La valla del hash de `test_puerta_de_csv.py` cazó un falso «el fichero no es
el que fue descargado» sobre el EURUSD_H1.csv recién traído: el parser por
defecto de pandas releía **25.650 valores un ulp por debajo** de lo escrito
(p. ej. `1.2794699999999999` volvía como `1.27947`, otro double). La puerta
`io.read_frame` lee ahora con `float_precision="round_trip"`: por esa puerta
pasan el hash, el replay y toda medición. Valla nueva con cuatro de los
valores reales que fallaban.

### 2.4 `/api/learning` fallaba siempre con 500 (ISS-028)

El panel importaba `app.learning` — un módulo del **bot** que en el panel no
existe. Resuelto por la vía D-017: el GET lee el artefacto que escribe el
bot (`data/learning/report.json`) o responde 404 con el motivo; el POST se
retira con 501 y el motivo. Valla AST: el import cruzado no puede volver
(la primera versión de la valla se disparó a sí misma con el docstring que
explicaba el problema — por eso mira nodos y no texto).

### 2.5 La evidencia de la regla embarcada citaba un fichero inexistente (ISS-009, ISS-018)

`data/research/sweep-H4.json` fue citado en cinco sitios como «la evidencia»
de EMA_Trend_Pullback; nunca existió en este árbol. La justificación dice
ahora la verdad: es la regla de **referencia** del camino de reglas, sin
evidencia de ningún timeframe; el único barrido en disco no sostiene nada
(0/5, p=0.9935) y el registro no confirma ninguna regla. Valla de citas:
toda ruta citada en config/README debe existir. El resumen de «Huecos de
validez» del README quedó sincronizado (costes: cerrado; corrección del
barrido: cerrada para el barrido, abierta para el fuera de muestra).

### 2.6 La inferencia fijaba un lookback distinto del entrenamiento (ISS-038, nueva)

`inference.py` usaba 200; el entrenamiento usó 100. Ahora `load_model` lee
`training_provenance.lookback_bars` del checkpoint y corrige; sin
procedencia conserva el configurado y el log declara que es un supuesto.

### 2.7 Cuatro lectores de precios se saltaban la puerta (vía de datos de medidas)

La doctrina de la puerta (`research/io.py`: «una sola, y que no se pueda
volver atrás») existía para el laboratorio, pero el sondeo de esta etapa
encontró **cuatro** lectores de CSV de precios fuera de ella, tres de ellos
en caminos de MEDIDA:

- `training/train_universal.py::load_data` — los datos con los que se
  ajustan los pesos del modelo.
- `scripts/walkforward_eval.py::load_data` — la vía de datos del
  walk-forward.
- `research/holdout_eval.py::load_and_split_data` — la vía del holdout
  (además leía **sin** `comment="#"`: una cabecera de metadatos lo habría
  reventado).
- `adapters/market_data/csv_feed.py` — el feed de barras del runtime (su
  propio comentario ya confesaba: «research/io.py ya lo hacía; este
  adaptador se escribió sin mirar ahí»).

Los tres primeros leen ahora por `research.io.read_frame` — cabeceras,
minúsculas y el relector exacto en un solo sitio. El cuarto, siendo un
adaptador del bot, no importa el paquete de research (la dirección de
dependencia no se abre por ahorrar una línea): lleva la misma bandera
`float_precision="round_trip"` con el motivo escrito. Verificado:
`load_data` del entrenamiento y la puerta producen **columnas de valor
bit a bit idénticas** sobre el EURUSD H1 real.

### 2.8 Guardianes de evidencia que saltaban para siempre (ISS-006)

Cinco saltos apuntaban a un fichero retirado; seis más saltaban con motivo
falso («no bars CSV» con tres CSV en el directorio, cableados a un
XAUUSD_H1.csv de otra era). Todos corren ya contra artefactos reales y
**fallan** — no saltan — si falta lo que vigilan. El guardian del universo
exige además que `n_trials` declarado sea igual al recuento de filas.

## 3. Verificación ejecutada

- **Suite del bot (tests/):** **código de salida 0** — 1.648 coleccionados,
  1.594 pasan, 54 saltos con motivo declarado y comprobable (el bundle
  retirado de Kaggle, los artefactos de la generación anterior, y el
  riskfolio presente que hace inejercitable la negativa del optimizador).
- **Suite del panel (dashboard/tests/):** **código de salida 0** — 500
  pruebas, 500 pasan, cero saltos.
- **Puerta local nueva:** `python -m scripts.suite_completa` ejecuta ambas
  y reporta los códigos de salida. El repositorio **no tiene remote**
  (medido), así que no hay push que dispare CI: la puerta que puede existir
  es esta, y el día que haya remote la orden se mueve a un workflow tal
  cual (ISS-012, parcial por falta de remote).
- **Lección de entorno registrada en el propio runner:** el directorio
  basetemp por defecto de pytest en esta máquina
  (`%TEMP%\pytest-of-fiver`) quedó bloqueado con Access denied y envenenó
  una corrida entera con 359+ errores de setup que se leían como rojo de
  código. El runner pasa `--basetemp` a un directorio fresco por corrida.
- Verificaciones dirigidas de esta etapa: costes 2/2, CSV puerta 3/3,
  cronología 8/8, reanudación 9/9, reality check 41/41, universo de barrido
  19/19, señal embarcada 7/7, learning del panel 4/4, lookback 3/3,
  optimizador honesto 2+1, cableado del entrenamiento + vallas de fuente del
  walk-forward + ciclo de vida (CsvMarketData) 81/81.

## 4. Defectos y bloqueos que quedan, y por qué

### 4.1 Científicos — deliberadamente diferidos (el encargo lo prohíbe hoy)

1. **ISS-002 (CRÍTICA):** no existe tramo fuera de muestra real para el
   modelo; el epoch se elige con la validación que luego se llama OOS, y el
   walk-forward no reentrena por fold (ISS-010). El runner lo dice en su
   resultado. Es diseño de experimento, no software: requiere el plan de
   cualificación (R-003).
2. **Ninguna metodología aprobada** y el laboratorio se niega — correcto.
3. **El registro no confirma ninguna regla** (valla nueva). La regla
   embarcada es referencia, no evidencia.
4. **Journal vacío** (ISS-011): la cadena de hash no se ha verificado contra
   una sesión real del bot.
5. **Sin backup declarado de data/** (ISS-016) y sin prueba de
   kill-y-relanzar (ISS-017): decisiones operativas del dueño.
6. **Cualificación científica** (Fases 2–6 del plan): intacta. El conjunto
   elegible de la Fase 1 (EURUSD H1 2010-09-01 → 2018-09-23) espera a que
   la Fase 2 congele `universal_v2`. **No se consumió ni declaró nada.**

### 4.2 Técnicos abiertos, con disposición

- **ISS-007 (optimizador HRP):** verificado como **instrumento
  desconectado** — nada del camino de decisión lo importa (valla AST),
  `codependence` sí se lee (ISS-008 obsoleta), riskfolio-lib 7.4.0 está
  instalado pero **sin declarar** en requirements. Cablearlo es un cambio
  de comportamiento del riesgo que exige encargo del dueño, librería
  declarada y tests — en el mismo commit.
- **ISS-015:** el panel corre bajo myenv con las versiones del repo
  funcionando (suites verdes); la divergencia con dashboard/requirements
  sigue siendo una deuda de pinned-versions, no un defecto activo.
- **ISS-020 (audit/ borrada):** decisión del dueño pendiente.
- **`_act` duplicado en runtime/loop.py (ISS-029):** resuelto en sesión
  previa con la extracción de casos de uso — verificado y cerrado hoy.

## 5. El árbol de trabajo: disposición completa

Criterio de cierre: «sin cambios ocultos». Estado tras la etapa:

| Fichero | Disposición |
|---|---|
| `config/jobs.yaml`, `data/jobs/README.md`, `docs/experimentos/EURUSD-H1-etapa-nueva.md`, `project-control/INFORME_TRABAJOS.md`, `.gitignore` | **Commiteados** (commit de la sesión previa pendiente): la cola es sistema, su config viaja; el criterio del dueño viaja porque el registro lo referencia |
| `runtime/_ver_job.py` | **Poda documentada**: sonda de una sola corrida con un job ID clavado; duplica `jobs_reader.leer` y la página de trabajos |
| `panel.log`, `suite-*.txt`, `.acl-reports/` | **Gitignored**: evidencia de corridas de esta máquina, como `data/` — los números viajan en los informes |
| `data/jobs/` (excepto README), `data/*` | Ya ignorados por diseño |

## 6. Cómo reproducir esta verificación

```
myenv\Scripts\python.exe -m scripts.suite_completa
```

Requisitos: `myenv/` (requirements.txt) y `dashboard/.venv/` (lo necesitan 3
pruebas del panel — receta en INFORME_INTEGRACION §10). El script reporta el
**código de salida** de cada suite: es el veredicto, no un recuento de
líneas.

---

**Lo que esta etapa NO demuestra** (y no pretende): que exista metodología
aprobada, que alguna estrategia sea rentable, que el modelo tenga habilidad
fuera de muestra. Terminar la arquitectura es la condición para poder
preguntarlo con herramientas que no mientan — no la respuesta.
