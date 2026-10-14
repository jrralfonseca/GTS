# Servido, alertas y el camino a live: qué se construyó y qué NO funciona

**Fecha:** 2026-10-14
**Commit:** `bce3e59` — "feat: Monitoring/Alerting stack + Live Trading path complete" (236 ficheros, +19.325 / −123)
**Estado:** el camino a live está **completo y cerrado** (`live.enabled: false`; `demo` es el techo, por decisión del dueño). El servido y la observabilidad funcionan. Pero hay **treinta defectos confirmados**, y cuatro de ellos hacen que una pieza afirme en su nombre algo que no hace.

**Origen:** revisión completa del commit, con lectura de código y verificación de cada afirmación. No es un informe de avance: es un inventario de lo que existe y de lo que no.

---

## 1. La respuesta corta

| Pregunta | Respuesta | Evidencia |
|---|---|---|
| ¿Arranca el bot desde el panel? | **Sí** | pid 16932, heartbeat `2026-10-14T05:46:00+00:00`, `control_seq: 7` |
| ¿Corre en simulación? | **Sí** | `"dry_run": true`, `"launch_dry_run": true` en `/api/bot/status` |
| ¿El panel lo lanza, pausa y reanuda? | **Sí** | seq 1 → 6 (aplanar) → 7 (reanudar), todas con respuesta del bot |
| ¿El servidor de modelo responde? | **Sí, medido** | HTTP 200, ~21 ms de latencia en `/predict` |
| ¿Las alertas a Telegram llegaron? | **Sí, pero NO desde el bot** | entregado al grupo `-5504195034`… por el script suelto, no por el bot. Ver §5.1 |
| ¿Hay tests sobre el trabajo nuevo? | **No. Cero.** | 13 módulos, 0 referencias en `tests/` y `dashboard/tests/` |
| ¿Hay un secreto en el repositorio? | **Sí** | token de Telegram en claro en 2 ficheros versionados. Ver §5.2 |
| ¿El commit arrastró basura? | **Sí, 176 ficheros** | `tmp_pytest*/`, `data/tests/`, logs, scripts de prueba |

Las dos primeras filas son las buenas. Las cuatro siguientes son el precio de haber escrito quince piezas sin una sola valla.

---

## 2. Lo que hay ahora

### 2.1 Servido del modelo (`serving/`)

`serving/model_server.py` (708 líneas) — FastAPI + ONNX Runtime. Expone:

| Endpoint | Qué hace |
|---|---|
| `GET /health` | vivo, modelo cargado, versión |
| `POST /predict` | regime · direction · volatility · size por símbolo y temporalidad |
| `GET /model/info` | metadata del modelo |
| `GET /metrics` | exposition de Prometheus |

**Nueve** métricas, no "15+" como afirma el mensaje del commit:

```
gts_model_requests_total · gts_model_request_latency_seconds · gts_model_loaded
gts_model_version · gts_predictions_total · gts_prediction_latency_seconds
gts_prediction_pnl_r · gts_model_memory_mb · gts_model_uptime_seconds
```

`serving/export_onnx.py` (151 líneas) convierte el checkpoint `.pt` en `.onnx` con `onnx.checker.check_model` como verificación. Entradas `ohlcv_features, asset_ids, timeframe_ids`; salidas `regime_logits, direction_logits, volatility_logits, size_logits`.

### 2.2 Observabilidad

| Pieza | Fichero | Estado |
|---|---|---|
| Prometheus | `config/prometheus.yml` | 2 trabajos de scrape: `gts-model-server` (:8080) y `gts-trading-loop` (:9090) |
| Grafana | `grafana/dashboards/gts-trading-monitor.json` | 12 paneles, aprovisionados |
| Alertmanager | `config/alertmanager.yml` | receptor Telegram |
| Composición | `docker-compose.monitoring.yml` | Prometheus :9090 · Grafana :3000 · Alertmanager :9093 |

**Medido hoy, en la máquina:** los tres contenedores contestan. `:9090`, `:9093` y `:3000` aceptan conexiones.

### 2.3 Alertas (`scripts/alerts.py`, 451 líneas)

Siete eventos, ocho claves:

| Evento | Nivel | Se emite desde |
|---|---|---|
| `fill` | info | `runtime/loop.py:816` (cierre), `:941` (apertura) |
| `rejection` | warning | `runtime/loop.py:924` |
| `disconnect` | error / critical | `runtime/loop.py:763`, `:784` |
| `daily_loss` | warning | `runtime/loop.py:1075` |
| `daily_loss_limit` | critical | mismo punto |
| `model_serving_down` | warning | `AlertMonitor` sondeando `:8080/health` — **nunca desde el bot** |
| `reconciliation_failed` | error | `runtime/loop.py:963` |

Deduplicación 300 s y límite 50/hora, **solo en memoria**: un reinicio del proceso las borra.

### 2.4 El camino a live

| Pieza | Qué hace |
|---|---|
| `core/ports/profiles.py` | `RuntimeConfig.allow_funded`, `AlertsConfig` |
| `adapters/broker/mt5_orders.py` | `order_port_para()` devuelve puerto para DEMO y para LIVE (solo si `allow_funded`) |
| `app/main.py::_perfil_de` | construye el `RuntimeConfig` del perfil |
| `scripts/preflight_live.py` | 8 comprobaciones; devuelve 0/1 |
| `scripts/go_live.py` | reescribe la config con `deep_merge`, pasa el preflight, lanza `app.main` |
| `scripts/flatten.py` | aplanado de emergencia |
| `scripts/reconcile_session.py` | bitácora contra estado del broker |
| `RUNBOOK_LIVE.md` | procedimiento escrito — **marcado DRAFT, y desalineado del código (§5.7)** |

**La doble llave de LIVE.** `execution.live.enabled: true` no basta. Hace falta además `execution.live.allow_funded: true` **y** que `account_info().trade_mode != DEMO`. Las tres condiciones, o ninguna.

### 2.5 El panel

`dashboard/templates/_bot_vivo.html` — el `<select>` de "Fuente de señal" lleva `hx-preserve="true"`, igual que el `<details>` que lo envuelve. Con eso la selección sobrevive al refresco de 2 s.

**Verificado de punta a punta:** `POST /api/signal/source {"source":"model","reason":"test switch to model"}` → 200; `data/control/signal_source.json` escrito con `requested_by: "panel"`.

---

## 3. Los tres pilares: qué es código y qué es andamiaje

El trabajo de las sesiones anteriores se presentó como "pilares". Con la revisión hecha, la Picture honesta es esta:

| Pilar | Módulo | Implementación | Validación | Operación |
|---|---|---|---|---|
| 1 · Filtro EV | `risk/entry_filter.py` | existe | **no_probada** (0 tests) | no_operativa (nadie lo llama con datos reales) |
| 2 · Walk-forward | `jobs/runners/walkforward_v2.py` | **parcial** | no_probada | **no_operativa** (`NameError` en el primer tick de progreso) |
| 3 · Etiquetado PnL | `research/pnl_labeling.py` | **parcial** | no_probada | **no_operativa** (`NameError`: `defaultdict` sin importar) |

Y un cuarto, que no es un pilar sino una advertencia:

| `risk/ml_integration.py` | **stub** | no_probada | no_operativa |
|---|---|---|---|
| `MLInferenceEngine.load()` es un `# TODO`; `predict()` devuelve `p_long=0.33` escrito a mano; `_regime_adjustment()` devuelve `0.0`. Nadie lo invoca. | | | |

Los tres pilares están escritos con una documentación que explica la intención muy bien. Ninguno se ha ejecutado nunca de punta a punta. La diferencia entre "existe" y "funciona" es exactamente la que este mapa de componentes existe para no confundir.

---

## 4. Lo que sí está medido

| Qué | Cómo | Resultado |
|---|---|---|
| Pre-flight demo | `python -m scripts.preflight_live --config config/gts.yaml` | **PASS** — MT5 build 6259, cuenta 10012953169, Algo Trading ON, XAUUSD OK, 10.000 USD de margen libre |
| Bot en marcha | `POST /api/bot/launch` + `/api/bot/status` | `running: true`, `dry_run: true`, heartbeat fresco |
| Control por panel | `POST /api/bot/start` | seq 6 → 7, `state: ready` |
| Servidor de modelo | `POST :8080/predict` | 200 OK, ~21 ms |
| Alertas Telegram | `python -m scripts.alerts --test` | entregado en el grupo `-5504195034` |
| Torre de tests del trabajo nuevo | `pytest` sobre 7 ficheros | ver `EVIDENCE.yaml` |

---

## 5. Los treinta defectos

Ordenados por lo que cuesta que no se detecten solos.

### 5.1 CRÍTICO — El bot no puede emitir alertas. Por dos motivos encadenados

**Motivo 1: no hay canales.** `app/main.py:189`

```python
return RuntimeConfig(profile=perfil, policy=RiskPolicy.ENFORCE, allow_funded=allow_funded)
```

No pasa `alerts=`. El `AlertManager` de `runtime/loop.py:635` recibe siempre el
`AlertsConfig()` por defecto, con `telegram.enabled=False`, y registra
*"No alert channels configured"*. El token de Telegram del YAML **no llega al
bot**.

**Motivo 2: aunque llegaran, no saldrían.** Los seis manejadores de
`AlertMonitor` (`scripts/alerts.py:361-383`) son **síncronos** y hacen
`asyncio.create_task(self.alert_manager.alert(...))`. Los llama
`runtime/loop.py` desde su bucle síncrono, donde no hay bucle de eventos:
`create_task` falla y la corrutina se descarta sin ejecutarse.

No es una hipótesis. La suite lo dice en su propia salida:

```
tests/test_cableado_demo.py::test_demo_con_puerto_si_envia_por_este_puerto
  RuntimeWarning: coroutine 'AlertManager.alert' was never awaited
```

**Consecuencia:** el canal funciona —lo que se verificó es que
`scripts/alerts.py --test` entrega bien— pero el **cableado no**. Si el bot
entrara en pérdida diaria, o perdiera la conexión, o le rechazaran una orden,
no diría nadie. Todo §2.3 es decorativo hasta que se arreglen las dos capas.

### 5.2 CRÍTICO — Un token de Telegram en claro dentro del repositorio

`config/gts.yaml:471` y `config/alertmanager.yml:27`:

```
8505587639:AAGmzuOiAN0SM2oP7XlSMnKfKK7R0TZtUrM
```

Dos veces, en dos ficheros versionados. Cualquiera con el repositorio puede escribir en ese grupo. Necesita: revocar en @BotFather, sacar el valor a variable de entorno, y limpiar el historial.

### 5.3 CRÍTICO — Dos `AlertsConfig` distintos

`app/config.py:289` y `core/ports/profiles.py:130`. Son la puerta de 5.1: el YAML carga uno, el bucle lee el otro, y no se habla.

### 5.4 ALTO — Cero cobertura sobre los 13 módulos nuevos

| Módulo | Tests |
|---|---|
| `serving/model_server.py` · `serving/export_onnx.py` | 0 |
| `scripts/alerts.py` · `go_live.py` · `preflight_live.py` · `flatten.py` · `reconcile_session.py` | 0 |
| `risk/entry_filter.py` · `ml_integration.py` · `regime_integration.py` · `strategy_performance.py` | 0 |
| `research/pnl_labeling.py` · `jobs/runners/walkforward_v2.py` | 0 |

Los dos ficheros que los tocan (`test_pnl.py`, `test_server.py`, en la raíz) **no son tests**: no tienen función de test ni `assert`, solo `print` dentro de un `try/except`. Además, `test_pnl.py` y `test_server.py` están commiteados.

### 5.5 ALTO — Dos `NameError` que impiden ejecutar los pilares 2 y 3

| Fichero | Qué falta |
|---|---|
| `research/pnl_labeling.py:563, 609` | `defaultdict` se usa, nunca se importa |
| `jobs/runners/walkforward_v2.py:273, 277, 279` | `state`, `total_folds`, `current_symbol` no están definidos en el ámbito de `_progress_cb` |

El primero revienta en el primer epoch. El segundo, en el primer tick de progreso.

### 5.6 ALTO — 176 ficheros de basura en el índice

`bce3e59` se hizo con `git add -A`. Se llevó por delante:

- ~171 artefactos de `tmp_pytest*/`, `data/tests/` y `_otra_forma0/` — resultados de tests, no código
- `panel-prueba.log`, `server.log`, `sweep_XAUUSD_M15.log`, `sweep_XAUUSD_M15.err`
- `testmt5.py`, `test_pnl.py`, `test_server.py`

Ninguno pertenece al sistema. El `.gitignore` se modificó en el mismo commit, pero **después** de añadir.

### 5.7 MEDIO — `RUNBOOK_LIVE.md` describe comandos que no existen

| Lo que dice | Lo que es |
|---|---|
| `python -m gts flatten --force` (§4) | `python -m scripts.flatten --config config/gts.yaml --force` |
| `python -m gts stop` (§4) | no existe ningún CLI `gts` |
| `sudo systemctl start gts-bot` (§2) | no hay unidad systemd; y estamos en Windows |
| `/health` del bot (§2 paso 4) | el bot no expone `/health`; ese puerto es el del servidor de modelo |
| "Journal hash chain intact" (§5) | `reconcile_session.py` no verifica ninguna cadena de hash |
| `--daemon` en `alerts.py` | documentado, no implementado |
| "Step 4" | aparece dos veces (§2) |

El documento está marcado **DRAFT**. Lo correcto es arreglarlo antes de que alguien lo siga con dinero de verdad.

### 5.8 MEDIO — Prometheus raspa un endpoint que no existe

`config/prometheus.yml:30-37` declara el trabajo `gts-trading-loop` contra `host.docker.internal:9090/metrics`. **El bot no expone `/metrics`.** Y el 9090 es el puerto del propio Prometheus. El trabajo no puede estar bien.

### 5.9 MEDIO — Más código duplicado dentro del mismo fichero

| Fichero | Duplicado |
|---|---|
| `serving/model_server.py` | `load_pytorch` en :195 y :341 · `load_onnx` en :221 y :367 · bloque muerto en :470-515 (tras un `return`) |
| `risk/entry_filter.py` | `DEFAULT_EV_FILTER` en :171 y :264 |
| `jobs/runners/walkforward_v2.py` | `_create_regime_classifier` en :194 y :213 · escritura atómica en :245 y :258 · **11 campos de dataclass repetidos** |
| `research/pnl_labeling.py` | 6 campos repetidos entre `PnLLabelConfig` y `PnLLabelResult`; el bloque de imports está pegado dos veces |

Todos del mismo tipo: el fichero se montó concatenando, no leyendo. Los duplicados de dataclass no son cosméticos: en Python el último gana, así que `min_holding_bars` vale 5 aunque la primera vez dijera otra cosa, y nadie lo ve.

### 5.10 MEDIO — Magia cableada en dos scripts de emergencia

`scripts/flatten.py:82` y `scripts/reconcile_session.py:78`: `magic = 123456`, los dos con `# TODO: get from config`. `app/main.py` sí usa `cfg.system.bot_magic`. Un script de emergencia que no ve la misma cuenta que el bot es un script de emergencia que no cierra las posiciones del bot.

Además `flatten.py:29` (`build_runtime_config`) recibe dos parámetros de override, no los usa, tiene un `pass` de cuerpo y **nadie lo llama**.

### 5.11 BAJO — El resto

| # | Dónde | Qué |
|---|---|---|
| 11 | `entry_filter.py:57` | `min_win_prob` se declara y documenta, y `evaluate()` no lo lee |
| 12 | `strategy_performance.py:10, 87` | `sqlite3` importado sin usar; el registro se carga y se tira |
| 13 | `strategy_performance.py:266-268` | tres métricas a `0.0` / `0` con un TODO: `avg_holding_bars`, `max_consecutive_losses`, `max_drawdown_r` |
| 14 | `alerts.py:356` | `_check_reconciliation()` es un `pass` — y el monitor nunca se arranca desde el bot |
| 15 | `reconcile_session.py:31` | si la bitácora no existe devuelve `[]`, y una bitácora rota reconcilia como "0 contra N" |
| 16 | `preflight_live.py:170` | los directorios se comprueban contra el directorio de trabajo, no contra la raíz del repositorio |
| 17 | `config/walkforward.yaml` | claves duplicadas: `min_regime_confidence`, `conservative`, `broker` |
| 18 | `config/train_pnl.yaml:17, 32` | `label_config:` declarado dos veces |
| 19 | `walkforward_v2.py:10` vs `:428` | el docstring anuncia `--config`; `main()` no lo tiene, y el YAML nunca se lee |
| 20 | `runtime/loop.py:1078` | lee `max_day_loss_pct` con defecto `0.02` en código; el YAML dice `3.0`. Si el YAML no trae la clave, el freno es veinte veces más blando |

---

## 6. Lo que se paga

- **Se compró velocidad de escritura y se pagó en vallas.** Quince módulos nuevos, tres commits, ninguna prueba. La mitad de los defectos de §5.5 y §5.9 los habría cazado un test de importación.
- **La puerta de live es sólida y el camino que la rodea no lo es.** `preflight_live` + `allow_funded` + `trade_mode` son un buen freno. El runbook que dice cómo usarlos miente en siete sitios, y el script que aplana posiciones tiene el número mágico escrito a mano.
- **Lo verificado por el camino corto no es lo verificado de verdad.** "Las alertas funcionan" es cierto del script y falso del bot. Esa frase era la que iba a sostener una decisión de capital.

---

## 7. Lo que NO se ha medido

- Que las alertas salgan del bot (§5.1: no pueden salir).
- Un epoch de `PnLTrainingConfig` (§5.5: `NameError` en el primero).
- Un solo fold de `walkforward_v2` (§5.5: `NameError` en el primer tick).
- Un `/predict` con la configuración real del bot: el modelo servido y el que el bot usaría (`data/models/*.json`) **no son el mismo formato**. El bot no consume `.pt` ni `.onnx`; consume un `.json` de `ModelStrategy`, y hoy no hay ninguno en disco.
- Si Grafana está mostrando algo: los paneles están aprovisionados, pero el servidor de modelo está caído y el trabajo de Prometheus que declara `gts-trading-loop` no existe.

---

## 8. Lo que corresponde

En orden de coste, con el detalle en `ISSUES.yaml` y el camino en `ROADMAP.yaml`:

1. **Revocar el token de Telegram** y sacarlo del repositorio. Es lo único que se puede arreglar desde fuera.
2. **Cablear `alerts=` en `app/main.py:189`** y unificar los dos `AlertsConfig`. Sin esto, todo §2.3 es decorativo.
3. **Borrar los 176 ficheros de basura del índice** (nunca del disco del trabajo).
4. **Escribir tests de importación y de contrato** para los 13 módulos: dos `NameError` y doce duplicados caen con eso.
5. **Arreglar `RUNBOOK_LIVE.md`** o marcarlo como no ejecutable. Un runbook con comandos inventados es peor que no tener runbook.
6. Decidir qué pasa con el servidor de modelo: o se conecta al `ModelStrategy` `.json` que el bot entiende, o se admite que son dos caminos distintos y se documenta.