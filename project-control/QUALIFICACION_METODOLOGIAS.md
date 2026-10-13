# Procedimiento de cualificación de metodologías

**Fecha:** 2026-10-12
**Estado:** procedimiento definido. **Ninguna metodología está aprobada hoy** y
esto no cambia el estado de ninguna.

---

## 1. Para qué existe este documento

Porque la respuesta a «¿por qué no puedo investigar?» tiene que ser la misma
siempre, y porque un estado que se puede cambiar editando un YAML no es una
garantía: es un campo.

El laboratorio se niega hoy. Esa negativa es lo correcto, y mientras siga siendo
la correcta hay que poder explicar **qué la haría incorrecta** y con qué
evidencia.

---

## 2. Los cuatro estados, y lo que cada uno promete

| Estado | Qué significa | Qué NO promete |
|---|---|---|
| `implementada` | El código existe y se ejecuta. | Nada sobre si el método sirve. |
| `verificada` | Las pruebas demuestran que ejecuta el procedimiento completo, con sus controles, sobre datos de verdad. | Nada sobre rentabilidad ni sobre validez del diseño. |
| `aprobada` | Hay evidencia fuera de muestra, revisada, que autoriza este metodo **en el ambito declarado**. |
| *(no existe)* `validada` | Se reserva para una **estrategia concreta** que haya superado el listón de una metodología aprobada. | — |

La distinción que más importa en la práctica es `implementada` vs `aprobada`.
Todo el sistema de metodología se construyó alrededor de no confundirlas, y
este documento dice cómo se pasa de una a otra.

**Ninguna metodología puede declararse `validada`.** La aprobación es de la
metodología; la validación es de una estrategia concreta, y requiere su
propio expediente.

---

## 3. Requisitos para pasar de `implementada` a `verificada`

Todo lo siguiente, y nada de esto es una excepción:

1. **Prueba de extremo a extremo** que ejercite el procedimiento completo con
   datos reales y costes declarados, no con un maqueto.
2. **Prueba negativa**: manipulado, el procedimiento tiene que negarse y decir
   por qué.
3. **El universo está registrado antes de los resultados.** Escriba el
   criterio antes de la primera cifra, o no vale.
4. **La corrección por pruebas múltiples se calcula sobre el universo
   declarado**, y su tamaño se contrasta con el del barrido.
5. **Los controles que el método declara** se ejecutan y se reportan, aunque
   salgan "inconcluso». Un resultado negativo no es un fallo de verificación.
6. **El informe nombra lo que se investigó**, qué regla pasó qué control, y por
   qué se rechazó cada una.

Lo que **no** hace falta: que el resultado sea positivo. Verificar es
comprobar que el procedimiento dice la verdad, no que diga algo bueno.

---

## 4. Requisitos para pasar de `verificada` a `aprobada`

Aquí es donde el criterio es alto a propósito, y es donde se decide si el
laboratorio se abre.

### 4.1 El expediente de evidencia

Un documento en `project-control/`, con identificador, que contenga:

**a) Ámbito declarado.** En qué par, qué temporalidad, qué horizontes, qué
régimen de costes. Fuera del ámbito, la metodología no está aprobada: está
`implementada` y punto.

**b) Evidencia fuera de muestra, por par y temporalidad.** No vale un resultado
de una corrida de ingeniería. El listón lo fija la metodología, pero la
evidencia la tienen que cumplir al menos:

| Condición | Por qué |
|---|---|
| La confirmación no intervino en la selección | Sin esto no hay nada que validar. |
| La separación es **prospectiva** o, si es retrospectiva, **declarada como tal** | Un holdout de 2024 no dice nada sobre 2026. |
| Hay **embargo** entre selección y confirmación | Sin margen, una decisión del final de una ventana llega al principio de la otra. |
| `|t|` y numero de operaciones alcanzan el liston declarado | El listón por defecto (`|t| >= 2.0`, `>= 30` operaciones) **no es una propiedad demostrada de ninguna estrategia**: es un valor por defecto del módulo. |
| La corrección por pruebas múltiples deja algo en pie | Un resultado que sobrevive solo sin corregir no sobrevive. |
| Costs of broker **declarados**, no por defecto | Un resultado sin costes declarados es un número sin contexto. |
| **Al menos dos pares o dos regímenes** | Un único par es un caso, no un método. |

**c) Registro de lo que se rechazó y por qué.** Un expediente que solo enumera
lo que paso es un informe a medias.

**d) Fecha, quién lo revisó, y qué se puede volver a pedir.**

### 4.2 Las condiciones de rechazo

Un methodology **no** se aprueba, y se dice por qué, cuando:

- La evidencia sale de un perfil de ingeniería. Los resultados de ingeniería no
  cuentan como nada, ni hacia arriba ni hacia abajo.
- La separación de ventanas es retrospectiva **y no está declarada como tal** en
  el criterio. Callar la diferencia no la convierte en prospectiva.
- No hay embargo entre ventanas y el método depende de features con lookback.
- La confirmación se hizo sobre una ventana que ya había sido leída. Un
  holdout gastado no es un holdout.
- El listón se ha movido para que la estrategia pase. Mover el listón es
  cambiar el método, y eso es una metodología nueva con otra versión.
- El único resultado favorable viene de una sola configuración. Una
  configuración afortunada entre 175 no es un método.

### 4.3 Lo que NO es motivo de aprobación

- Un test verde.
- Una corrida que termino sin errores.
- Un |t| alto sin corrección por pruebas múltiples.
- Rentabilidad histórica alta. **Nunca**, en ninguna circunstancia. Ese es el
  espejismo que el diseño entero existe para no confundir.

---

## 5. Cómo se cambia un estado en la práctica

1. Se escribe el expediente (§4.1) en `project-control/`, con su identificador.
2. Se sube la **versión** del perfil en `config/metodologias/`. Una metodología
   corregida es `v2`, no `v1` con el YAML tocado.
3. Se pone `estado: aprobada` y `ambito:` con el alcance exacto.
4. Se escribe en el criterio qué expediente la autoriza, para que un lector de
   2027 pueda seguir el rastro sin preguntar.
5. Se ejecuta `scripts/corrida_integracion.py` una vez contra el ámbito
   declarado, para comprobar que el camino nuevo no se ha roto.

El paso 3 es el único que abre el laboratorio, y es el único que no puede
hacerse sin los pasos 1 y 2.

---

## 6. Cómo se revierte

Bajar un estado es más fácil que subirlo:

1. `estado: verificada` en el YAML.
2. Entrada en el changelog diciendo qué evidencia se withdrawó.
3. El laboratorio se vuelve a negar, con el motivo.

Los criterios ya escritos **no se tocan**: son inmutables y lo seguiran siendo.
Un criterio que dice `aprobada`followe diciendo lo que decía el día que se
escribió, aunque después la metodología se degrade. Eso es lo correcto: el
criterio registra una decisión en su momento, y el changelog registra el
cambio de opinión.

---

## 7. El laboratorio mientras no haya ninguna aprobada

Hoy: se niega, y el mensaje dice qué falta. Eso está bien y es lo que hace.

Lo que **no** debe pasar:

- Que aparezca un interruptor, una casilla o un parámetro de entorno que lo
  abra. El laboratorio no tiene modo avanzado. Ya no lo tuvo.
- Que una metodología pase a `aprobada` para desbloquear la interfaz. Ese
  sería cambiar el listón para que pase, que es §4.2.
- Que las pruebas técnicas se mezclen con los resultados de investigación. Las
  pruebas técnicas siguen usando `ingenieria_v1`, que se marca como tal en el
  criterio, en el informe y en el panel. Sus resultados no cuentan.

---

## 8. Dónde está hoy cada metodología

| Perfil | Estado | Uso | Ámbito |
|---|---|---|---|
| `universal_v1` | `implementada` | `investigacion` | ninguno declarado |
| `ingenieria_v1` | `implementada` | `ingenieria` | ninguno — **no es investigación** |

Ninguna de las dos ha pasado por §4. La segunda no lo intentará nunca: existe
para que el camino se pueda probar, y su `uso: ingenieria` lo impide mezclarla
con investigación por construcción, no por disciplina de quien la usa.

---

## 9. Pruebas

`tests/test_guardia_cientifica.py` (54) y `tests/test_metodologia_criterio.py`
(30) comprueban, entre otras cosas, que:

- `seleccionar()` exige `aprobada` y se niega sin ella.
- Ningún perfil del catálogo está `aprobada`.
- El criterio imprime el estado y el uso de la metodología que lo generó.
- El laboratorio se niega con un motivo que dice qué falta.

`tests/test_cronologia_ventanas.py` (8) añade la condición §4.2 sobre la
separación temporal, de forma verificable.