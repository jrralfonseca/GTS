# GTS — Puertas de validación

**Revisión:** `e801d4e` · **Fecha:** 2026-10-10

Este fichero dice **qué tiene que ser cierto** para pasar de una etapa a la
siguiente. Una tarea no está completada porque su código esté escrito: está
completada cuando su puerta está en verde.

Las etapas son deliberadamente restrictivas. El sistema no puede llegar a operar
capital real hasta pasar la Etapa 4, y hoy está en la Etapa 1.

---

## La escalera

```
   Etapa 0 ──► Etapa 1 ──► Etapa 2 ──► Etapa 3 ──► Etapa 4
   escribir    medir      barrer     confirmar    operar
   codigo      repetir    en H1      fuera de     con capital
                         el TF que   muestra      real
                         corre
     ▲           ▲          ▲           ▲           ▲
  HOY ESTA   R-001      R-003      R-003       (lejos)
```

**Estado real del 2026-10-10: Etapa 1.** La etapa 0 tiene código; la etapa 1 está
roja por tres fallos de arnés.

---

## Reglas que aplican a TODAS las etapas

Estas no son etapas: son condiciones que no se negocian en ninguna.

### R-A — El basetemp va fuera del repositorio

```powershell
$env:TMP="C:\...\opencode"
myenv\Scripts\python.exe -m pytest tests --basetemp="C:\...\opencode\pt"
```

**Por qué:** `tests/test_research_cannot_trade.py` recorre el repo con `rglob` en
busca de `order_send`. Con el temporal dentro del árbol, encuentra el código de
otros tests y falla sin que exista ninguna violación real. Falso positivo
documentado en la auditoría del 2026-10-09.

### R-B — Ninguna cifra sin su N

Un Sharpe sin el número de folds, o un p-valor sin el tamaño del universo, no se
reporta. Con N ensayos, el máximo esperado es `sqrt(2 ln N)/T` aunque todas las
estrategias valgan cero en población.

### R-C — Un skip no es un pass

Si un guardián se salta porque le falta un fichero que **existe**, es un fallo
disfrazado. Si el fichero no existe, el test debe **fallar** diciendo qué falta.

### R-D — Ninguna tarea se marca completa si su criterio de aceptación no se cumple

Incluido este mapa. Un componente no pasa de `implementacion` a `validada` porque
el código "parece bien": pasa porque un test lo ejerce y ese test está verde.

---

## Etapa 0 → Etapa 1: el código es ejecutable y repetible

**Pregunta que responde:** ¿puedo volver a obtener el mismo resultado mañana?

| # | Condición | Cómo se comprueba | Estado |
|---|---|---|---|
| 0.1 | Las dos suites coleccionan sin errores de import | `pytest tests --collect-only` y `pytest dashboard/tests --collect-only` | **VERDE** — 1384 y 411 |
| 0.2 | La suite del bot sale en verde | `pytest tests` → exit 0 | **ROJA** — 3 fallos (ISS-001) |
| 0.3 | La suite del panel sale en verde | `pytest dashboard/tests` → exit 0 | **VERDE** — 411/411 |
| 0.4 | `ruff` y `mypy` limpios | `ruff check .` y `mypy` | **NO MEDIDO** |
| 0.5 | El bundle de entrenamiento no contamina `sys.path` | `pytest tests/test_kaggle_layout.py tests/test_sweep_parallel.py` → exit 0 | **ROJA** (ISS-001) |

**Sale de la etapa cuando:** 0.2 y 0.5 en verde, y 0.4 medido y registrado en
`EVIDENCE.yaml` (aunque sea para decir que falla).

**Tarea que la desbloquea:** R-001.

---

## Etapa 1 → Etapa 2: lo que se mide está protegido contra el azar

**Pregunta que responde:** ¿el número que tengo es real, o es ruido con suerte?

| # | Condición | Cómo se comprueba | Estado |
|---|---|---|---|
| 1.1 | El barrido registra el universo y la serie de retornos | `n_trials`, `configurations`, `trade_returns_b` por fila | **VERDE** — los 3 barridos en disco |
| 1.2 | El p-valor es sobre el universo, no el `\|t\|` del ganador | Campo `reality_check.p_value` | **VERDE** — 0,6988 / 0,9980 / 0,2715 |
| 1.3 | La corrección está calibrada contra ruido y contra edge real | `test_reality_check.py` (33 tests) | **VERDE** |
| 1.4 | El barrido paralelo da los mismos números que en serie | `test_sweep_parallel.py` | **ROJA** en suite completa (ISS-001); **VERDE** aislada |
| 1.5 | Los ensayos se declaran **antes** de correr | `historial_ensayos` en el registro | **VERDE** — 50 / 150 / 75 declarados |
| 1.6 | Los tramos usados para elegir quedan gastados | `estado: gastada` con motivo y fecha | **VERDE** |
| 1.7 | Ningún guardián de la evidencia se salta en silencio | `pytest -rs` → 0 skips de evidencia | **ROJA** — 5 skips (ISS-006) |
| 1.8 | El leakage del scaler está cerrado y vallaado | `test_scaler_fit_scope.py` | **VERDE** |

**Sale de la etapa cuando:** 1.4 y 1.7 en verde.

**Tareas que la desbloquean:** R-001 (1.4), R-006 (1.7).

---

## Etapa 2 → Etapa 3: el resultado se sostiene fuera de la muestra

**Esta es la puerta que hoy no existe, y es la más importante.**

**Pregunta que responde:** ¿esto aguanta en barras que nadie ha mirado?

| # | Condición | Cómo se comprueba | Estado |
|---|---|---|---|
| 2.1 | Existe un tramo que no entra ni en entrenamiento ni en selección de epoch | La partición de `create_data_loaders` tiene un tercer tramo | **ROJA** (ISS-002) |
| 2.2 | La selección del checkpoint no se hace sobre el tramo que se reporta como fuera de muestra | El epoch se elige antes del corte del held-out | **ROJA** — el epoch se eligió en la fila 45010 |
| 2.3 | Existe ≥ 1 fold cuyo test empieza **después** del fin de la validación | Conteo declarado por el propio programa | **ROJA** — hoy: 0 de 148 en sentido estricto |
| 2.4 | El agregado declara cuántos folds hay de cada tipo, con su N | `in_sample_folds` / `out_of_sample_folds` | **VERDE** — 112 / 28 (pero con el corte equivocado) |
| 2.5 | La dispersión se reporta junto al valor | `sharpe_std` junto a `sharpe_mean` | **VERDE** — −0,064 ± 1,363 |
| 2.6 | El resultado sobrevive a haber sido elegido | p de White < 0,05 **en el tramo de confirmación** | **ROJA** — el único-tramo-gastado es una confirmacion refutada |

**Sale de la etapa cuando:** 2.1, 2.2 y 2.3 en verde, y hay al menos una cifra
fuera de muestra con N y dispersión.

**Tarea que la desbloquea:** R-003 (partición). La forma cara (reentrenar por
fold) es posterior y no es necesaria para abrir la puerta.

**Nota sobre el resultado actual:** la etapa 2 **no se puede saltar** por
"ya tengo un Sharpe de +3,7 en la cabeza de volatilidad". Ese número es un
control positivo sobre la misma muestra, y el criterio preinscrito del
experimento `vol-head-product` dice explícitamente que **no se puede correr**
mientras no exista el tramo reservado.

---

## Etapa 3 → Etapa 4: la evidencia autoriza al bot

**Pregunta que responde:** ¿el sistema tiene permiso de operar lo que opera?

| # | Condición | Cómo se comprueba | Estado |
|---|---|---|---|
| 3.1 | El registro tiene ≥ 1 par **confirmado** | `evidencia_confirmada()` no vacío | **ROJA** — cero confirmados |
| 3.2 | La confirmación se hizo sobre un tramo limpio, ya gastado | `tramos[*].estado == gastada` | **ROJA** — la única confirmación se hizo sobre `examinar-1` (gastado) pero **refutó** |
| 3.3 | La regla por defecto tiene evidencia en el símbolo y el timeframe que **corre** | `config/gts.yaml` vs `data/research/` | **ROJA** (ISS-009) — se cita un fichero inexistente; el barrido real es de otro símbolo |
| 3.4 | El bot se niega a arrancar sin evidencia | `build_bot()` lanza `RuntimeError` | **VERDE** |
| 3.5 | `live.enabled` es `false` y `allow_funded` es `false` | `config/gts.yaml:363,367` | **VERDE** — dos cerrojos |
| 3.6 | La regla por defecto sobrevive al Reality Check en H1 | p < 0,05 | **ROJA** — p = 0,998 |

**Sale de la etapa cuando:** 3.1 y 3.3 en verde.

**Tareas que la desbloquean:** R-003 → R-009 → R-010.

**Advertencia:** hoy el sistema está parado **por su propia regla**, que es lo
correcto. No hay atajo para la Etapa 4: si no hay evidencia, no se opera, y el
`RuntimeError` es la puerta funcionando.

---

## Etapa 4 → Operación con capital real

**NO SE ALCANZA DESDE ESTE REPOSITORIO.** Se documenta para que quede escrito qué
haría falta, no para anticiparla.

| # | Condición | Estado |
|---|---|---|
| 4.1 | Etapa 3 en verde | **NO ALCANZABLE** |
| 4.2 | Reconciliación probada con el bot real y una posición abierta | **No medida** (ISS-017) |
| 4.3 | El journal verificado contra una sesión real | **ROJA** — 0 bytes (ISS-011) |
| 4.4 | El panel no puede desarmar los validadores | **ROJA** (ISS-003) |
| 4.5 | CI en verde en cada commit | **ROJA** (ISS-012) |
| 4.6 | Alertas que avisan sin mirar el panel | **ROJA** (ISS-023) |
| 4.7 | Copia de seguridad de `data/` y de los resultados | **Sin declarar** (ISS-016) |
| 4.8 | Una sesión completa del bot, en simulación, sin incidentes | **No medida** |

**Condición adicional, propia de este sistema:** la Etapa 4 exige además una
decisión explícita del propietario, porque activarla implica poner
`live.enabled: true` en un árbol que hoy está preparado para que eso sea difícil.

---

## Condiciones que no dependen de una etapa

Estas aplican en todo momento y no se negocian:

| # | Condición | Por qué | Estado |
|---|---|---|---|
| G.1 | El panel no expone ninguna ruta de orden | D-015 | **VERDE** — 84 rutas, 0 de orden |
| G.2 | El panel no importa el bot | D-017 | **VERDE** — 0 aristas |
| G.3 | El panel no escribe el heartbeat del bot | D-017 | **VERDE** |
| G.4 | El heartbeat rancio se lee como "no corriendo" | — | **VERDE** |
| G.5 | El panel enlaza a `127.0.0.1` | — | **VERDE** |
| G.6 | Ningún `.env` versionado | — | **VERDE** |
| G.7 | Una regla se opera solo si el registro la confirma | — | **VERDE** |
| G.8 | Un tramo usado para elegir no vuelve a usarse para elegir | Regla 8 | **VERDE** |
| G.9 | Las once reglas del catálogo son once preguntas distintas | — | **VERDE** — `test_una_voz_por_pregunta.py` |
| G.10 | El panel no puede desactivar un validador sin acción deliberada | encargo del dueño | **ROJA** (ISS-003) |

---

## Cómo se registra una puerta

Cuando una puerta cambia de estado, en el mismo commit:

1. `EVIDENCE.yaml` → el comando que lo midió y su resultado.
2. `COMPONENTS.yaml` → el componente afectado, con su nuevo estado.
3. `ISSUES.yaml` → el problema, con `estado: cerrado` y la evidencia del cierre.
4. Este fichero → la fila, de ROJA a VERDE.

Una puerta no se marca verde **antes** de tener el comando que la mide. Un
"listo" sin comando es una intención, y este mapa no registra intenciones.