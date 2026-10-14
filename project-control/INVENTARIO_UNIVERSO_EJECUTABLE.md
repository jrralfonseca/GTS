# Inventario del Universo Ejecutable — Fase 2

**Fecha:** 2026-10-13  
**Estado:** Documentación técnica — no ejecuta barridos, no consume ventanas

---

## Resumen

| Métrica | universal_v1 (perfil) | sweep.py::_grids() (código) | Diferencia |
|---------|----------------------|------------------------------|------------|
| Reglas declaradas | 11 | 4 | 7 faltantes |
| Configuraciones totales | 175 | 80 | 95 faltantes |
| Reglas barribles | 11 | 4 | 7 sin rejilla |

> **Conclusión:** El perfil metodológico declara un universo que el código de barrido no cubre completamente. La Fase 2 debe completar las 7 rejillas faltantes en `research/sweep.py::_grids()` antes de poder ejecutar un barrido coherente con la metodología declarada.

---

## Tabla completa por regla

| # | Regla | Familia | Pregunta (Strategy.question) | Defaults | Rejilla en universal_v1 | Rejilla en sweep.py | Estado |
|---|-------|---------|------------------------------|----------|-------------------------|---------------------|--------|
| 1 | MA_Crossover | trend | "la media reciente esta por encima de la vieja" | fast=20, slow=100 | 25 (5×5) | 25 ✓ | **Completa** |
| 2 | EMA_Trend_Pullback | trend | "un retroceso dentro de una tendencia ya establecida" | ema=20, lookback=20 | 25 (5×5) | 25 ✓ | **Completa** |
| 3 | ATR_Breakout | trend | "el movimiento de esta barra supera su propia volatilidad" | atr=14, breakout=1.5, exit=1.0 | 25 (5×5) | 25 ✓ | **Completa** |
| 4 | RSIDivergence | momentum | "el precio hace un extremo que el momentum no confirma" | rsi=14, pivot=5, gap=5 | 5 (5×1) | 5 ✓ | **Completa** |
| 5 | Donchian_Breakout | trend | "el cerrojo rompió el rango de las barras anteriores" | entry=20, exit=10 | 25 (5×5) | 0 | **FALTA rejilla** |
| 6 | SuperTrend | trend | "el cerrojo cruzó la banda ATR que lo persigue" | atr=10, mult=3.0 | 25 (5×5) | 0 | **FALTA rejilla** |
| 7 | TimeSeriesMomentum | momentum | "el precio está por encima de donde estaba hace unas barras" | lookback=40, buffer=0.0 | 5 (5×1) | 0 | **FALTA rejilla** |
| 8 | HeikinAshi_Trend | trend | "la mayoría de las velas blendas apunta al mismo lado" | trend_bars=5 | 5 (5×1) | 0 | **FALTA rejilla** |
| 9 | ZScore_Reversion | mean_reversion | "el cerrojo se fue de su media más desviaciones de lo normal" | period=20, entry=2.0, exit=0.5 | 5 (5×1) | 0 | **FALTA rejilla** |
| 10 | Ichimoku_Cloud | trend | "el cerrojo cruzó la nube desplazada de hace unas barras" | conv=9, base=26, span=52, disp=26 | 25 (5×5) | 0 | **FALTA rejilla** |
| 11 | ParabolicSAR | trend | "el movimiento lleva el tiempo suficiente como para girar el stop" | step=0.02, max_step=0.2 | 5 (5×1) | 0 | **FALTA rejilla** |

---

## Detalle de rejillas faltantes (para implementar en sweep.py::_grids())

### 5. Donchian_Breakout — 25 configs (5×5)
```python
entry_period = int(default_params("Donchian_Breakout")["entry_period"])  # 20
exit_period = int(default_params("Donchian_Breakout")["exit_period"])    # 10
plans["Donchian_Breakout"] = [
    {"entry_period": e, "exit_period": x}
    for e in _scaled(entry_period) for x in _scaled(exit_period)
]
```

### 6. SuperTrend — 25 configs (5×5)
```python
atr_period = int(default_params("SuperTrend")["atr_period"])      # 10
multiplier = float(default_params("SuperTrend")["multiplier"])    # 3.0
plans["SuperTrend"] = [
    {"atr_period": p, "multiplier": m}
    for p in _scaled(atr_period) for m in _scaled(multiplier)
]
```

### 7. TimeSeriesMomentum — 5 configs (5×1)
```python
lookback = int(default_params("TimeSeriesMomentum")["lookback"])   # 40
plans["TimeSeriesMomentum"] = [
    {"lookback": l} for l in _scaled(lookback)
]
```

### 8. HeikinAshi_Trend — 5 configs (5×1)
```python
trend_bars = int(default_params("HeikinAshi_Trend")["trend_bars"]) # 5
plans["HeikinAshi_Trend"] = [
    {"trend_bars": t} for t in _scaled(trend_bars)
]
```

### 9. ZScore_Reversion — 5 configs (5×1)
```python
period = int(default_params("ZScore_Reversion")["period"])         # 20
plans["ZScore_Reversion"] = [
    {"period": p} for p in _scaled(period)
]
```

### 10. Ichimoku_Cloud — 25 configs (5×5)
```python
base = int(default_params("Ichimoku_Cloud")["base"])               # 26
displacement = int(default_params("Ichimoku_Cloud")["displacement"]) # 26
plans["Ichimoku_Cloud"] = [
    {"base": b, "displacement": d}
    for b in _scaled(base) for d in _scaled(displacement)
]
# Nota: conversion y span se mantienen en defaults (9, 52) — variar 4 ejes serían 125 configs
```

### 11. ParabolicSAR — 5 configs (5×1)
```python
step = float(default_params("ParabolicSAR")["step"])               # 0.02
plans["ParabolicSAR"] = [
    {"step": s} for s in _scaled(step)
]
```

---

## Costes y supuestos de ejecución (de universal_v1 / broker metaquotes-demo)

| Símbolo | contract_size | pip_size | spread_pips (feed) | floor_pips (eval) | swap_long | swap_short | slippage_bps |
|---------|---------------|----------|---------------------|-------------------|-----------|------------|--------------|
| EURUSD | 100,000 | 0.0001 | 0.1 | **0.9** | -0.70 | -1.00 | 0.5 |
| GBPUSD | 100,000 | 0.0001 | 0.2 | **1.2** | -0.20 | -2.20 | 0.5 |
| USDJPY | 100,000 | 0.01 | 0.2 | **1.0** | -0.06 | -0.38 | 0.5 |
| XAUUSD | 100 | 0.10 | 0.2 | **3.2** | -12.60 | -4.60 | 0.5 |
| EURGBP | 100,000 | 0.0001 | 0.3 | **1.5** | -0.30 | -1.80 | 0.5 |

> **Nota:** La evaluación usa `max(spread_pips, floor_pips)` → siempre cobra el suelo (modo conservador). El feed de demo es optimista (EURUSD 28% velas con spread 0).

---

## Métricas de rendimiento y riesgo (delimitadas por el perfil)

| Métrica | Valor en universal_v1 | Origen | Validado |
|---------|----------------------|--------|----------|
| |t| mínimo (umbral_abs_t) | 2.0 | Convención del módulo | **NO** |
| Mínimo operaciones (min_operaciones) | 30 | Convención del módulo | **NO** |
| Max drawdown % | 25.0% | Límite convencional | **NO** |
| Ventanas.min_velas_ventana | 501 | research/window.py:HISTORY_BARS | Código |
| Ventanas.fraccion_confirmacion | 0.40 | 40% para confirmación | Convención |

> **Importante:** Ninguno de estos valores está "científicamente validado". Son convenciones del módulo escritas en el perfil para que sean auditables y versionables. La validación requiere evidencia fuera de muestra (Fase 5+).

---

## Cálculo del estadístico t y suficiencia

```python
# research/reality_check.py:t_statistic()
def t_statistic(returns):
    x = np.asarray(returns, dtype=float)
    n = x.size
    if n < 2: return 0.0
    sd = float(x.std(ddof=1))
    if sd <= 0.0: return 0.0
    return float(x.mean() / (sd / math.sqrt(n)))
```

- **Unidad de muestreo:** operaciones (trades), no tiempo
- **Mínimo para t válido:** 30 operaciones (`DEFAULT_MIN_LENGTH = 30`)
- **Por debajo:** veredicto **inconcluso** (no refutación, no ventaja)

---

## Corrección estadística — White's Reality Check

| Parámetro | Valor | Origen |
|-----------|-------|--------|
| Método | White's Reality Check (2000) | research/reality_check.py |
| Bootstrap | 10,000 remuestreos | DEFAULT_BOOTSTRAPS |
| Semilla | 20261003 (fija) | Código |
| Two-sided | True (corrige selección sobre |t|) | sweep.py |
| Denominador | Newey-West HAC (long-run variance) | Andrews bandwidth |
| Block bootstrap | Opcional, medido por serie | medir_bloque() |

> **Universo de la corrección:** TODAS las configuraciones barridas (175 en universal_v1), no solo las que sobreviven. Corregir sobre los supervivientes sería corregir después de elegir.

---

## Semillas y reproducibilidad

| Semilla | Uso | Dónde se fija |
|---------|-----|---------------|
| 7 | Semilla base del experimento / corrección | universal_v1:correccion_multiple.semilla |
| 20261003 | Semilla del Reality Check | research/reality_check.py:DEFAULT_BOOTSTRAPS seed |
| 7919 | Offset por configuración en bootstrap | research/reality_check.py:_bootstrap_max_statistics |
| null (auto) | Semilla del criterio pre-inscrito | Se elige al escribir el criterio |

---

## Condiciones de veredicto (del pipeline)

| Resultado | Condición | Qué significa |
|-----------|-----------|---------------|
| **aguanta** | Las 3 configs (una por regla elegida) pasan: neto > 0, \|t\| ≥ 2.0, ops ≥ 30 | Ventaja confirmada fuera de muestra |
| **refutada** | Falla la elegida (RSIDivergence) O la del config (EMA_Trend_Pullback) | Sobreajuste O regla de ejecución sin edge |
| **inconcluso** | Alguna config no llega a 30 ops | Insuficiencia de datos — no cuenta ni a favor ni en contra |
| **parcial** | Solo falla la 3ra (Ichimoku/ZScore/Parabolic/TimeSeries/Donchian/SuperTrend/HeikinAshi) | Ventaja es del punto concreto, no de la familia |

> La 3ra config (mejor de su familia) **no decide** el veredicto global. Su fallo dice "la ventaja no es de la familia".

---

## Reglas de versionado para cambios posteriores

| Cambio | Qué requiere | Ejemplo |
|--------|--------------|---------|
| Añadir/quitar regla del universo | Nuevo perfil (universal_v3) | universal_v1 → universal_v2 añade Donchian grid |
| Cambiar rejilla de una regla | Nueva versión del perfil (v2, v3...) | universal_v1 → universal_v2 |
| Cambiar umbral_abs_t / min_operaciones | Nueva versión del perfil | universal_v1 (2.0/30) → universal_v2 (2.5/50) |
| Cambiar ventana (fracción, embargo) | Nueva versión del perfil | universal_v1 (40%/500) → universal_v2 (35%/1000) |
| Cambiar broker / costes | Nuevo perfil O mismo perfil v+1 | metaquotes-demo → exness |
| Corregir bug en sweep/replay | NO cambia perfil; fix en código + test | El perfil congela la metodología, el código la ejecuta |

> **Principio:** Un perfil YAML viaja en git. Cambiar la metodología es cambiar un fichero con su nombre y fecha en el diff, no editar una constante en Python.

---

## Estado actual de implementación (checklist)

- [x] Perfil `universal_v1` escrito y validado (11 reglas, 175 configs)
- [x] Registro de datos y auditoría completados (Fase 1)
- [x] 4/11 rejillas implementadas en `sweep.py::_grids()` (80/175 configs)
- [ ] **7/11 rejillas FALTANTES en sweep.py** (95 configs)
- [ ] Ventanas candidatas para EURUSD H1 definidas (ver §6 del informe Fase 1)
- [ ] Pruebas sintéticas de protocolo (punto 5 del mandato)

---

## Bloqueos para pasar a investigación histórica

1. **Completar las 7 rejillas faltantes en `sweep.py::_grids()`** — sin ellas el barrido no ejecuta el universo declarado.
2. **Congelar `universal_v2`** con las 175 configs implementadas (estado: `implementada`).
3. **Pruebas técnicas de protocolo** (punto 5): sintéticos, controles negativos, detección de filtración, verificación de inmutabilidad de criterio.
4. **Metodología aprobada** — requiere evidencia fuera de muestra (Fase 5+). Hoy **ninguna** metodología está aprobada.