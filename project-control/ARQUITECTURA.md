# GTS — Arquitectura y ciclo de vida (estado tras la migración)

**Fecha:** 2026-10-10 · **Base:** `e801d4e`
**Este documento describe lo que el sistema HACE, no lo que planea.** Lo que
sigue sin estar implementado está en la última sección, con su nombre.

---

## 1. Qué se ejecuta y cómo se comprueba

```powershell
# El recorrido completo: barras → estrategia → riesgo → tamaño → libro → journal
myenv\Scripts\python.exe -m scripts.recorrido_runtime

# La suite del bot
myenv\Scripts\python.exe -m pytest tests

# La suite del panel
dashboard\.venv\Scripts\python.exe -m pytest dashboard\tests
```

`recorrido_runtime` es la comprobación de que el sistema funciona, y es la
importante: levanta el runtime de PRODUCCIÓN (`runtime.loop.TradingLoop`, el
mismo que arranca `python -m app.main run`) y lo conduce con `run_once`. Lo
unico que sustituye es el terminal, por uno que no sabe enviar ordenes.
Recorre barras de verdad y dice cuántas procesó, qué decidió el riesgo, si
la ejecución fue por el adaptador simulado, si la reconciliación da segura y si
el journal sigue encadenado.

Antes de esto existía `recorrido_demo`, que montaba el pipeline en aislamiento
con un feed de CSV. Se borró el 2026-10-10: leía de `data/raw/*.csv`, que la
limpieza del entorno quitó, así que ya no recorría nada - y aún así decía
`RESULTADO: OK` con cero journal. Un chequeo en verde que no comprueba nada es
peor que no tener chequeo, porque se lee como cobertura.

### La suite y sus artefactos

Dos ficheros de la suite **exigen artefactos que el repositorio no guarda**, y se
saltan solos cuando no están, con el motivo escrito dentro:

| Test | Artefacto | Por qué no está |
|---|---|---|
| `test_colab_bundle.py` (11 tests) | `colab/gts_colab_bundle.zip` | Está en `.gitignore`. Lo construye `python colab/build_bundle.py`. |
| `test_scaler_canonical.py` (4 tests) | `checkpoints_training/scalers/canonical_scaler.joblib` | Lo produce el entrenamiento, no el repo. |

Estos no son regresiones ni fallos: son precondiciones que faltan. Antes de que
los `skip` existieran, un clon nuevo arrancaba con 15 rojos que no significaban
nada y la Sesión perdía tiempo intentando arreglar un código sano. El `skip` es
a nivel de módulo porque los tests del fichero comparten el mismo artefacto.

Dos reglas de la valla de infraestructura (`tests/conftest.py`), que imprime un
cartel al final de cada corrida cuando todo el rojo es ruido de Windows:

- **Usa un `--basetemp` nuevo en cada corrida.** El de por defecto
  (`pytest-of-fiver`) se queda cogido entre corridas y reproduce el fallo.
- **No borres el temporal ni mates procesos Python mientras corre una suite.**
  Hacerlo genera cientos de errores fantasma en el setup; la valla los detecta,
  pero la corrida entera se pierde.

---

## 2. Las capas

```
adapters/          Hablan con el mundo. No saben qué hay arriba.
├── broker/in_memory.py    libro de órdenes con ciclo de vida completo
├── market_data/csv_feed.py    barras desde un CSV
├── clock.py           reloj de pared y reloj fijo (replay)
└── portfolio/returns.py    datos para el optimizador

core/              El vocabulario y los contratos. CERO dependencias hacia fuera.
├── types.py         Action, FailReason, HealthState
├── models.py        TickSnapshot, PositionSnapshot, AccountSnapshot
├── costs.py         modelo de costes unificado
├── exit_reasons.py  vocabulario de cierres (movido aquí, ver §5)
├── ports/           LOS CONTRATOS: qué necesita el sistema del mundo
│   ├── market_data.py     MarketDataPort, ClockPort, Bar
│   ├── orders.py          OrderPort, OrderIntent, OrderAck, Fill, OrderState
│   ├── position_book.py   PositionBookPort
│   ├── journal.py         JournalPort
│   └── profiles.py        ExecutionProfile, RiskPolicy, RuntimeConfig
└── usecases/        LA ORQUESTACIÓN. Un caso de uso por fichero.
    ├── process_bar.py     el ciclo completo de una barra
    ├── authorize_trade.py los validadores
    ├── manage_position.py abrir/cerrar/voltear/vigilar stops
    └── reconcile.py       ¿el libro coincide con el broker?

runtime/lifecycle.py   El bucle: esperar la siguiente barra y repetir.
simulation/             El libro en memoria (heredado, ya no importa a live/)
live/                   El ejecutor MT5. Apagado.
research/               El laboratorio.
dashboard/              El panel. Solo lectura.
```

**La regla que lo sostiene:** las dependencias solo apuntan hacia dentro. Un
puerto no importa un adaptador; un adaptador no importa un caso de uso; un caso
de uso no importa un adaptador. Es lo que permite que el mismo caso de uso corra
contra un libro en memoria o contra MT5 sin cambiar una línea.

---

## 3. El ciclo de vida de una operación

`core/usecases/process_bar.py` recorre estos pasos, en este orden:

| # | Paso | Si falla |
|---|---|---|
| 1 | **Barra cerrada.** Se rechaza una barra ya procesada. | `already_processed`, no infla el recuento |
| 2 | **Stops y objetivos**, antes de decidir | `close` y se registra; el estado de la barra es la salida |
| 3 | **La estrategia.opina.** Series alineadas al histórico | `internal_error`, se registra, el bucle sigue |
| 4 | **Traducir a acción** (`decisions.decide`, función pura) | `hold` si coincide con lo abierto |
| 5 | **Dimensionar** al precio real de entrada (ask/bid), no al cierre | `unsizable` |
| 6 | **Autorizar.** Los once validadores, un solo sitio | `risk_rejected` con los motivos y los números |
| 7 | **Ejecutar.** Enviar si el perfil permite; si no, aplicar en local | `rejected` del broker, registrado |
| 8 | **Registrar** en el journal encadenado | la entrada es la evidencia |

El orden tiene dos decisiones deliberadas:

- **Stops antes de decidir.** Una posición que el mercado ya cerró por stop no
  se puede mantener porque la regla diga "mantener".
- **Dimensionar antes de autorizar.** Es cálculo barato sin efectos; al revés
  se haría trabajo para una operación que se va a rechazar.

### Estados de una orden

```
PENDING ──▶ FILLED
   ├────▶ PARTIAL ──▶ FILLED
   │          └─────▶ CANCELLED
   ├────▶ REJECTED
   └────▶ CANCELLED
```

Los cinco existen en `adapters/broker/in_memory.py` y en el ejecutor real, con la
misma máquina de estados. `PARTIAL` importa: el fill parcial es real, y un
libro que no lo modela no puede reproducir el estado en el que una orden está a
medio construir.

**Idempotencia por `client_id`:** reenviar la misma orden devuelve su estado en
lugar de crear otra. Un reintento tras un timeout no puede duplicar una
posición.

---

## 4. Perfiles de ejecución

| Perfil | Envía | Política | Para qué |
|---|---|---|---|
| `research` | no | cualquiera | barras de un CSV, sin cuenta |
| `backtest` | no | cualquiera | walk-forward, paridad |
| `paper` | **no** | enforce / observe | precios reales, libro simulado |
| `shadow` | **no** | enforce / observe | cuenta demo, **nunca envía** |
| `demo` | **sí** | **solo enforce** | envío real a dinero que no es tuyo |
| `live` | **sí** | **solo enforce** | envío real a dinero tuyo |

La garantía no es un `if` en el arranque: es un tipo. `require_send_permission()`
devuelve un `SendCapableProfile`, que **no se puede construir** para un perfil
sin envío, ni con una política distinta de `ENFORCE`. Un perfil nuevo obliga a
decidir si entra en ese conjunto, y la decisión queda en el código.

```python
from core.ports.profiles import require_send_permission, RuntimeConfig
from core.ports import ExecutionProfile, RiskPolicy

cfg = RuntimeConfig(ExecutionProfile.SHADOW, RiskPolicy.ENFORCE)
cfg.puede_enviar          # False
require_send_permission(cfg)   # ConfigError: shadow no tiene permiso para enviar
```

---

## 5. Dependencias que se corrigieron

| Antes | Ahora | Por qué importaba |
|---|---|---|
| `risk.portfolio_optimizer → research.io_arctic` | `adapters.portfolio.returns` | El motor de riesgo no debe saber que hay un laboratorio. El import estaba **dentro de dos funciones**, así que un fallo de datos salía a mitad de una optimización. |
| `simulation.broker_book → live.execution` | `core.exit_reasons` | El simulador no debe conocer al ejecutor real. Se importaba una función que traduce un código a una palabra; no necesita el broker. |
| `/gts/` (copia del bundle) contaminaba `sys.path` | ya no existe | Los procesos hijos del barrido paralelo importaban una `core/costs.py` vieja y fallaban con un `TypeError` sin relación con el barrido. Tres tests rojos solo en la suite completa. |

**Pendientes, sin tocar** (no bloquean el ciclo de vida):

| Dependencia | Por qué sigue |
|---|---|
| `runtime.loop → app.decision_core` | El bucle heredado aún no usa el `ProcessBar` nuevo. Es el siguiente paso natural. |
| `research.replay_runner → runtime.loop` | El replay reutiliza el bucle a propósito: es lo que hace comparables el replay y la operación. Desacoplarla requiere un `MarketDataPort` para el replay. |
| `research.baseline → app.models` | Import trivial de tipos. |

---

## 6. Qué se puede ejecutar ahora

```powershell
# 1. El recorrido completo
myenv\Scripts\python.exe -m scripts.recorrido_runtime
```

El recorrido genera sus propias barras deterministas (`numpy.default_rng`, semilla
fija), asi que no necesita nada en disco para arrancar. Y usa la configuracion
REAL de `config/gts.yaml`, no numeros inventados en el script: una comprobacion
que lleva sus propios limites de riesgo diverge de produccion en cuanto alguien
cambia uno, y diverge hacia el lado peligroso.

```
[1] el entorno se limpio de artefactos historicos
    residuos presentes: ninguno
[3] arranque
    salud              : ready
    razon              : '500 velas cargadas (1 simbolos); esperando la proxima'
[4] estado de pausa tras arrancar en directorio limpio
    pausado            : True  (sin estado previo, no se opera)
[5] barras conducidas por run_once()
    pasadas con barra : 120 / 120
    decisiones        : 120
    riesgo aprobado   : 5
    riesgo rechazado  : 0
    ordenes           : 5
[6] que ejecuta el runtime
    casos de uso      : ['ProcessBar']  <- ['core.usecases.process_bar']
    adaptador         : PortfolioBook sobre SimulatedPortfolio
    reconciliacion    : Reconcile en 1/1 caso(s) de uso
[8] reconciliacion
    veredicto         : both_flat
    seguro continuar  : True
[9] envio externo
    puerto de ordenes : NEGADO
[10] journal
    entradas          : 124
    cadena valida     : True
RESULTADO: OK
```

Lo que eso demuestra es que el RECORRIDO esta cableado y es correcto: el runtime
real construye los casos de uso nuevos, decide barra a barra, el riesgo emite
veredictos, la ejecucion va por el adaptador de simulacion y el journal queda
encadenado. Lo que NO demuestra, y conviene no dejar de decir: que la estrategia
gane dinero. Las barras son ruido con semilla fija, y un sistema que gana con
ruido con semilla fija no ha demostrado nada que valga fuera de aqui. Para
datos de verdad: `scripts/download_mt5_data.py` con el terminal abierto.

---

## 7. Lo que NO está implementado

Estado a 2026-10-10, tras conectar el ciclo de vida al runtime real.

| Falta | Estado |
|---|---|
| ~~Adaptador MT5 de `MarketDataPort`~~ | **Hecho 2026-10-10.** `adapters/market_data/mt5_feed.py`. El runtime ya lee por el puerto. |
| **Perfiles en la configuración** | `config/gts.yaml` sigue usando `live.enabled: false` como interruptor, y `app/main.py` lo traduce a un `ExecutionProfile` con `_perfil_de`. El perfil es ya lo que decide, pero sigue having un solo interruptor en el YAML en vez de una declaración de permisos. |
| **Observabilidad de las métricas** | `drawdown`, `sharpe`, `expectancy` se calculan y no se pintan. |
| **Alertas** | No hay ninguna. |
| **Adapters de broker reales** | `NoSendBroker` y `PortfolioBook` niegan el envio por diseno. No hay un adaptador MT5 del `OrderPort`: el camino de envio real no existe todavía, y por eso no hay nada que puedan activar DEMO ni LIVE. |
| **Entrenamiento y barridos** | Sin datos de mercado; los CSV sintéticos se retiraron del árbol. |

### Lo que se cerró en esta tanda

| Estaba abierto | Ahora |
|---|---|
| `ProcessBar` existía pero `runtime/loop.py` y `app/decision_core.py` tenían su propio camino; los dos sin fusionarse | El bucle construye un `ProcessBar` por símbolo y delega en él. `app/decision_core.py` se borró: era la segunda implementación del ciclo. `runtime/lifecycle.py`, que era un bucle paralelo sin conectar, también. |
| La reconciliación solo corría al arrancar | Corre en cada barra, antes de aumentar exposición. Las salidas y el plano de emergencia nunca se bloquean. |
| El panel (`POST /api/features`, sin autenticación ni CSRF) podía cambiar los once interruptores de riesgo y los números de límite | Su bloque `risk` y su bloque `limites` se descartan en `app/config.load`, con aviso en el log. Los límites se leen solo de `config/gts.yaml`, que está versionado. Lo demás del panel (`learning`) sigue llegando. |
| Un reinicio devolvía el bot a `ready` desde pausa | La pausa se lee del heartbeat anterior. Sin estado previo, el bot arranca **pausado**. |
| El bot se negaba a arrancar sin resultados históricos de experimentos |
| El feed leía de la terminal sin validar nada | `MT5MarketData` descarta la barra en formación, ordena, deduplica, rechaza precios incoherentes y convierte un corte de conexión en "no hay barra". Un feed que retrocede en el tiempo ya no se toma por barra nueva: era una segunda decisión sobre un cierre ya operado. |
| El bucle no ejecutaba la reconciliación por barra | Corre antes de aumentar exposición; con estado en disputa la barra queda negada y el plano de emergencia sigue pudiendo cerrar. | Sin evidencia arranca si y solo si el proceso no puede enviar órdenes. Con permiso de envío, la evidencia sigue siendo obligatoria. |
| `runtime/` importaba `app.decision_core` | Cerrado. `verificar_dependencias` lo comprueba y falla si vuelve. |

---|---|
| **`ProcessBar` conectado al bucle real** | El caso de uso funciona y se ejecuta en el recorrido y en los tests, pero `runtime/loop.py` y `app/decision_core.py` siguen usando su propio camino. Los dos existen y no se han fusionado. |
| **Perfiles en la configuración** | `ExecutionProfile` y `RiskPolicy` existen y se validan, pero `config/gts.yaml` sigue usando `live.enabled: false` como interruptor. Migrarlo es un cambio de configuración y del cableado de `app/main.py`. |
| **Reconciliación en el bucle real** | `runtime/lifecycle.py` reconcilia en cada barra y está probado. El bucle heredado `runtime/loop.py` sigue reconciliando una vez al arrancar. |
| **Adaptador MT5 de `OrderPort`** | Este es el que falta, y es el que importa: sin él no hay manera de mandar una orden, así que SHADOW no puede filtrarse por accidente. `NoSendBroker` y `PortfolioBook.submit()` cierran el paso mientras tanto. |
| **Observabilidad de las métricas** | `drawdown`, `sharpe`, `expectancy` se calculan y no se pintan. |
| **Alertas** | No hay ninguna. |
| **Panel sin autoridad sobre el riesgo** | `POST /api/features` sigue escribiendo `features.json`, que se traduce a límites del motor de riesgo. **Es el hallazgo de seguridad abierto más importante.** |
| **Entrenamiento y barridos** | functioning sobre datos sintéticos; no verificados con datos de mercado. |

---

## 8. Un defecto que se encontró y se corrigió por el camino

`scripts/add_csv_header.py` calculaba el `data_hash` de un DataFrame y escribía
**otro** DataFrame distinto (`pd.read_csv(csv_path).to_csv(index=False)`).
El viaje de ida y vuelta por CSV cambia cómo se imprimen los flotantes, así que
el hash escrito nunca correspondía al fichero resultante: **el fichero se
declaraba "no es el que fue descargado" desde el momento en que se le ponía la
cabecera**. Ahora se serializa una vez y se hashea esa misma serialización.