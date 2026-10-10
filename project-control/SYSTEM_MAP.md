# GTS — Mapa del Sistema

**Revisión:** `e801d4e` + 36 ficheros de trabajo sin commitear · **Fecha:** 2026-10-10
**Alcance:** auditoría de solo lectura. No se modificó código de producción.
**Método:** inspección del árbol, lectura de código/configuración, y ejecución de
las dos suites de tests (1384 + 411). No se conectó a MetaTrader 5, no se envió
ninguna orden, no se entrenó ni se barrió nada.

---

## 1. Lo que este sistema es, y lo que no es

GTS es una **plataforma de investigación de trading con un simulador y un panel
de observabilidad**. El bucle de trading es real y opera contra un libro en
memoria; el panel es real y lee una cuenta MT5 de verdad.

**El sistema no puede colocar órdenes reales.** Esto está verificado por
construcción y por test: `dashboard/app/main.py` expone 84 rutas y ninguna es de
orden; el envío de órdenes vive en `live/terminal.py` y solo se alcanza con
`live.enabled: true`, que hoy es `false` en `config/gts.yaml:363`.

> **Consecuencia que gobierna todo este mapa:** ninguna cifra de rendimiento de
> este repositorio es una cifra de dinero. Un Sharpe medido aquí es un Sharpe
> sobre una serie de barras, con un libro simulado y un perfil de costes
> declarado. No es un resultado de cuenta.

## 2. Arquitectura real detectada

Dos programas, dos entornos virtuales, un directorio de ficheros donde se
encuentran (`data/`). Esa separación es el hecho central del diseño (D-017).

```
   ┌───────────────────────── BOT (myenv, python -m app.main) ─────────────────────────┐
   │                                                                                  │
   │  mt5/feed.py ──► runtime/loop.py ──► strategies/ ──► app/decision_core.py        │
   │  (barras H1)      (1305 LOC,        (11 reglas,        (señal→riesgo→sizing)     │
   │                   journaliza         −1/0/+1)              │                    │
   │                   CADA barra)                              ▼                    │
   │                                                   risk/engine.py                │
   │                                                   (11 validadores)               │
   │                                                           ▼                    │
   │                                            simulation/portfolio.py              │
   │                                            (libro en memoria + costes)         │
   │                                                           ▼                    │
   │                                          storage/journal.py (cadena de hash)     │
   │                                                           ▼                    │
   │                                       runtime/control.py ──► data/control/      │
   └───────────────────────────────────────────────────┬──────────────────────────────┘
                                                       │ ficheros
   ┌───────────────────────────────────────────────────▼──────────────────────────────┐
   │  data/control/{state,command,features,selections}.json + data/journal/          │
   └───────────────┬──────────────────────────────────────────┬──────────────────────┘
                   │ lee                                    │ lee
   ┌───────────────▼──────────────┐          ┌───────────────▼──────────────────────┐
   │  PANEL (dashboard/.venv)     │          │  INVESTIGACIÓN (CLI / laboratorio)   │
   │  FastAPI + Jinja2 + HTMX     │          │  research/ (14 216 LOC, 43 módulos)  │
   │  84 rutas, SOLO LECTURA      │          │  barrido → Reality Check → registro  │
   │  + pause/resume/flatten      │          │  registro → evidencia_confirmada()   │
   └──────────────────────────────┘          └──────────────────────────────────────┘
```

### 2.1 Componentes por dominio y tamaño real

| Dominio | Rutas | LOC | Naturaleza |
|---|---|---|---|
| Bot / composition root | `app/` (9 módulos) | 4 433 | Cableado, CLI `run/doctor/once` |
| Núcleo de decisión | `app/decision_core.py` | 926 | Señal → riesgo → sizing → libro |
| Bucle | `runtime/` (6 módulos) | 2 319 | Orden de operaciones, journal, heartbeat |
| Núcleo compartido | `core/` (+ `features/`, `heads/`, `universal/`) | 4 418 | Tipos, costes, broker, features, modelo |
| Investigación | `research/` (43 módulos) | 14 216 | La mayor capa del sistema |
| Riesgo | `risk/` (4 módulos) | 1 490 | 11 validadores + sizer + optimizador |
| Estrategias | `strategies/` (4 módulos) | 759 | 11 reglas, podadas de 18 el 2026-10-08 |
| Simulación | `simulation/` (3 módulos) | 1 395 | Libro + equity + costes |
| Ejecución real | `live/` (3 módulos) | 923 | **Apagada.** Único camino a `order_send` |
| MT5 lectura | `mt5/` (3 módulos) | 842 | Feed de barras, ciclo de vida |
| Journal | `storage/journal.py` | 377 | Cadena de hash |
| Indicadores | `indicators/` (3 módulos) | 925 | ATR, Ichimoku, streaming |
| Entrenamiento | `training/`, `core/universal/` | 707 + modelo | Transformer multi-activo |
| Panel | `dashboard/app/` | 7 640 | FastAPI, 22 ficheros de test |
| **Tests** | `tests/` (79 ficheros) | **23 036** | 1384 tests del bot |

**El laboratorio de investigación (14 216 LOC) es tres veces el bucle que
opera (2 319 LOC).** Ese desequilibrio es el hecho estructural más importante
del mapa: el sistema sabe medir mucho más de lo que sabe operar, y esa asimetría
es coherente con el estado de la evidencia (sección 4).

### 2.2 Capas de datos y flujo de decisión

| # | Capa | Artefacto | Consumido por |
|---|---|---|---|
| 1 | Bytes de mercado | `data/raw/*.csv` (8 CSVs, 7 símbolos/timeframes) | `research/`, `scripts/download_data.py` |
| 2 | Features | `core/features/` (extractor universal, normalizadores por clase de activo) | entrenamiento, panel |
| 3 | Etiquetas | `research/labeling.py` (triple barrera relativa a volatilidad, numba) | `research/dataset.py` |
| 4 | Barrido | `research/sweep.py` + `sweep_parallel.py` | `data/research/barrido-*.json` |
| 5 | Corrección | `research/reality_check.py` (White + Newey-West, 10 000 boots) | Campo `reality_check` del barrido |
| 6 | Confirmación | `research/confirmacion.py` sobre tramo `limpia` | `data/research/confirmacion-*.json` |
| 7 | Registro | `research/registro.py` → `data/research/registro-experimentos.json` | `evidencia_confirmada()` |
| 8 | Puente | `app/main.py:193` — **el bot opera lo confirmado, o se niega a arrancar** | `runtime/loop.py` |

**El paso 8 es la pieza de mayor valor del sistema.** No es una convención: es
un `RuntimeError` con la condición escrita en el docstring
(`app/main.py:171`: *"If no CLI selections AND no confirmed evidence exists"*).
Tener un checkpoint no es tener una ventaja, y el código lo aplica.

### 2.3 Divergencias entre lo documentado y lo medido

| # | Lo que dice la documentación | Lo que mide el código |
|---|---|---|
| D1 | `app/main.py:36-44`: *"the dependency graph is a tree"* | 8 aristas del bot hacia `research/` (el instrumento) |
| D2 | `runtime/loop.py:49` importa `app.decision_core` | **Ciclo duro `app` ↔ `runtime`**, contradice *"app wires them"* |
| D3 | README D-017: *"the panel does not import the bot"* | **Verificado en el sentido fuerte: 0 aristas de `dashboard/app/*` hacia `app.*`.** Las entradas reales al bot son `core.*`, `strategies`, `research.*` |
| D4 | `tests/test_no_order_path.py`: *"la simulación no alcanza el broker"* | `simulation/broker_book.py:66` importa `live.execution` a nivel de módulo |
| D5 | `config/gts.yaml:186` cita `data/research/sweep-H4.json` como evidencia de `EMA_Trend_Pullback` | **El fichero no existe.** Citado también en README:433, 442, 609, 642 |

D3 se ha **corregido** respecto a la auditoría anterior del 2026-10-09: la
premisa de que el panel alcanzaba `app.*` era un falso positivo de resolución de
imports (el panel tiene su propio `dashboard/app/` en `sys.path[0]`).

## 3. Estado verificado por dominio

Leyenda de estados, aplicada en `COMPONENTS.yaml`:
`implementado` (el código existe) · `validado` (un test lo prueba) ·
`operativo` (medido en ejecución) · `No evaluado` (sin evidencia suficiente).

| # | Dominio | Implementado | Validado | Operativo |
|---|---|---|---|---|
| 1 | Requisitos y objetivos | Sí | Sí | Sí — registro de experimentos con criterios preinscritos |
| 2 | Investigación de estrategias | Sí | Sí | Sí — 4 barridos, 1 confirmación ejecutados |
| 3 | Datos y calidad | Sí | Sí | Sí — `provenance.yaml` declara origen y límites |
| 4 | Features | Sí | Sí | Sí — leakage del scaler **arreglado y verificado** (16 tests) |
| 5 | Estrategias / ML | Sí | Sí | **No** — el modelo no tiene habilidad en dirección (medido) |
| 6 | Construcción de portafolio | Parcial | Parcial | **Roto en silencio** (ver ISS-06) |
| 7 | Backtesting | Sí | Sí | Sí — paridad paralelo/serie exigida por test |
| 8 | Validación estadística | Sí | Sí | Sí — Reality Check **ejecutado y publicado** |
| 9 | Gestión de riesgos | Sí | Sí | Sí — 11 validadores, todos cableados en `risk/engine.py` |
| 10 | Ejecución y conectividad MT5 | Sí | Sí | **No evaluado** — `live.enabled: false` |
| 11 | Contabilidad de operaciones | Sí | Sí | Sí — libro con equity y costes |
| 12 | Controlador del bot | Sí | Sí | Sí — pause/resume/flatten + lock de instancia |
| 13 | Observabilidad | Parcial | Parcial | Sí — panel lee 5 logs; **métricas de riesgo no se pintan** |
| 14 | Infraestructura | Parcial | Sí | Sí — dos venvs, **sin CI** |
| 15 | Calidad y versiones | Sí | Parcial | **1381/1384 verdes; 3 fallos por contaminación de bundle** |
| 16 | Seguridad y gobernanza | Sí | Sí | Sí — ver sección 5 |
| 17 | Despliegue y recuperación | Mínimo | Mínimo | **No evaluado** — sin despliegue documentado |
| 18 | Viabilidad económica | **No** | **No** | **No** — sin evidencia positiva |

## 4. La evidencia, medida

### 4.1 El Reality Check se ha ejecutado, y falla

Los tres barridos que hay en disco **traen ya el p-valor corregido**:

| Fichero | Símbolo/TF | Ganador | t observado | **p-valor** | Veredicto |
|---|---|---|---|---|---|
| `barrido-AUDUSD-H1-entrenar-1.json` | AUDUSD H1 | ATR_Breakout | 2,066 | **0,6988** | No sobrevive |
| `barrido-AUDUSD-H1-entrenar-2.json` | AUDUSD H1 | EMA_Trend_Pullback | 1,258 | **0,9980** | No sobrevive |
| `barrido-AUDUSD-H4-entrenar-1.json` | AUDUSD H4 | MA_Crossover | 2,605 | **0,2715** | No sobrevive |

Los tres p-valores están **muy por encima de 0,05**. Con la corrección aplicada,
**ninguna configuración de ninguna rejilla es distinguible del ruido**. Esto es un
resultado legítimo y publicable — el criterio preinscrito del experimento
`reality-check` dice exactamente eso.

### 4.2 El registro de experimentos: nada confirmado

`data/research/registro-experimentos.json` es lo que gobierna qué puede operar
el bot. Su contenido, medido:

| Par | Regla | Resultado |
|---|---|---|
| EURUSD M15 | ATR_Breakout | 0/25 sobreviven → **refutada** |
| AUDUSD H1 | ATR_Breakout | 0/25 sobreviven |
| AUDUSD H1 | EMA_Trend_Pullback | 0/25 sobreviven |
| AUDUSD H4 | MA_Crossover | 4/25 sobreviven → **confirmación `refutada`** (t = −0,488; beneficio neto **negativo** con 565 operaciones) |
| EURUSD H1 | — | Tramos declarados, **sin resultados** |

**Cero estrategias confirmadas.** Por tanto `evidencia_confirmada()` devuelve
vacío y el bot **se niega a arrancar** salvo `--selection` explícito por CLI.

### 4.3 El walk-forward del modelo

`data/research/archive/walkforward-H1-modelo-{medido,conservador}-20261010.json`,
generado por el commit `44f8591` con `checkpoints_training/best_model.pt`:

| Métrica | Medido | Conservador |
|---|---|---|
| Folds | 148 (112 dentro / 28 fuera) | idem |
| **Sharpe fuera de muestra** | **−0,064 ± 1,363** | **−0,067 ± 1,362** |
| Sharpe dentro de muestra | −0,421 ± 1,770 | −0,427 ± 1,771 |
| Operaciones totales | 2 756 | 2 756 |
| Retorno anual medio (OOS) | +1,83 % | +1,79 % |

**El Sharpe fuera de muestra es ~0 y su desviación (1,36) supera el propio
valor.** Con 28 folds, esta cifra no distingue "nada" de "algo". El propio
fichero lo declara en `selection_note`: *"el epoch del checkpoint se eligió con
validación que empieza en la fila 45010, dentro de la muestra reportada; por eso
los folds llamados out-of-sample no lo son para ese checkpoint"*.

Esto **confirma y cierra** el experimento `no-signal-in-direction` que el
README declaraba desde 2026-10-03: la cabeza de dirección no tiene habilidad.

### 4.4 Lo que este sistema NO ha demostrado

- Que ninguna regla supere el ruido tras corregir por pruebas múltiples
  (**medido: las tres rejillas fallan**).
- Que el modelo universal tenga habilidad en dirección (**medido: Sharpe OOS
  ≈ 0**).
- Que exista un tramo fuera de muestra real (**medido: no existe**, ISS-02).
- Cualquier cifra de dinero (**no aplica: el sistema no llega al broker**).

## 5. Seguridad — lo verificado

| Propiedad | Estado | Evidencia |
|---|---|---|
| El panel no coloca órdenes | **Verificado** | 84 rutas, 0 de orden; test con nombre explícito |
| El panel no importa el bot | **Verificado** | 0 aristas `dashboard/app/*` → `app.*` |
| El panel escribe el heartbeat del bot | **Verificado** | `PanelControl` no tiene `publish` |
| Heartbeat rancio se lee como "no corriendo" | **Verificado** | test dedicado |
| Enlace a la red | **Verificado** | `GTS_HOST` por defecto `127.0.0.1` |
| Credenciales MT5 fuera del repo | **Verificado** | `.env` ignorado; `.env.example` con valores vacíos |
| `live.enabled` | **`false`** | `config/gts.yaml:363`; `allow_funded: false` = segundo cerrojo |
| **El panel puede desarmar los 11 validadores** | **ABIERTO** | `POST /api/features` (main.py:2118) escribe `features.json` sin autenticación ni CSRF, y `app/features.py` lo traduce a los límites que lee `risk/engine.py` |

El último punto es el único hallazgo de seguridad abierto, y es serio: el panel
no puede **colocar** órdenes (correcto), pero sí puede **apagar los frenos** que
las autorizan.

## 6. Fuentes de verdad y su estado

| Artefacto | Contenido | Estado |
|---|---|---|
| `memory-graph.jsonl` | 117 entidades + 147 relaciones | **Vivo.** Fuente de verdad del conocimiento |
| `README.md` | 914 líneas: decisiones, bugs, experimentos, huecos | **Vivo**, parcialmente desalineado (ver ISS-04) |
| `data/research/registro-experimentos.json` | 4 pares, 5 tramos, 6 resultados | **Vivo.** Gobierna qué opera el bot |
| `data/raw/provenance.yaml` | Origen, rango y barras de 8 CSVs | **Vivo.** Declara que el spread del feed demo no sirve para costear |
| `config/gts.yaml` | Contrato del bot | **Vivo**, con una cita rota (D5) |
| `project-control/` | Este mapa | **Nuevo en esta revisión** |
| ~~`audit/`~~ | Auditoría Fase 1 del 2026-10-09 (7 informes) | **Borrada del árbol**, recuperable en `HEAD` |
| ~~`inventory/`~~ | Inventario de 3 ficheros | Retirado 2026-10-03 (D-018) |

**Nota sobre `audit/`:** 220 ficheros y 65 540 líneas fueron borrados y están en
el índice de git como eliminación preparada. Sus informes siguen recuperables
con `git show HEAD:audit/<dominio>/<fichero>.md`. **No se ha restaurado nada**
— es una decisión del propietario, no de la auditoría. La evidencia de esa
auditoría que sigue siendo válida está citada arriba y marcada como tal cuando
se ha re-verificado hoy.

## 7. Qué se midió y qué no

**Medido hoy (2026-10-10, `e801d4e` + trabajo sin commitear):**
- 1384 tests del bot recogidos; 3 fallan, 10 se saltan.
- 411/411 tests del panel en verde.
- Reproducción de los 3 fallos y su causa raíz.
- p-valores de los 3 barridos leídos de disco.
- Contenido del registro de experimentos leído.
- 84 rutas del panel contadas; 0 de orden.
- 11 validadores de riesgo contados y todos localizados en `risk/engine.py`.
- 13 ficheros divergentes entre `gts/` (bundle) y la raíz.

**No medido:**
- Rendimiento del árbol completo, ni en navegador real.
- Comportamiento con MT5 conectado (prohibido en esta fase).
- Cualquier cifra de dinero: no aplica, el sistema no llega al broker.
- El `basetemp` dentro del repo (falso positivo conocido de
  `test_research_cannot_trade.py`) — evitado en esta corrida.
- Si `dashboard/run.py` re-ejecuta bajo el intérprete correcto (auditado en
  2026-10-09, **no re-verificado hoy**).