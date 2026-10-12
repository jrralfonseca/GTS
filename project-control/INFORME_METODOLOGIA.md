# GTS — Estado real tras las tres etapas de la arquitectura de metodología

**Fecha:** 2026-10-12
**Motivo de este documento:** el encargo pedía informar por etapa de los
archivos tocados, el motivo, las pruebas y las limitaciones que quedaran. Eso no
cabe en un mensaje de chat sin perder detalle, y un informe que se queda en la
conversación se pierde en cuanto toca scrolls.

Todo lo de aquí está **comprobado contra el código o contra una corrida**. Donde
hay una suposición o un límite sin resolver, está escrito como tal y no como un
logro.

---

## 0. Resumen en una página

| | |
|---|---|
| **Catálogo de metodologías** | `config/metodologias/` — 2 perfiles versionados, validados al cargar |
| **Criterio estructurado** | `data/research/criterios/<id>.json` — inmutable, con SHA-256 del contenido canónico |
| **Documento de auditoría** | `docs/experimentos/<id>.md` — **generado** desde el JSON, nunca editado |
| **Sistema heredado** | intacto: `config/criterios.json`, los 4 `.md` legacy, el CLI, los runners |
| **Metodologías aprobadas hoy** | **ninguna.** El flujo normal del laboratorio se niega y lo dice |
| **Perfil de ingeniería** | `ingenieria_v1`, 2 reglas / 5 configuraciones, marcado como no investigación |
| **Pruevas** | 104 nuevas + suite completa verde en los dos raíces |
| **Corrida real** | XAUUSD M15, el par del trabajo que falló, de extremo a extremo: INCONCLUSO |

**La respuesta corta a «¿puedo ya investigar?» es que no**, y es la respuesta
correcta: no hay metodología aprobada, y aprobar una exige evidencia, no un
cambio de YAML.

---

## 1. El problema que existía

`jobs/runners/pipeline.py` declaraba ventanas y daba por hecho que la promesa y
la grabación ya estaban puestas. En un par nuevo no lo están, así que la puerta
de `research/experimento.py` se negaba:

```
RegistroError: XAUUSD M15 no tiene criterio pre-inscrito.
```

Faltaban **los dos** prerrequisitos a la vez, no uno: `criterio` y `grabacion`
(ambos `null` en el registro para XAUUSD M15).

---

## 2. Etapa 1 — Catálogo, criterio automático y preflight

### Archivos nuevos
| Archivo | Qué es |
|---|---|
| `config/metodologias/universal_v1.yaml` | Metodología general: 11 reglas, 175 configuraciones, listón, corrección, política de ventanas |
| `config/metodologias/ingenieria_v1.yaml` | Perfil de ingeniería: 2 reglas, 5 configuraciones |
| `research/metodologia.py` | Carga y valida el catálogo; **selección estricta**; intersección de universos |
| `research/criterios.py` | Almacén inmutable del criterio + generación del documento |
| `research/preinscripcion.py` | Preflight, elección de metodología, escritura y registro de la promesa |

### Archivos modificados
| Archivo | Motivo |
|---|---|
| `core/criteria.py` | Reescrito. Estaba roto: `build_criterion_from_form` parseaba los valores y devolvía todos por defecto, y no lo importaba nadie |
| `research/registro.py` | `fijar_criterio` acepta ahora `datos` (id, hash, metodología, estado). Opcional: los criterios legacy siguen funcionando |
| `research/experimento.py` | `ejecutar_barrido` acepta rejillas del criterio; la trazabilidad (id + hash del criterio) viaja **dentro** del JSON de salida |
| `jobs/runners/pipeline.py` | Reescrito: preinscribe antes de declarar nada; **no acepta ningún parámetro científico** |

### Dos bugs reales encontrados por el camino
1. **Colisión de identificadores.** El id del criterio era `par-tf-sello` con
   sello de **segundos**: dos criterios creados en el mismo segundo colapsaban y
   el segundo se encontraba con la valla de inmutabilidad de un criterio ajeno.
   Corregido con entropía, como `jobs/store.py`.
2. **`rule` vacío en la confirmación.** `ensamblar` pone el nombre de la regla
   en el **bloque**, no en la fila; el runner lo leía de la fila y pasaba
   `''` a `build_strategy`. Fallaba **después de haber gastado la ventana**.
   Bug latente del código anterior, que nunca se había ejecutado entero.

### Decisión de diseño que conviene conocer
El planificador de ventanas elige **dentro del rango no declarado**, no sobre la
grabación entera. Sin esto, en cualquier par con historial —que es todos los que
llevan tiempo— el preflight se negaría siempre: casi todo ya está declarado,
porque declarar es lo primero que se hizo.

---

## 3. Etapa 2 — Interfaz y valla del backend

### Archivos nuevos
| Archivo | Qué es |
|---|---|
| `dashboard/guardia.py` | Frontera usuario/sistema; lista de endpoints retirados; bandera de desarrollo |
| `dashboard/templates/_lab_preparar.html` | El resumen previo, **solo lectura** |
| `dashboard/templates/criterios.html` | Índice de criterios escritos, sin un solo botón de escribir |
| `dashboard/templates/_criterio_lectura.html` | El documento de auditoría, generado |

### Las dos capas de la valla
1. **`jobs_reader.config_laboratorio`** — lista blanca en la puerta por la que
   pasa **todo** trabajo nuevo que crea el panel. Rechaza antes de crear nada,
   que es lo que evita gastar una ventana para descubrir después que el
   criterio no era el suyo.
2. **El runner** — no lee ningún parámetro científico de la configuración del
   trabajo. No hay por dónde pasarlos.

Ocultar campos en el HTML no habría bastado, y por eso el test comprueba el
`source` del runner, no solo el formulario.

### Rutas retiradas de la interfaz, no borradas
`declarar`, `gastar`, `ensayar`, `criterio`, `barrer`, `confirmar`, `pipeline`,
`probar` → **410 Gone** por middleware (uno solo, no ocho), y
`/laboratorio/avanzado` → 410 sin `GTS_LAB_AVANZADO=1`.

**No se borra el CLI, ni los runners, ni los trabajos históricos.** Eso está
comprobado con pruebas que los importan y los enumeran.

### Entrenamiento y walk-forward
`_no_dejar_entrar` rechaza épocas, lote, acumulación, lookback, salto, barrera,
volatilidad objetivo, barras, tamaño de modelo, compilación, cortes del
walk-forward… con un motivo que dice **dónde** se cambian de verdad.

### Un fallo preexistente que salió al rodar las pruebas
`test_un_cuerpo_valido_pasa_todas_las_validaciones` **ya estaba roto** antes de
esta sesión: las tres puertas ya no llaman a `Popen` (encolan y lanzan el worker),
así que el marcador de «llegar al final» había cambiado. Corregido y **aislado**:
la prueba escribía en la cola real.

---

## 4. Etapa 3 — Descubrimiento automático

El modo por defecto es **descubrir**: el objetivo concreto es opcional. El
universo es la intersección de estrategias registradas × rejillas declaradas ×
ámbito del perfil.

El dato importante, y la razón de que no se haya hecho «solo ejecutar varias
estrategias»: se confirma **una configuración por regla**, todas antes de
elegir nada. Elegir la ganadora global y confirmar solo esa sería una segunda
búsqueda dentro de la ventana limpia.

`tests/test_descubrimiento.py` verifica lo que el encargo nombró:
registro previo del universo, aislamiento temporal, correcciones estadísticas y
trazabilidad. Más uno propio: que **«ganar» no sea «rentabilidad histórica»**.

---

## 5. Evidencia

### 5.1 Pruebas

Corrida final, en la máquina descargada (nada más corriendo):

| Suite | Resultado |
|---|---|
| `tests/test_metodologia_criterio.py` | 30 — verde |
| `tests/test_laboratorio_integracion.py` | 4 — verde |
| `tests/test_descubrimiento.py` | 16 — verde |
| `tests/test_guardia_cientifica.py` | 54 — verde |
| `tests/test_como_va_metricas.py` (arreglado) | 4 — verde, +1 nuevo |
| `tests/` completo | **1 595** — 0 fallos, 0 errores |
| `dashboard/tests/` completo | **493** — 0 fallos, 0 errores |

Los dos raíces se ejecutan **por separado**: juntos, el paquete `app` de la raíz
se come el del panel y cuatro ficheros de pruebas no llegan a importar. No es un
fallo del código, es el nombre del paquete, y por eso está escrito aquí para que
nadie lo lea como un rojo.

Una nota sobre errores que aparecieron y disappeared: en varias corridas con
trabajo pesado en paralelo (el entrenamiento que se me escapó, dos paneles de
pruebas) `tests/test_training_provenance.py` dio **ERROR** en el setup. Sola da
verde, y en la máquina descargada la suite entera da verde. Lo de `ERROR` en
setup con la máquina saturada no es un fallo de código, y decirlo es más útil que
repetir la corrida hasta que salga el número que uno quiere ver.

### 5.2 Navegador real (panel arrancado, MT5 enlazado)
- `/laboratorio` muestra **tres** desplegables y nada más. Sin «Modo avanzado».
- Elegir XAUUSD + M15 responde **«No hay metodología para esto»** con el motivo
  técnico y qué falta. No sugiere cambiar un campo, porque no hay ninguno.
- `/investigacion/criterios` lista el criterio con su metodología, su estado
  (`implementada`) y su hash.
- El documento generado se lee entero y dice, entre otras cosas, que
  «*un test verde no es evidencia de rentabilidad*».

### 5.3 Peticiones manipuladas (desde HTTP, no desde el formulario)
| Petición | Respuesta |
|---|---|
| `POST /laboratorio/investigar` con `umbral_t=0.01&min_ops=1` | **200** con el motivo, nombrando las claves. **No se encoló nada.** |
| `POST /laboratorio/barrer` | **410** |
| `POST /laboratorio/declarar` | **410** |
| `GET /laboratorio/avanzado` | **410** |

### 5.4 Corrida real con datos reales y costes declarados
`scripts/corrida_integracion.py`, sobre **XAUUSD M15** — el par del trabajo que
falló — con datos ampliados a 80 000 velas desde 2023-05-16:

```
preflight → preinscrito → declarado → búsqueda (5 configuraciones)
→ reality check (White, 10 000 remuestreos, 5 ensayos)
→ elegida (mejor por regla) → medición (2 reglas) → INCONCLUSO
```

- Criterio: `XAUUSD-M15-ingenieria_v1-20261012-144723-8254d6`, hash
  `ae8e45c0033a…`, metodología `ingenieria_v1 v1 (implementada, uso ingeniería)`.
- Ventanas: `examinar-2` 2023-05-16 → 2023-11-16 (confirmación) y
  `entrenar-2` 2023-11-16 → 2024-08-20 (selección). **La confirmación va antes.**
- Las ventanas **anteriores** (`examinar-1`, `entrenar-1`) siguen **limpias**.
- Veredicto: **INCONCLUSO** — 11 y 9 operaciones, por debajo de las 30 mínimas.
  Ninguna regla declarada ganadora. Motivo por regla, literal.
- Progreso real por configuración y por fase, visible en los eventos del trabajo.

**Lo que esta corrida NO demuestra:** nada sobre rentabilidad. El perfil es de
ingeniería, la metodología no está aprobada y el veredicto es «inconcluso».

---

## 6. Incidencias y decisiones que necesito del dueño

### 6.1 Metodología aprobada: hoy no hay ninguna
El flujo normal **se niega a investigar**, con el motivo. Esto es lo que pediste
—«no inventes una aprobación para desbloquear la interfaz»— y es lo que hace el
sistema. Para desbloquearlo hace falta evidencia fuera de muestra y un perfil
nuevo con `estado: aprobada`. Eso es trabajo de investigación, no de software.

### 6.2 Error mío: arranqué tu entrenamiento en cola
`corrida_integracion.py` arrancó un worker **contra la cola de producción**, y el
worker reclama el trabajo **más antiguo**: se llevó tu `job-...064349-936caa`
(entrenamiento 40 épocas), que llevabas en cola desde las 06:43.

- **Contenido:** detenido en la epoch 3/40. Marcado `interrupted` con el motivo.
- **No he borrado nada.** Quedan `epoch_0.pt` y `best_model.pt` (41 MB cada uno)
  en `checkpoints_training/`. Reencolarlo es tuyo.
- **Corregido de raíz:** el script crea ahora su propia cola
  (`data/jobs/_integracion-<id>.db`), porque el problema era usar la de
  producción.

También detecté que **esos dos trabajos ya traían** un `progress_file` de un
`temporary` de pytest desde las 06:43, antes de esta sesión. No lo he tocado; lo
señalo porque el entrenamiento apuntaba a un fichero que ya no existe.

**Consecuencia del error, y una valla que estaba de más.** Ese entrenamiento
escribió `checkpoints_training/best_model.pt`. Con él, un test del panel que
llevaba meses en verde se puso rojo: `test_como_va_metricas.py`, que comprobaba
que la ficha del bot **dice «sin checkpoint»**. El código del panel hacía bien —
nombraba el checkpoint que acababa de aparecer—, y el test se puso rojo por un
hecho de la máquina, no por un fallo.

No he borrado el checkpoint para volver a ponerlo verde: eso sería esconder la
evidencia y además tirar tu trabajo parcial. Lo que he hecho es **arreglar el
test**, que es lo que estaba mal:

* `test_la_fuente_de_senal_lleva_la_fila_del_checkpoint` monta ahora su propia
  premisa (motor apuntando a un directorio vacío) en vez de depender de lo que
  haya en el disco.
* Y he añadido `test_con_un_checkpoint_en_disco_la_fila_lo_nombra`, que
  comprueba el caso contrario: **con** checkpoint, la fila lo nombra y no dice
  que no lo hay. Ese test no existía, y es el que importa más.

Un test que depende del estado de la máquina no es una valla: es una casualidad
que se rompe sola.

### 6.3 La suite del panel escribía en la cola real
`test_entrenamiento_confirmar.py` encolaba en `data/jobs/trabajos.db`. Cuatro
trabajos falsos se crearon y los borré. La prueba **ya estaba rota** antes de esta
sesión (las puertas ya no usan `Popen`); la he arreglado y la he aislado.

Hay un segundo sitio con el mismo problema, en `tests/` y en `dashboard/tests/`:
los fixtures que usan `tmp_path` fallan en el setup bajo este sandbox
(`PermissionError` en `C:\Users\fiver\AppData\Local\Temp\pytest-of-fiver`), así
que las pruebas nuevas usan directorios propios bajo `data/tests/` y un
`--basetemp` nuevo. Es el mismo motivo por el que `tests/test_jobs_worker.py`
usa `data/jobs/_test-*` desde hace tiempo.

### 6.4 `docs/experimentos/*.md` y `config/criterios.json`
Intactos, como pediste. Los 4 archivos legacy conservan su fecha original.

### 6.5 Lo que **no** he hecho
- **No he hecho commits.** El árbol tiene trabajo sin commitear de otra sesión
  (`jobs/`, `dashboard/jobs_reader.py`, `research/sweep.py`, `training/`,
  `scripts/walkforward_eval.py`…) y varios de los ficheros que he tocado
  contienen ese trabajo además del mío. Un commit por etapa arrastraría lo
  ajeno, y uno único no respeta la separación que pediste. **Decide tú.**
- **No he aprobado ninguna metodología**, ni he tocado el walk-forward más allá
  de la valla de parámetros: no hay metodología de validación que lo autorice.
- **No he ejecutado el barrido de 80 configuraciones.** No lo pediste y no
  habría creado evidencia: la metodología no está aprobada.

---

## 7. Archivos míos, para revisarlos

**Nuevos**
```
config/metodologias/{universal_v1,ingenieria_v1}.yaml
research/{metodologia,criterios,preinscripcion}.py
dashboard/guardia.py
dashboard/templates/{_lab_preparar,criterios,_criterio_lectura}.html
scripts/corrida_integracion.py
tests/test_{metodologia_criterio,laboratorio_integracion,descubrimiento,guardia_cientifica}.py
```

**Modificados**
```
core/criteria.py                       (reescrito)
research/{registro,experimento}.py
jobs/runners/pipeline.py               (reescrito)
dashboard/{app/main.py,jobs_reader.py}
dashboard/templates/{laboratorio,_lab_empezar,_trabajo_detalle}.html
dashboard/tests/{test_chrome_estatico,test_dos_caminos_un_dato,test_entrenamiento_confirmar,test_trabajos_pages}.py
```