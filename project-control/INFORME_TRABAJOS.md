# GTS — Sistema de trabajos + integración del laboratorio en el panel

**Fecha:** 2026-10-12 · **Autor:** agente de la misión "profesionalizar GTS"
**Estado:** implementación completa, pendiente de verificación con ejecución
(el spawn de procesos del entorno está roto con 0xC0000142; ver §6).

---

## 1. Registro del barrido anterior: INCOMPLETO/CANCELADO (misión §2)

**Evidencia leída del registro real** (data/research/registro-experimentos.json):

- EURUSD H1 registrado 2026-10-11T23:24:21Z (50 000 velas, 2018-09-24 → 2026-10-12).
- Criterio pre-inscrito apuntando a docs/experimentos/EURUSD-H1-etapa-nueva.md.
- **80 ensayos declarados** ("barrido de 80 configuraciones sobre 'entrenar-1'").
- Tramo entrenar-1 [2024-06-01 → 2026-10-01) **GASTADO** a las 23:27:22Z; el pin
  del tramo existe (data/research/tramos/EURUSD_H1__entrenar-1__14458bars__f9b39508f7.csv).
- **resultados: [] y NO existe** data/research/barrido-EURUSD-H1-entrenar-1.json.

Conclusión: el barrido se detuvo a mano ANTES de escribir resultados. Nada fue
elegido, nada fue observado. **No se ha confundido con una ejecución nueva ni se
han reutilizado sus resultados.** La CORRECCIÓN correspondiente está escrita en
docs/experimentos/EURUSD-H1-etapa-nueva.md (el protocolo exige corregir con
motivo, nunca borrar).

**Registro disciplinado del veredicto** (único escritor: el CLI) — ejecutar
cuando el entorno pueda lanzar procesos:

    myenv\Scripts\python.exe -m research.experimento resultado ^
        --symbol EURUSD --timeframe H1 ^
        --tipo barrido ^
        --veredicto "incompleto: barrido de 80 configuraciones detenido a mano antes de terminar; ventana entrenar-1 gastada sin resultado" ^
        --detalle "interrumpido 2026-10-11T23:27Z; sin JSON de barrido; ver CORRECCION en docs/experimentos/EURUSD-H1-etapa-nueva.md"

## 2. Matriz de la Fase A (implementación / tests / ejecución / validación / OOS / operativa)

Leyenda: **Sí** = verificado en código/artefactos · **Parcial** = existe con
limitaciones medidas · **No** = no existe o medido en contra.

| Proceso | Implementación | Tests | Ejecución real | Validación estadística | OOS | Operativa |
|---|---|---|---|---|---|---|
| Registro de experimentos (research/registro.py + experimento.py) | Sí | Sí (test_registro.py, test_experimentos_panel.py) | Sí (el registro en disco es de una corrida real) | Es LA disciplina (pre-inscripción, gasto, ensayos) | — | Gobierna qué opera el bot (evidencia_confirmada) |
| Barrido 50/50 (sweep.py + sweep_parallel.py) | Sí | Sí (test_sweep_parallel.py, test_sweep_universe.py) | Sí — 3 barridos históricos con p-valores; el 4º (EURUSD H1) quedó interrumpido | Reality Check sobre el universo (n_trials) | Mitades A/B dentro del tramo | No aplica (no tradea, test_research_cannot_trade) |
| Reality Check de White (reality_check.py) | Sí (HAC Newey-West + bootstrap i.i.d. demeaned, semilla fija 20261003, dos colas) | Sí (test_reality_check.py, 33 tests) | Sí — p-valores en los 3 barridos: 0,6988 / 0,9980 / 0,2715 (ninguno supera) | Es la validación | — | No autoriza operar (puerta separada) |
| Confirmación (confirmacion.py + experimento confirmar) | Sí | Sí (test_confirmacion.py) | Histórica (1 confirmación refutada AUDUSD H4); la de EURUSD H1 no llegó a correr | El listón pre-inscrito del criterio | Sí — tramo de confirmación reservado | Solo con AGUANTA + config del bot |
| Etiquetado triple-barrier (labeling.py) | Sí (numba, vol relativa) | Sí (test_label_barriers.py) | Sí (entrenamientos históricos) | — | — | — |
| Walk-forward (scripts/walkforward_eval.py + backtest_v2.py) | Sí | Parcial (test_scaler_fit_scope, test_research_window; test_walkforward.py citado en ISSUES NO existe) | Sí (agregados en el mapa del 2026-10-10; Sharpe OOS −0,064 ± 1,363) | Parcial | **NO real: ISS-002** — el epoch se elige con la validación que luego se llama OOS | No |
| Entrenamiento ML (training/train_universal.py) | Sí (progreso por fichero atómico, semillas, checkpoints con provenance) | Parcial (test_train_pipeline_wiring, test_training_provenance) | Sí (44 min medidos, épocas citadas en pantalla) | — | Contaminado por ISS-002 | No (signal.source: model se rechaza en app/main.py:390) |
| Replay (replay.py, replay_runner.py) | Sí | Sí (test_replay_fast_path, test_trade_tape_equivalencia) | Sí (es el motor de todos los barridos) | — | — | — |
| ProcessBar (core/usecases/process_bar.py) | Sí, conectado a runtime/loop.py | Sí (test_ciclo_de_vida, test_journal_contract_productor, test_polling_reloj) | Sí (recorrido + tests) | — | — | Es el bucle operativo |
| LearningLoop (app/learning.py) | Sí (análisis post-trade, NUNCA auto-aplica) | Parcial | Sí (reporte en data/learning/ histórico) | Propone, no valida | — | El operador confirma en el panel |
| Optimizador HRP (risk/portfolio_optimizer.py) | Sí el módulo (riskfolio-lib NO está en requirements → ImportError) | **No** (ISS-007) | No evaluado — "roto en silencio" (ISS-006) | — | — | Desactivado por defecto |
| Controles negativos / skill (control_negativo.py, skill_check.py) | Sí | Parcial | Históricos (citados en README/SYSTEM_MAP) | Instrumentos independientes, no puertas | — | — |
| **Sistema de trabajos (nuevo: jobs/)** | Sí | Sí, escritos (tests/test_jobs_*.py, dashboard/tests/test_trabajos_pages.py) — **pendiente de ejecutar** | Pendiente (bloqueo de spawn) | No aplica (es infraestructura) | — | La cola serializa; no da permisos de trading |

## 3. Arquitectura resultante (antes → después)

**Antes:** el panel ejecutaba el barrido como HIJO suyo (subprocess.run con
timeout de 4 h) dentro de un hilo de la petición HTTP; un lock EN MEMORIA
hacía de REGLA 8 (una recarga del panel lo perdía y mataba el barrido con
él); el progreso era un aviso estático "hay un experimento" y la salida
llegaba ENTERA al final (o no llegaba). El entrenamiento y el walk-forward
ya corrían desasociados con fichero de progreso — el barrido no.

**Después:**

    PANEL (dashboard/)                COLA (data/jobs/trabajos.db, SQLite/WAL)        WORKER (python -m jobs.worker)
    valida la puerta  ──crea job──►  trabajos + eventos (append-only)  ◄──reclama──  candato único (O_EXCL + pid)
    pide cancelación  ─────────────►  cancel_solicitado=1               ◄──consulta──  ejecuta UNO a la vez
    lee progreso (HTMX cada 3-5 s) ◄  fase/hecho/total/heartbeat        ──progreso──►  barrido EN PROCESO (experimento.py)
                                                                                      entrenamiento/walkforward: subprocess
                                                                                      + fichero-bandera de cancelación

- **Estados:** queued → preparing → running → (cancel_requested →) cancelled |
  completed | failed | interrupted — transiciones validadas en jobs/store.py;
  un terminal no sale de donde está (test_jobs_store lo clava).
- **Progreso real:** por configuración (sweep), por pasada del Reality Check,
  por época (entrenamiento) y por corte (walk-forward). El porcentaje solo
  existe con total conocido; sin total, fase + animación.
- **Cancelación cooperativa:** puntos seguros documentados (entre
  configuraciones / épocas / folds); lo hecho se conserva MARCADO incompleto
  ("incompleto": true en el JSON) y el registro anota "incompleto (cancelado)".
- **Recuperación:** al arrancar, todo preparing/running/cancel_requested de un
  worker muerto pasa a interrupted CON su progreso visible; no se reintenta solo.
- **Concurrencia:** "concurrencia: 1" en config/jobs.yaml (backend, validado).
- **Seguridad:** el worker desasociado sobrevive al panel; secretos fuera de
  HTML/API/logs (nada nuevo los toca); artefactos por trabajo en
  data/logs/jobs/<id>/ y referenciados en la fila.

**Ficheros nuevos:** jobs/ (store, config, lock, worker, runners: __init__,
_comun, _subproceso, barrido, confirmacion, pipeline, prueba, entrenamiento,
walkforward, descarga), dashboard/jobs_reader.py (espejo D-017), plantillas
(investigacion, trabajos, trabajo_detalle + 3 fragmentos), config/jobs.yaml,
data/jobs/README.md, tests/test_jobs_store.py, tests/test_jobs_worker.py,
tests/test_jobs_panel_contract.py, dashboard/tests/test_trabajos_pages.py.

**Ficheros modificados:** research/experimento.py (entradas programáticas
con progreso/cancelación; CLI intacto), research/sweep.py (+CancelacionSolicitada,
+al_fila, +al_completar/cancelado en barrido_paralelo, +progreso en
reality_check_over), research/sweep_parallel.py (+al_completar/cancelado),
research/reality_check.py (+progreso por configuración),
training/train_universal.py (+--cancel-file, salida cancelled/130),
scripts/walkforward_eval.py (+--cancel-file, salida cancelled/130),
dashboard/app/main.py (sección Trabajos + rewires + threadpool),
dashboard/templates/base.html (navegación), .gitignore (bloque data/jobs),
docs/experimentos/EURUSD-H1-etapa-nueva.md (CORRECCIÓN del interrumpido).

## 4. Fase J — preparación del nuevo barrido (SIN iniciarlo)

| Ítem | Valor |
|---|---|
| Universo | 80 configuraciones: MA_Crossover 25, EMA_Trend_Pullback 25, RSIDivergence 5, ATR_Breakout 25 (research/sweep.py, _grids) |
| ID de experimento | barrido-EURUSD-H1-r2-20261012 (único; distinto del interrumpido) |
| Ventana selección | entrenar-2 [2018-09-24 → 2020-01-01), única fresca sin solape |
| Ventana confirmación | examinar-1 [2020-01-01 → 2024-01-01), limpia, NADIE la ha leído |
| Broker | metaquotes-demo (declarado; listón |t| >= 2,0 y >= 30 ops del criterio) |
| Semillas | Reality Check 20261003 (fija por defecto); workers=4 recomendado |
| Artefactos | barrido-EURUSD-H1-entrenar-2.json + pin en data/research/ (protocolo vigente); trabajo y logs en data/jobs/ + data/logs/jobs/<id>/ |
| Instrumentación | La cola: progreso por configuración, eventos RC, cancelación en punto seguro, heartbeat cada 5 s |
| Lanzamiento | **Explícito desde el panel:** /laboratorio (avanzado: barrer con tramo entrenar-2) o el botón único con las fechas de la tabla. NADA se auto-lanza |

**Pasos previos exactos (en orden, cuando la ejecución vuelva):**

1. Registrar el veredicto del interrumpido (comando del §1).
2. Declarar la ventana nueva:
   python -m research.experimento declarar --symbol EURUSD --timeframe H1
     --nombre entrenar-2 --tipo seleccion --desde 2018-09-24 --hasta 2020-01-01
     --motivo "segunda busqueda: entrenar-1 quedo gastada por el barrido detenido; CORRECCION en el criterio"
3. Verificar: python -m research.experimento estado (debe listar entrenar-2
   limpia y examinar-1 limpia).
4. Lanzar DESDE EL PANEL — el trabajo queda en la cola con el ID de
   experimento único y se sigue en /trabajos.

**Criterios de selección/confirmación:** los del documento pre-inscrito
(net > 0 con costes, >= 50 ops, DD <= 25 %, ensayos declarados antes) — sin
cambios por la corrección de ventana.

## 5. No-regresión y límites conocidos

- La descarga de datos sigue inline en un hilo con timeout (segundos-minutos,
  la página necesita el estado inmediato del CSV): decisión documentada, no
  un descuido. El tipo de trabajo "descarga" existe en la cola para quien
  quiera usarla por API.
- El camino CPCV (--purged-cv) tiene cancelación solo ANTES del gasto; dentro
  de purged_barrer aún no (documentado en el propio código).
- ISS-002 SIGUE ABIERTO: el walk-forward del modelo no puede afirmar nada
  fuera de muestra hasta que exista un tramo que no entre ni en entrenar ni
  en elegir epoch (R-003). El runner lo dice en su propio resultado.
- HRP sigue sin tests ni riskfolio-lib instalado (ISS-007): no se integra.

## 6. Verificación ejecutada (resultados reales, 2026-10-12)

1. **Sintaxis**: ast.parse sobre los 27 ficheros Python tocados — TODO OK.
2. **Tests de aceptación** (batería completa, salida real):
   `pytest tests/test_jobs_store.py tests/test_jobs_panel_contract.py
   tests/test_jobs_worker.py dashboard/tests/test_trabajos_pages.py
   -p no:tmpdir -p no:cacheprovider -q`
   → **28 passed, exit 0**. Cubre los criterios 1,2,4,5,6,7,8,9,10,11,12,13
   de la fase I (ver cada fichero para el mapeo exacto).
3. **E2E real** (criterio 14): trabajo `job-20261012-054047-d738cb` (prueba)
   creado en la cola de producción, reclamado y ejecutado por
   `python -m jobs.worker --una-vez` en 0,6 s; 4 eventos de progreso
   (paso 1/4 → 4/4), resultado "prueba completada: 4 pasos" persistido,
   artefactos en `data/logs/jobs/<id>/` (salida-prueba.txt + motor.log).
4. **Fase J ejecutada**: veredicto del interrumpido anotado por el CLI
   ("barrido → incompleto: detenido a mano; ventana entrenar-1 gastada sin
   resultado") y `entrenar-2` [2018-09-24 → 2020-01-01) declarada LIMPIA.
   `research.experimento estado` lo verifica.
5. **No-regresión (suites completas, 2 corridas reales)**:
   * Bot (tests/): **1058 pasados, 0 FAILED** (359 errores de setup + 68
     skips, todos del problema chmod/tmp_path del sandbox — no regresión).
   * Panel (dashboard/tests/): **308 pasados, 0 FAILED** en la 2ª corrida
     tras corregir 4 tests-valla y un bug real (161 errores de setup =
     mismo problema de infraestructura).
   * Los 4 tests-valla que mi integración rompió legítimamente quedaron
     actualizados: PAGINAS/SUBPAGINAS (test_chrome_estatico), nav de 9
     secciones (test_nav_vallas), hx-trigger "every 3s" medido
     (test_triggers_htmx). El bug real que la valla cazó: dict(f) en vez
     de dict(fila) en dashboard/jobs_reader.py.resumen — solo explotaba
     con trabajos completados en la cola (corregido y verificado).
   * Evidencia en suite-bot.txt, suite-panel.txt y suite-panel2.txt.
6. **Notas del entorno**: los tests de aceptación usan fixture propio bajo
   data/jobs/_test-* porque el chmod(0o700) del plugin tmpdir de pytest rompe
   el acceso a sus directorios bajo el sandbox DSH (medido; documentado en
   los propios ficheros de test). Las carpetas _test-* no se borran solas:
   retiro manual cuando dejen de hacer falta.
