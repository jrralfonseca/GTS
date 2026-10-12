# Fase 1 — Auditoría de datos y ventanas utilizables

**Fecha:** 2026-10-12
**Estado:** fase completada. **Ninguna ventana se declaró, se gastó ni se midió.**
**Ninguna metodología cambió de estado.** El laboratorio sigue cerrado al uso normal.

---

## 0. Resumen para quien decide

Se auditaron las **tres** combinaciones de activo y temporalidad que existen en
`data/raw/`: EURUSD H1, XAUUSD M15 y **XAUUSD M30, que no estaba registrada**.
Se descargó historial adicional (autorizado por el plan) porque **no existía
ni una vela utilizable sin historia previa de observación**. Tras la
descarga, existe exactamente **un** bloque de datos nunca observado:

> **EURUSD H1, 2010-09-01 → 2018-09-23: 49.977 velas, 2.945 días.**

Ese bloque es el **primer (y único) conjunto elegible**. Se propone partirlo
en selección (20.000 velas) → embargo (500) → confirmación (15.000) → holdout
(14.477). La propuesta es provisional: la Fase 2 congela las cifras en el
perfil candidato `universal_v2` antes de declarar nada.

Lo que más importa de la auditoría no son los números sino dos hallazgos:

1. **XAUUSD no tiene nada limpio en ninguna temporalidad**: los experimentos
   documentados de la era H4 (2018-04-05 → 2026-10) ya observaron ese
   calendario completo, y todo lo que el broker puede servir de XAUUSD cae
   dentro.
2. **El registro miente por omisión en EURUSD H1**: sus tramos `examinar-1` y
   `entrenar-2` están marcados `limpia`, pero el pipeline de ML los leyó
   enteros el 2026-10-12 (job `entrenamiento-1p-40e`, 2/40 épocas). El
   registro solo anota lecturas del laboratorio de reglas. Bajo la regla
   estricta de observación, esos tramos **no son muestras vírgenes**.

**La Fase 2 puede empezar** (existe conjunto elegible). Lo que sigue
bloqueado: barridos, declaraciones de tramos, criterios, y el uso normal del
laboratorio.

---

## 1. El inventario, combinación por combinación

Cobertura verificada sobre los ficheros reales tras la descarga adicional
(`research/data_audit.py`, artefacto: `data/research/auditoria-datos-20261012-post.json`).

| | EURUSD H1 | XAUUSD M15 | XAUUSD M30 |
|---|---|---|---|
| Símbolo canónico → nativo en broker | EURUSD → EURUSD | XAUUSD → XAUUSD | XAUUSD → XAUUSD |
| Broker de proveniencia | metaquotes-demo | metaquotes-demo | metaquotes-demo |
| Cobertura | 2010-09-01 19:00 → 2026-10-12 23:00 | 2022-07-11 19:00 → 2026-10-12 22:30 | 2018-04-13 10:30 → 2026-10-12 22:00 |
| Años | 16,11 | 4,25 | 8,50 |
| Velas | 99.998 | 99.998 | 99.998 |
| Duplicados / hacia atrás | 0 / 0 | 0 / 0 | 0 / 0 |
| Fines de semana (cortes normales) | 841 (52,2/año ✓) | 222 (52,2/año ✓) | 444 (52,2/año ✓) |
| Cierres diarios (corte normal del metal) | 0 (forex no cierra intra-semana) | 790 | 1.610 |
| Huecos reales | 29 | 105 | 151 |
| Hueco máximo | 2.280 min (Navidad) | 1.710 min (Navidad) | 2.040 min (Navidad) |
| Spread del feed: mediana / % cero | 2,0 / 28,2% | 7,0 / 3,3% | 6,0 / 5,8% |
| Suelo de coste (evaluación) | 0,9 pips | 3,2 pips | 3,2 pips |
| Swap long/short $/lote/noche | −0,70 / −1,00 | −12,60 / −4,60 | −12,60 / −4,60 |
| Ensayos declarados del par | 80 | 5 | 0 |

Lectura de la integridad:

- **Los "huecos reales" son feriados publicados**, no datos perdidos: los
  mayores son Nochebuena→San Esteban y Nochevieja→Año Nuevo en los tres
  ficheros, y el 4 de julio en XAUUSD. EURUSD, que no cierra intra-semana,
  acumula 29 en 16 años (≈1,8/año): coherente.
- **El feed de demo es optimista y no se usa para costear** (lección de
  `docs/costos_unificados.md`): el 28% de las velas EURUSD reporta spread
  cero. La evaluación cobra `max(spread, suelo)` y el suelo manda siempre.
- **La columna de spread de XAUUSD M30 no vale para nada**: en el fichero
  descargado hoy a las 06:07 tenía spread 0 en la mayoría de velas. Para
  costear no importa (manda el suelo), pero como "dato de ejecución" es
  basura, y así queda dicho.
- `trade_stops_level=0` en evaluación offline (replay): la restricción de
  distancia mínima de parada del broker no se aplica. Es la dirección
  permisiva y está declarada en `research/replay.py`.

### La descarga adicional: qué, cómo y qué no tocó

El plan autoriza descargar historial cuando el inventario no alcanza. Antes
de la descarga, la auditoría (`auditoria-datos-20261012-pre.json`) mostró
**cero velas nunca observadas en los tres pares**. Sin descarga, la Fase 1
terminaba en "no existe conjunto elegible".

| Paso | Mecanismo | Resultado |
|---|---|---|
| Copia de seguridad | copia a `data/raw/backup-pre-descarga-20261012/` | 3 CSV preservados con su contenido exacto |
| Descarga | `scripts/download_data.py --bars 99998 --broker metaquotes-demo` (el mecanismo existente) | EURUSD H1: 50.000 → 99.998 velas (2010-09-01); XAUUSD M15: 80.000 → 99.998 (2022-07-11); XAUUSD M30: 50.000 → 99.998 (2018-04-13) |
| Re-registro | `python -m research.experimento registrar` (la puerta del registro) | las tres grabaciones descritas de nuevo; XAUUSD M30 registrada **por primera vez** |
| Verificación de revisiones | comparación bar a bar contra la copia | solapes idénticos salvo **1 vela por fichero**: la fronteriza (EURUSD 2026-10-12 02:00, Δ1,5 pips; XAUUSD 2026-10-12 17:30, Δ0,63). Los pines (`data/research/tramos/`) preservan lo medido exacto, así que ningún histórico cambia |

Límite técnico documentado: el terminal sirve como máximo **99.998 velas por
petición** (99.999 falla). EURUSD H1 llega a 2010-09-01; el servidor tiene
unas ~300 velas más (hasta 2010-08-19, comprobado con `copy_rates_from`) que
quedan fuera del alcance de una petición. No cambian ningún diseño de
ventana; si algún día importan, hay que pedirlas por rangos.

Las cabeceras nuevas **ya declaran broker** (`# broker: metaquotes-demo`),
lo que corrige la denuncia de la auditoría previa: la proveniencia viaja
ahora con cada fichero.

**Lo que la descarga NO hizo:** no declaró tramos, no gastó ventanas, no
midio ninguna regla, no escribió criterios. Descargar es adquirir; observar
es medir. Aquí solo se adquirió.

---

## 2. Ventanas declaradas, gastadas y consumidas

### 2.1 Lo que dice el registro (`data/research/registro-experimentos.json`)

```
EURUSD H1   grabación 99.998 velas  2010-09-01 -> 2026-10-12   (ensayos: 80)
  entrenar-2  selección    2018-09-24 -> 2020-01-01   limpia
  examinar-1  confirmación 2020-01-01 -> 2024-01-01   limpia
  entrenar-1  selección    2024-06-01 -> 2026-10-01   GASTADA (barrido de 80 configs detenido a mano; sin JSON, sin resultados)

XAUUSD M15  grabación 99.998 velas  2022-07-11 -> 2026-10-12   (ensayos: 5)
  examinar-2  confirmación 2023-05-16 -> 2023-11-16   GASTADA (con resultados: inconcluso, 11 y 9 ops)
  entrenar-2  selección    2023-11-16 -> 2024-08-20   GASTADA (barrido de 5: 0/5 sobreviven B)
  examinar-1  confirmación 2024-08-20 -> 2025-06-29   limpia (declarada, nunca leída por reglas)
  entrenar-1  selección    2025-06-29 -> 2026-10-12   limpia (declarada, nunca leída por reglas)

XAUUSD M30  grabación 99.998 velas  2018-04-13 -> 2026-10-12   (ensayos: 0; sin criterio, sin tramos)
```

### 2.2 El mapa de observación: lo que el registro NO ve

El registro anota lecturas del laboratorio de reglas. Pero "consumido" no
es solo lo que pasó por esa puerta. Este es el mapa completo de periodos
cuyo **resultado o cuyo contenido** ya fue leído por algún experimento
documentado, con la fuente de cada uno (vive en código, con tests, en
`research/data_audit.py:OBSERVADOS_DOCUMENTADOS`):

| Subyacente | Periodo observado | Fuente |
|---|---|---|
| XAUUSD | 2018-04-05 → 2024-02-02 | `research/registro.py` (docstring): tramo de confirmación de la era H4, gastado |
| XAUUSD | 2024-04-23 → 2026-10-12 | `research/registro.py` + `research/window.py` SPLIT=2024-04-23: selección H4 gastada |
| XAUUSD | 2023-05-16 → 2024-08-20 | registro: examinar-2 y entrenar-2 de M15 gastados el 2026-10-12 con resultados |
| EURUSD | 2018-09-24 → 2026-10-12 | job `job-20261012-064349-936caa`: entrenamiento ML (2/40 épocas, `symbols=[EURUSD]`, lookback 100) leyó `EURUSD_H1.csv` entero el 2026-10-12 14:37→14:46 |
| EURUSD | 2024-10-01 → 2026-10-07 | `docs/experimentos/EURUSD-M15.md`: ATR_Breakout refutada (0/25, 1.650 ops) sobre EURUSD M15 el 2026-10-08 |

**Unión:** XAUUSD está observado de 2018-04-05 a 2026-10-12 sin huecos
relevantes. EURUSD está observado de 2018-09-24 a 2026-10-12.

Dos consecuencias que el registro no refleja:

1. **EURUSD H1 `examinar-1` y `entrenar-2` no son vírgenes.** El pipeline ML
   las leyó completas el 2026-10-12. Que el entrenamiento se interrumpiera a
   las 2 épocas no borra la lectura. Para la cualificación no se usan.
2. **Todo XAUUSD alcanzable está observado.** El broker solo sirve XAUUSD
   desde 2018-04 (M30) y 2022-07 (M15), y la observación empieza 2018-04-05.

**Decisión metodológica adoptada (a congelar en Fase 2):** un periodo
calendario cuenta como observado si algún experimento documentado lo leyó,
en **cualquier** temporalidad del mismo subyacente. Seleccionar en M30 sobre
años cuyo resultado H4 ya se vio es una segunda mirada a los mismos
movimientos con otra lupa. La alternativa laxa (consumo por par, como el
registro) declararía "limpias" 100.000 velas de XAUUSD M30; no se adopta
porque la cualificación es justamente el sitio donde la regla estricta
demuestra algo. Si la Fase 2 quiere relajarla, que lo escriba en el perfil
con el motivo, no por defecto.

---

## 3. Cobertura útil restante

| Par | Libre (sin declarar) | Nunca observado | Para selección/confirmación/holdout |
|---|---|---|---|
| EURUSD H1 | 2010-09-01 → 2018-09-23 (2.945 días, 49.977 velas); 2024-01-01 → 2024-05-31 (152 d, 2.613 v); 2026-10-01 → 2026-10-12 (12 d, 192 v) | **2010-09-01 → 2018-09-23: 49.977 velas** | **suficiente** (ver §4) |
| XAUUSD M15 | 2022-07-11 → 2023-05-15 (309 d, 19.931 v); 2026-10-12 (87 v) | nada | insuficiente: 0 velas |
| XAUUSD M30 | todo el rango (3.105 d, 99.998 v) | nada | insuficiente: 0 velas |

Los huecos libres de EURUSD H1 dentro del periodo observado (2.613 y 192
velas) no sirven aunque se ignorara la observación: 2.613 velas repartidas
entre selección, embargo, confirmación y holdout dejan ~650 por ventana
—por encima de las 501 mínimas y muy por debajo de las ~10.000 que hacen
falta para 30 operaciones en la mitad medida.

---

## 4. Suficiencia preliminar de datos y operaciones

El universo a evaluar es el de `universal_v1`: **11 reglas, 175
configuraciones**. Cada cifra de abajo lleva su origen.

**Requisitos registrados** (únicos que existen hoy):

- ≥ 501 velas por ventana (`universal_v1.ventanas.min_velas_ventana`).
- ≥ 30 operaciones para que un |t| signifique algo
  (`universal_v1.min_operaciones`). *Valor convencional, no validado por
  estrategia; la Fase 2 lo re-escribe con su origen.*

**Frecuencia de operaciones medida (solo en ventanas ya gastadas):**

| Evidencia | Resultado |
|---|---|
| XAUUSD M15 `entrenar-2`, mitad B (8.987 velas), barrido del 2026-10-12 (`barrido-XAUUSD-M15-entrenar-2.json`) | ATR_Breakout(14, 1,5): 91 ops; ATR_Breakout(7, 0,75): 332; RSIDivergence(14): 107; RSIDivergence(7): 163; RSIDivergence(28): 3 |
| EURUSD M15 2026-10-08 (`docs/experimentos/EURUSD-M15.md`) | 1.650 ops entre 25 configs de ATR_Breakout en 12 meses M15 ≈ 66 ops/config-año |

**Supuesto de extrapolación a H1 (declarado como supuesto, sin medir):** la
misma regla con los mismos parámetros dispara, por año calendario, igual o
menos en H1 que en M15 (barras más gruesas → menos eventos de barrera por
unidad de tiempo). Se toma la mitad: **~33 ops/config-año en H1**. Ninguna
evidencia directa H1 existe en el repositorio (el experimento AUDUSD H1 dejó
criterio pero no resultados).

**Aritmética sobre el bloque limpio:**

- Selección propuesta: 20.000 velas → mitad medida B ≈ 10.000 velas ≈ 1,6
  años → **~53 ops por config de frecuencia media** (≥30 con margen 1,8×).
  Configs lentas (como RSIDivergence(28): 3 ops en 9.000 velas M15)
  quedarán por debajo de 30 y su veredicto será *inconcluso por
  insuficiencia*: un resultado legítimo, registrado, que cuenta en el
  universo de la corrección por comparaciones múltiples.
- Confirmación: 15.000 velas ≈ 2,4 años ≈ ~80 ops config media.
- Holdout: 14.477 velas ≈ 2,3 años.
- Todas ≫ 501 velas.

**Calidad interna del bloque** (verificada sobre el índice real):
~6.150–6.230 velas por año completo, sin años delgados; 0 velas con volumen
cero; los 428 cortes >2 h son los 416 findes de semana del bloque más los
feriados. No hay tramos muertos ni sospechosos.

**Veredicto de suficiencia: el bloque califica.** Con el supuesto M15→H1
explícito; la Fase 3 (controles) puede medir la frecuencia real en
sintéticos y la Fase 4 la mide de verdad en preflight.

---

## 5. Selección automática del primer conjunto elegible

**Regla, declarada antes de aplicarse:** entre los pares con bloque nunca
observado que (a) permita ≥3 ventanas de ≥501 velas más embargo ≥500 velas,
(b) tenga el símbolo costeado en el perfil de broker declarado, (c) traiga
columnas de ejecución completas, se elige el de **más velas nunca
observadas**; empate por orden alfabético. Si ninguno califica: motivo
concreto y parada antes de consumir ventanas.

| Candidato | ¿Califica? | Motivo concreto |
|---|---|---|
| **EURUSD H1** (2010-09-01 → 2018-09-23) | **SÍ — seleccionado** | 49.977 velas vírgenes; costeado (suelo 0,9 pips, swaps declarados); OHLC+volumen+spread completos; aloja 3 ventanas ≫501 + embargo ≥500; ~53 ops/config media en la mitad B |
| XAUUSD M15 | NO | 0 velas nunca observadas: 309 días libres caen dentro del calendario observado de XAUUSD (era H4); sus tramos limpios declarados (2024-08→2026-10) también |
| XAUUSD M30 | NO | 0 velas nunca observadas: el broker no sirve XAUUSD anterior a 2018-04-13 y la observación empieza 2018-04-05 |
| EURUSD H1, hueco 2024 | NO | observado (ML + barrido gastado); y 2.613 velas no alojan 3 ventanas con ≥30 ops |

---

## 6. Propuesta de ventanas candidatas (provisional — congela la Fase 2)

Sobre el bloque limpio de EURUSD H1, corte por **velas contadas** (no por
fechas redondas), orden cronológico prospectivo exigido por el plan:

| Ventana | Rango | Velas | Días | Papel |
|---|---|---:|---:|---|
| selección | 2010-09-01 → 2013-11-21 | 20.000 | 1.175 | elegir; el barrido mide su mitad B (~10.000 velas) |
| embargo | 2013-11-21 → 2013-12-20 | 500 | 28 | ni se mide ni se elige; separa decisiones de la selección del arranque de la confirmación |
| confirmación | 2013-12-20 → 2016-05-27 | 15.000 | 889 | evaluar las candidatas congeladas, cronológicamente posterior |
| holdout | 2016-05-27 → 2018-09-22 | 14.477 | 847 | intacto hasta después de la confirmación |

**Origen de cada número** (el plan exige que ninguno sea arbitrario):

- **Embargo = 500 velas**: `research/window.py:HISTORY_BARS` — cubre el
  warm-up más ancho del universo (MA lenta 200; Ichimoku base+desplazamiento
  104) con margen, y excede con mucho cualquier horizonte de posición
  abierta documentado en H1 ("holds de varios días", `docs/costos_unificados.md`).
  Las 500 velas del embargo pueden servir de historia de warm-up para la
  confirmación: es para lo que existe un embargo. La Fase 3 debe demostrarlo
  con una prueba de filtración diseñada.
- **Selección = 20.000 velas (40%)**: el doble del mínimo que la aritmética
  del §4 pide para ≥30 ops en la mitad B. Margen 1,8×–2×.
- **Confirmación y holdout ≈ 15.000 (30%) cada una**: cada una debe poder
  sostener un veredicto independiente por sí misma.
- **Cronología**: selección → embargo → confirmación → holdout, estrictamente
  en el tiempo. La confirmación no interviene en la selección; el holdout no
  se abre hasta después de la confirmación (plan, Fase 5).

**Limitaciones declaradas de esta propuesta** (no se esconden):

1. **Regímenes distintos por ventana.** La selección cubre la post-crisis
   europea (2010-13); la confirmación, el derrumbe del EUR de 2014-15 y el
   rango posterior; el holdout, la normalización y el rally de 2017.
   Evaluar prospectivamente contra un régimen distinto es lo que debe hacer
   una confirmación — pero hay que decirlo antes de verlo.
2. **El 15-01-2015 (fin del suelo del franco suizo) cae dentro de la
   confirmación.** Un evento extremo único: una sola operación puede dominar
   un |t|. La Fase 2 decide cómo tratar outliers ANTES de ver resultados
   (bootstrap por bloques ya mitiga; lo demás es diseño del perfil).
3. **La selección empieza en el borde de la grabación**: no hay historia
   previa a 2010-09-01, así que sus primeras ~500 velas son warm-up ciego
   para las reglas anchas. Está contado dentro de las 20.000; no amenaza la
   suficiencia.
4. **Spread del feed inútil, stops no aplicados, slippage supuesto** (§1): el
   coste de evaluación es el suelo declarado, que es el conservador.

---

## 7. Archivos producidos, experimentos ejecutados, fallos

### Archivos

| Fichero | Qué es |
|---|---|
| `research/data_audit.py` | la herramienta de auditoría: solo lectura, sin MT5, sin escrituras al registro; lleva el mapa de observación documentado (con fuente por entrada) en código, con la valla de que no puede estar vacío |
| `tests/test_data_audit.py` | 15 pruebas: clasificador de huecos (fin de semana con reapertura temprana, cierre diario del oro que cruza medianoche, forex sin cierre diario, hueco real), rangos libres y nunca observados (bordes exactos), denuncias fichero↔registro, inventario completo contra un mundo pequeño |
| `data/research/auditoria-datos-20261012-pre.json` / `-post.json` | el inventario antes y después de la descarga (máquina-local, como el registro) |
| `data/raw/backup-pre-descarga-20261012/` | los tres CSV exactos que el registro describía antes de re-descargar |
| `data/research/registro-experimentos.json` | re-registradas las tres grabaciones (por la puerta `registrar`); XAUUSD M30 registrada por primera vez |
| este informe | el entregable de la fase |

### Experimentos ejecutados en Fase 1 (ninguno midió reglas)

Auditorías pre/post; sondeo de profundidad del terminal (solo lectura);
descarga adicional autorizada por el plan; comparación bar a bar contra la
copia (revisiones); análisis de calidad del bloque limpio; pruebas de
software. **No se ejecutó ningún barrido, ninguna medición de reglas sobre
ventana limpia, ninguna declaración de tramos, ningún criterio.**

### Fallos encontrados y corregidos durante la fase

1. **Bug real cazado por los tests antes de que doliera:**
   `rangos_no_observados` comparaba `str` con `date` en su filtro final. La
   corrida real no lo mostró porque hoy todos los pares devuelven lista
   vacía — justo el camino que la Fase 1 necesita. Un test con rangos no
   vacíos lo destapó.
2. **Clasificador de fines de semana frágil** (arreglado): una reapertura
   dominical temprana habría convertido el fin de semana del mercado en
   "hueco real". Regla nueva: contiene el inicio del sábado.
3. **Cierres diarios mal calibrados:** el oro cierra cruzando la medianoche
   UTC en este broker (última vela 23:45 → 01:00), y forex no cierra
   intra-semana. El clasificador es ahora consciente del símbolo: lo que en
   XAUUSD es cierre del mercado, en EURUSD es hueco real del feed.
4. **Hallazgos de datos:** XAUUSD M30 existía sin registrar; las cabeceras
   no declaraban broker (la re-descarga lo corrige); una vela fronteriza por
   fichero fue revisada por el broker entre descargas (los pines preservan
   lo medido exacto).

### Pruebas

`tests/test_data_audit.py`: 15, verdes. Vallas existentes re-verificadas:
`test_research_cannot_trade.py` (el paquete research sigue sin poder operar,
el módulo nuevo no toca MT5), `test_registro.py`, `test_cronologia_ventanas.py` —
todas verdes.

---

## 8. Por qué se permite avanzar y qué sigue bloqueado

**Se permite la Fase 2** porque la Fase 1 cumplió su entregable con
evidencia: existe un inventario auditado, existe un conjunto elegible
(EURUSD H1, 2010-09-01 → 2018-09-23) con ventana candidatas dimensionadas
con origen por número, y la suficiencia preliminar es positiva con supuestos
declarados.

**Sigue bloqueado**, y por qué:

- **El uso normal del laboratorio**: ninguna metodología está aprobada y
  nada aquí cambia eso (regla de gobierno; `QUALIFICACION_METODOLOGIAS.md`).
- **Declarar tramos de las ventanas propuestas**: la declaración es
  preinscripción y pertenece al momento en que el perfil `universal_v2`
  esté congelado. Declararlas ahora sería comprometer velas antes de tener
  criterio.
- **Barridos y mediciones**: Fase 4, tras los controles de Fase 3.
- **Reutilizar el bloque 2018-09-24 en adelante de EURUSD H1, o cualquier
  XAUUSD, como muestra virgen**: observados (§2.2). Gastados siguen
  gastados.

**La Fase 2 debe resolver** (herencias directas de esta auditoría):
congelar las cifras del §6 en `universal_v2` (con `estado: implementada`,
nunca `aprobada`); registrar el tratamiento de lecturas no-laboratorio (el
hallazgo ML del §2.2) en la política de ventanas consumidas; decidir el
tratamiento de outliers (SNB 2015) antes de ver resultados; y fijar si la
regla estricta de observación por subyacente queda como política del perfil.
