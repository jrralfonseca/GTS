# Cronología de las ventanas y protocolo temporal propuesto

**Fecha:** 2026-10-12
**Origen:** pregunta del propietario tras revisar `INFORME_METODOLOGIA.md`.
**Estado:** análisis completo. **No** se ha cambiado ninguna fecha del registro,
ni ningún estado de metodología, ni el planificador.

---

## 1. La pregunta

> La ventana de confirmación `examinar-2` abarca del 16 de mayo al 16 de noviembre
> de 2023, mientras que la selección `entrenar-2` abarca del 16 de noviembre de
> 2023 al 20 de agosto de 2024. La confirmación es anterior a la selección en el
> tiempo. Para demostrar una evaluación fuera de muestra de tipo temporal
> prospectivo, la selección debe basarse en datos anteriores y la confirmación
> debe utilizar datos posteriores.

**El propietario tiene razón en el fondo.** Y hay un matiz que decide lo que hay
que hacer, así que va antes que el veredicto.

---

## 2. Lo que se auditó

No los nombres: el código que decide y el registro que lo registra.

| Qué | Dónde | Resultado |
|---|---|---|
| Quién fija el orden | `research/metodologia.py:190` `plan_ventanas()` | Confirmación **siempre** antes |
| Por qué | `research/metodologia.py:193-195` | Razonado y escrito |
| Cómo se nombra en el panel | `research/experimento.py:71` | «debe estar ANTES de la seleccion» |
| Registro real, los dos pares | `registro.json` | 4 de 4 pares confirm/sel son retroactivos |
| Solapamiento entre ventanas | registro real | Ninguno |
| Margen entre ventanas | registro real | **Ninguno: son contiguas** |
| Purga / embargo | `ingenieria_v1.yaml`, `universal_v1.yaml` | `purged_cv.activo: false` |
| Lectura cruzada entre ventanas | `experimento.py::_cargar_tramo` | Ninguna: cada una recorta su CSV |

### El registro completo, en orden de tiempo

```
EURUSD H1   grabación 2018-09-24 -> 2026-10-12
  entrenar-2  selección    2018-09-24 -> 2020-01-01   limpia
  examinar-1  confirmación 2020-01-01 -> 2024-01-01   limpia
  entrenar-1  selección    2024-06-01 -> 2026-10-01   gastada

XAUUSD M15  grabación 2023-05-16 -> 2026-10-12
  examinar-2  confirmación 2023-05-16 -> 2023-11-16   gastada   <- la de la corrida
  entrenar-2  selección    2023-11-16 -> 2024-08-20   gastada   <- la de la corrida
  examinar-1  confirmación 2024-08-20 -> 2025-06-29   limpia
  entrenar-1  selección    2025-06-29 -> 2026-10-12   limpia
```

No hay ni un solo par confirm/sel donde la confirmación vaya después.

---

## 3. Qué es esto, exactamente

Es una separación **retrospectiva**, no prospectiva.

**Lo que sí es.** Fuera de muestra dentro del experimento. El barrido lee el
CSV de la ventana de selección y solo ese; la confirmación lee el suyo. La
elección no pudo verse influida por los datos de confirmación, y eso es la
garantía que da el sistema hoy. Es real y no estrivial.

**Lo que no es.** Prospectivo. La confirmación ya había ocurrido cuando se
eligieron los parámetros. Decir «validado fuera de muestra» sin ese matiz
cuenta una cosa por otra: no dice nada sobre lo que pasó, dice que el
parámetro no era una obra de ajuste al pasado.

**Por qué importa el nombre.** Quien lea `confirmacion` en un criterio y
`examinar` en el panel no puede distinguir retrospectivo de prospectivo. Hoy
mismo, el informe de la corrida podría decir «confirmado fuera de muestra» y
no estar mintiendo, pero tampoco estar diciendo lo que el lector va a
entender. **Esa es la laguna concreta, y es de rotulación, no de lógica.**

---

## 4. Lo que además está mal, y no lo había detectado

**Las ventanas son contiguas, sin una sola vela de margen.** En ambos pares
la selección empieza el día que acaba la confirmación. Encima,
`confirmacion.recortar_fechas` recorta exactamente `[inicio, fin)`, sin
historia delante, y las features se calculan con `lookback_bars` velas hacia
atrás. Consecuencias:

1. Las primeras `lookback_bars` de cada ventana no tienen historia propia.
2. No hay ninguna protección contra que una decisión del final de una ventana
   llegue a la primera de la siguiente.
3. `purged_cv` está apagado en los dos perfiles, así que tampoco lo cubre el
   CV interno.

**Ninguna ventana consumida se reutiliza como holdout.** Confirmado: el
preflight rechaza cualquier rango que pise un tramo declarado, y las
`gastada` no se vuelven a abrir. Esto está bien y no se toca.

---

## 5. Protocolo propuesto

### 5.1 Lo que hace falta para que «prospectivo» sea verdad

Tres cosas, y las tres hacen falta:

**(a) Un segundo modo de division, declarado, no por defecto.**
`ventanas.modo: retrospectiva | prospectiva`. En `prospectiva`:
```
  seleccion  ->  embargo  ->  confirmacion
  T0 .............. T1 ...... T2 ... T3
```
El embargo no es decorativo: es lo que impide que una decisión del final de la
selección llegue al principio de la confirmación. Tamaño mínimo: la ventana de
purga de `TrainingConfig` (`purge_pct = 0.1`) o el `lookback_bars` de las
features, lo que sea mayor.

**(b) Que el criterio diga cuál de los dos es.**
Un campo `ventanas.modo` en el criterio y en el documento generado, escrito en
frases: *«confirmación posterior a la selección: evaluación prospectiva»* o
*«confirmación anterior: evaluación retrospectiva, no prospectiva»*. Sin esto,
el campo de arriba es un adorno.

**(c) Que la metodología pueda declararse `aprobada` con esto dentro.**
`plan_ventanas()` tiene hoy la invariante escrita en el docstring
(«sin excepción configurable»). Hacerla configurable es un cambio de
protocolo, y por eso no lo he hecho: es exactamente lo que el propietario dijo
que no se toque sin respaldo.

### 5.2 Qué datos hay, de verdad

```
XAUUSD M15   80 000 velas   2023-05-16 -> 2026-10-12
             rangos libres: NINGUNO

EURUSD H1    50 000 velas   2018-09-24 -> 2026-10-12
             rangos libres: [2024-01-01 -> 2024-06-01)  2 613 velas  152 días
                           [2026-10-01 -> 2026-10-12)    168 velas   11 días
```

**XAUUSD M15 no puede empezar nada.** La corrida de ingeniería del 2026-10-12
se llevó el último hueco libre. No es un defecto: es el sistema funcionando, y
la consecuencia que hay que decir es esa.

**EURUSD H1 sí puede**, y el reparto sale asi:

| Trozo | Días | Velas H1 |
|---|---|---|
| Selección | 92 | ~1 568 |
| Embargo | 2 | ~34 |
| Confirmación | 58 | ~1 011 |
| **Total** | **152** | **2 613** |

Cada ventana por encima de las 501 velas mínimas del perfil. Encaja.

### 5.3 La limitación que hay que decir antes de nada

**Ese hueco es de 2024. Dividirlo prospectivamente sigue siendo retrospectivo
en términos absolutos.** Es prospectivo *relativo al experimento* — la
confirmación no intervino en la selección — que es la garantía que importa
para no autoengañarse, pero no es una prueba de que la estrategia vaya a
funcionar hacia delante.

Para una prueba **prospectiva de verdad** hay que esperar:

1. Declarar la ventana de selección y dejarla correr.
2. **Cerrar el sistema.** Congelar los parámetros en el criterio.
3. Esperar a que la ventana de confirmación termine en el tiempo real.
4. Solo entonces confirmar.

Hoy el sistema hace las cuatro cosas en un mismo trabajo, en 531 segundos. Eso
no puede ser una prueba prospectiva, porque no hubo ninguna espera. **Es la
diferencia de fondo, y no se arregla con fechas: hay que partir el trabajo en
dos y dejar que el tiempo pase.**

---

## 6. Qué falta para poder hacerlo

| Necesario | Por qué | Coste |
|---|---|---|
| Historia nueva de XAUUSD M15 | no queda nada libre | una descarga de MT5 |
| Historia nueva de EURUSD H1 | el hueco libre es de 2024 | una descarga de MT5 |
| `ventanas.modo` en el perfil | el planificador no puede hacer las dos cosas | pequeño |
| Embargo real entre ventanas | hoy son contiguas | pequeño |
| `lookback_bars` de historial en el recorte | hoy las primeras velas de cada ventana están ciegas | pequeño |
| Partir el trabajo en dos con espera | sin esto no hay prospectividad | grande, es diseño |
| Evidencia fuera de muestra que autorice un perfil | sin esto el laboratorio sigue cerrado | investigación, no software |

**El último es el que manda.** Aunque las cinco cosas técnicas se hicieran
mañana, el laboratorio normal seguiría negándose a investigar, porque ninguna
metodología está aprobada —y aprobar una requiere evidencia, no una función
nueva.

---

## 7. Lo que no se ha hecho

- **No se ha cambiado ninguna fecha** del registro.
- **No se ha cambiado el planificador.** La invariante «confirmación antes»
  sigue siendo la que hay, y ahora hay una prueba que la fija.
- **No se ha marcado ninguna metodología como `aprobada`.**
- **No se ha reutilizado ninguna ventana consumida.**
- **La corrida del 2026-10-12 sigue identificada como ingeniería, veredicto
  `inconcluso`, 11 y 9 operaciones.** No es validación de nada.

---

## 8. Pruebas

`tests/test_cronologia_ventanas.py` — 8, todas verdes.

Fijan lo que no puede romperse (solapamiento, cronología de la convención,
tipos en el criterio) y dejan escrito el hueco: la ausencia de embargo es una
**limitación declarada**, y el test falla el día que se añada, para que el
cambio sea visible.