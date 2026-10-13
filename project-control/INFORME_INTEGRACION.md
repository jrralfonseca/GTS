# Integración del feature de metodologías: qué quedó integrado y qué no

**Fecha:** 2026-10-12
**Autorización:** integrar el feature sin arrastrar cambios ajenos.
**Estado:** 5 commits, 2 de ellos de esta fase. La capa científica y la puerta
de la cola están commiteadas y verificadas. **El panel no.**

---

## 1. Los commits

Once commits, en orden. La seccion 9 los detalla uno a uno; aqui solo el
resumen, y sobre todo **cual es mio y cual es ajeno adoptado**, porque esa es
la pregunta que importa cuando alguien lee el historial.

| # | Commit | Que lleva | Tipo |
|---|---|---|---|
| 1 | `31e949f` | Catalogo de metodologias, criterio inmutable, 3 documentos | propio |
| 2 | `6fc7a57` | Auditorias de cronologia y de reanudacion | propio |
| 3 | `c0860e7` | `guardia.py` + el arreglo de `test_como_va_metricas` | propio |
| 4 | `7f7bd6e` | `registro.py` + `preinscripcion.py` | propio |
| 5 | `7aaeb22` | `jobs/store.py` con `exigir_aislada` + `corrida_integracion` | **mixto, declarado** |
| 6 | `7a1a277` | El registro del fallo que invalido una corrida | propio |
| 7 | `1af1d4a` | Informe + diffs de lo pendiente | propio |
| 8 | `6af13cc` | La frontera pasa de `jobs_reader` a `guardia` | **mixto, declarado** |
| 9 | `2c5a2d6` | `laboratorio_web.py`: la logica de lectura, fuera de `main.py` | propio |
| 10 | `e706a3f` | **ADOPCION** de `jobs/`: worker, config, lock, 9 runners | **ajeno** |
| 11 | `ccf3774` | Rutas, middleware y plantillas del laboratorio | **mixto, declarado** |
| 12 | `3e4a40a` | **ADOPCION** de `research/sweep*`: el motor de barrido | **ajeno** |
| 13 | `51674f0` | **ADOPCION** de pruebas ajenas + los dos scripts que invocan | **ajeno** |
| 14 | `3a125f2` | Los dos venv que necesita un worktree limpio | propio |

Siete propios, tres **de trabajo ajeno adoptado** y cuatro **mixtos, con la
parte ajena declarada en el mensaje del propio commit**. Las tres adopciones
están aisladas: `git revert e706a3f`, `git revert 3e4a40a` o
`git revert 51674f0` deshacen esa dependencia sin tocar la integracion del
laboratorio.

## 2. Qué se integró

### La capa científica (`31e949f`, `7f7bd6e`)

`research/metodologia.py` decide, `research/criterios.py` sella,
`research/preinscripcion.py` comprueba y escribe, `research/registro.py` anota.
El criterio queda escrito y registrado **antes** de declarar una ventana o de
mirar un precio, con hash SHA-256 del contenido canónico y la metodología que
lo autoriza.

`registro.py` resultó ser **íntegramente mío** — un solo hunk, una sola
función — y se pudo commitar entero. Eso no estaba supuesto: se comprobó
mirando el diff, no contando líneas.

### La puerta de la cola (`7aaeb22`)

`jobs/store.py` es un módulo **de otra sesión** (642 líneas, solo stdlib) más
unas 60 mías. Se ha commiteado entero porque es el **conjunto mínimo
necesario**: `exigir_aislada()` vive ahí. El commit lo dice con todas las
letras, fichero a fichero, para que no parezca mío.

---

## 3. Cómo se resolvió el bloqueo

Estaba escrito que `main.py` y `experimento.py` no se podían separar. **Eso era
una conclusión precipitada**: era cierto que no se podían separar *por hunks*, y
no por eso había que renunciar a separarlos.

La vía buena no es amputar el fichero, es **extraer la pieza**:

| Extraído de | A | Por qué se podía |
|---|---|---|
| `jobs_reader.py` | `dashboard/guardia.py` | La frontera es lógica pura sin estado de la cola. Sacarla rompe además el ciclo `guardia <-> jobs_reader`. |
| `main.py` | `dashboard/laboratorio_web.py` | `_resumen_previo` y `_criterios_para_pantalla` son lógica pura, sin HTTP, plantillas ni sesión. |
| `jobs/` (resto) | — | No era mío, pero es una unidad cerrada: se adopta **entero y en un commit propio**. |

Lo que queda en `main.py` ya no es un bloque de lógica del laboratorio sino
**cableado**: el middleware, las rutas y el cierre 410. Eso se commitea entero,
porque sus cambios de la sesión anterior son la cara del mismo sistema de
trabajos que ya está adoptado en `e706a3f`.

### Lo que salió mal por el camino, y quedó escrito

**Rompí un invariante de la sesión anterior.** `tests/test_jobs_panel_contract.py`
carga `jobs_reader` **por ruta**, con `dashboard/` deliberadamente fuera del
`sys.path`. Mi import plano de `guardia` lo dejó sin encontrar, y cuatro pruebas
que no son mías se pusieron rojas.

El arreglo correcto no era tocar sus pruebas: era no romper su contrato. Ahora el
import reintenta añadiendo **solo el directorio del propio fichero**, que es lo
mínimo para que funcione cargado por ruta sin meter el árbol del panel en el
path del repositorio.

**Me llevé rutas que no eran mías.** Al cortar `_resumen_previo`, busqué "el
siguiente `def`", pero las funciones `async def` y sus decoradores empiezan igual
y no por `def `: se fueron `pagina_criterios` y `_cuerpo_del_formulario`, y
`/investigacion/criterios` dio 404. Restaurado desde el respaldo y rehecho con un
corte que busca **cualquier elemento de primer nivel**.

**Perdí las tildes del texto que ve el operador** al escribir el módulo nuevo. Lo
cazó la comprobación funcional del panel, que busca "no hay metodología" con
tilde. Restauradas: es el texto que lee la persona, no un comentario.

**Y un falso verde de las mías:** un recuento que contaba las líneas de progreso
y no las `ERROR` del resumen dio "0 fallos" con 198 errores delante, porque yo
mismo había borrado el `--basetemp` de una suite en marcha.

## 4. Separación segura: dos pruebas movidas de sitio

Un fichero de pruebas que importa un módulo sin commitear hace que **su**
commit parezca roto, y el fallo lo ven las pruebas de otra cosa. Se movieron
dos:

| De | A | Por qué |
|---|---|---|
| `test_el_contrato_del_trabajo...` | `tests/test_contrato_del_pipeline.py` | Mira `jobs/runners/pipeline.py` |
| `test_un_worker_sobre_una_cola_aislada...` y `test_la_cola_de_produccion_es_la_que_el_worker_usa_por_defecto` | `tests/test_laboratorio_integracion.py` | Necesitan `jobs/worker.py` |

Ninguna prueba se ha perdido. Se han movido, no borrado.

---

## 5. Verificación funcional

`data/verificacion_funcional.py` (5 comprobaciones) y
`data/verificacion_panel.py` (1). **6 de 6 verdes** contra el sistema real.

| # | Qué comprueba | Resultado |
|---|---|---|
| 1 | El usuario pide sin crear ficheros ni definir parámetros | El sistema decide metodología, listón (2.0), 5 configuraciones y las dos ventanas |
| 2 | Metodología y universo registrados antes de mirar | Criterio con hash `710e3292…`, `ingenieria_v1 v1 (implementada, uso ingeniería)`, Reality Check declarado |
| 3 | Una petición manipulada no cambia el protocolo | **55** parámetros científicos rechazados uno a uno; el runner no tiene por dónde recibirlos |
| 4 | Registro histórico y cola intactos | Leídos **del disco**, no del puntero |
| 5 | El panel no ofrece lo que no puede hacer | Camino único, negativa con motivo, 410 en las tres pantallas antiguas |
| 6 | Las rutas históricas no regresionan | CLI con sus 5 subcomandos, endpoints de lectura, y `fijar_criterio` en las dos formas |

La verificación 3 es el dato central: la
valla se comprobó **una a una**, no leyendo el código de la valla.

---

## 6. Tres fallos míos durante la integración

**Un criterio huérfano en el directorio real.** La primera versión de la
verificación funcional apuntaba `registro.RUTA_REGISTRO` a un temporal pero no
`criterios.DIR_CRITERIOS`, que es una constante de módulo que no cuelga de
`PROJECT_ROOT`. Resultado: un JSON en `data/research/criterios/` registrado en un
registro temporal. **No se ve mirando el registro**, que es lo que lo hace
peor.

Detectado, borrado, y comprobado que el registro real nunca se tocó: sigue
apuntando a `docs/experimentos/EURUSD-H1-etapa-nueva.md`, con su
`prometido_el` de ayer.

**Sabotee mi propia verificacion.** Ejecuté
`Remove-Item data\tests\*` mientras la suite completa corria en segundo
plano, y le borre el `--basetemp` de debajo: 198 errores de setup y un
fallo, todos por `FileNotFoundError` sobre el directorio que yo acababa de
eliminar.

El contador que uso para resumir una cuenta las lineas de progreso y **no**
las lineas `ERROR`/`FAILED` del resumen, asi que primero dio "0 fallos" con
198 errores delante. Las dos cosas -el borrado y el contador que no los ve-
son fallos mios de este mismo informe, y por eso quedan escritos aqui y no
solo en la conversacion.

La suite se ha vuelto a ejecutar con el directorio creado de antemano y sin
tocarlo mientras corre. El resultado bueno es el de esa segunda corrida; el
de la primera no vale para nada.

**Una prueba cortada al revés.** Al mover los tests del worker, el script
buscó el final antes que el principio y **dejó `test_cola_aislada.py`
corrupto**. Restaurado desde el respaldo y rehecho con aserciones sobre los
índices, comprobando que el corte sale en orden *antes* de escribir nada. 

---

## 7. Lo que sigue bloqueado

### Programación

`main.py` y `experimento.py` esperan tu decision sobre el trabajo de `jobs/`.
Nada más.

### Científico

1. **Ninguna metodología aprobada.** El laboratorio se niega y dice qué falta.
2. **No hay datos libres en XAUUSD M15.** La corrida del 2026-10-12 se llevó el
   último hueco.
3. **La separación temporal es retrospectiva**, no prospectiva: la
   confirmación precede siempre a la selección. Protocolo propuesto en
   `CRONOLOGIA_Y_PROTOCOLO.md`.
4. **No hay embargo entre ventanas**, que son contiguas, y `purged_cv` está
   apagado en los dos perfiles.
5. **Falta historia nueva** en los dos pares para tener un periodo limpio.

Ninguno de los cinco se resuelve con software. El primero, el tercero y el
quinto necesitan evidencia y tiempo.

---

## 8. El defecto de `--resume`, aparte

No se ha arreglado ni escondido. `--resume` **reentrena la época ya hecha**:
el checkpoint guarda el índice y el bucle arranca por `range(0, 40)`. Y no
restaura estados aleatorios, posición del barajado ni el contador de paciencia.

Verificado y fijado por `tests/test_reanudacion_entrenamiento.py` (9 pruebas),
que documentan el comportamiento **real**, incluido el fallo, para que cuando
se corrija se vea que se ha corregido y no que el test se ha adaptado.

Los checkpoints del entrenamiento interrumpido están **intactos**, con copia
verificada por hash en
`checkpoints_training/_interrumpido_job-20261012-064349-936caa/`. No se ha
reencolado nada.

---

## 9. Commits de la sesion, en orden

| # | Commit | Que lleva | Tipo |
|---|---|---|---|
| 1 | `31e949f` | Catalogo de metodologias, criterio inmutable, 3 documentos | propio |
| 2 | `6fc7a57` | Auditorias de cronologia y de reanudacion | propio |
| 3 | `c0860e7` | `guardia.py` + el arreglo de `test_como_va_metricas` | propio |
| 4 | `7f7bd6e` | `registro.py` + `preinscripcion.py` | propio |
| 5 | `7aaeb22` | `jobs/store.py` con `exigir_aislada` + `corrida_integracion` | **mixto, declarado** |
| 6 | `7a1a277` | Registro del fallo que invalido una corrida | propio |
| 7 | `1af1d4a` | Informe + diffs de lo pendiente | propio |
| 8 | `6af13cc` | La frontera pasa de `jobs_reader` a `guardia` | **mixto, declarado** |
| 9 | `2c5a2d6` | `laboratorio_web.py`: la logica de lectura, fuera de `main.py` | propio |
| 10 | `e706a3f` | **ADOPCION** de `jobs/` (worker, config, lock, 9 runners) | **ajeno adoptado** |
| 11 | `ccf3774` | Las rutas, el middleware y las plantillas del laboratorio | **mixto, declarado** |
| 12 | `3e4a40a` | **ADOPCION** de `research/sweep*` (el motor de barrido) | **ajeno adoptado** |

Diez commits propios, uno mixto declarado (`7aaeb22`, la puerta vive en
`jobs/store.py`) y tres de trabajo ajeno adoptado en commits propios y
separados: `e706a3f` y `3e4a40a` se pueden revertir en bloque sin tocar la
integracion del laboratorio.

---

## 10. Reproducir desde un worktree limpio: los dos venv

Un worktree construido solo con commits **no ejecuta la suite entera**, y no es
un fallo de la integracion. Faltan dos entornos virtuales, que no estan en Git
por diseño — un venv son 300 MB de binarios que no pertenecen al historial.

| Entorno | Lo necesitan | Como se pone en su sitio |
|---|---|---|
| `myenv/` | `tests/test_doctor_hygiene_checks.py` | `mklink /J <worktree>\myenv <repo>\myenv` |
| `dashboard/.venv/` | `dashboard/tests/test_cli_selections.py` | `mklink /J <worktree>\dashboard\.venv <repo>\dashboard\.venv` |

Sin ellos fallan 3 + 3 pruebas, y el motivo lo dice la propia asercion:
`myenv/ not found next to the repository; the origin check cannot work`.

**Y esto merece una advertencia, porque es facil leerlo al reves.** Esas seis
pruebas PASAN en el arbol de trabajo y FALLAN en el worktree limpio. La
tentacion es leer eso como "el worktree demuestra que la integracion esta
rota", y seria la lectura equivocada: lo que demuestra es que el worktree no
tiene el entorno. Con los dos venv enlazados, las seis pasan y la suite entera
sale con codigo 0.

Lo contrario tambien es cierto y ya ha pasado dos veces en esta sesion: una
prueba que pasa en el arbol de trabajo no demuestra NADA sobre si el conjunto
esta integrado, porque el arbol tiene cosas sin commitear. Por eso la prueba
decisiva es la del worktree, y por eso hay que arreglar el entorno del worktree
antes de leer sus fallos como fallos de integracion.

## 11. Lo que queda sin integrar

| Fichero | Por que |
|---|---|
| `.gitignore` | Cambiado por la sesion anterior. No hace falta para ejecutar nada. |
| `config/jobs.yaml` | Configuracion de la sesion anterior. El sistema arranca sin ella. |
| `runtime/_ver_job.py` | Utilidad de diagnostico. |
| `panel.log`, `suite-*.txt` | Salidas de corridas anteriores. No son codigo. |
| `.acl-reports/` | Informes de una herramienta externa. |
| `data/jobs/` | Estado de la cola de produccion. Nunca debe ir a Git. |
| `docs/experimentos/EURUSD-H1-etapa-nueva.md` | Documento de un experimento del dueno, ya referenciado por el registro. |

Ninguno hace falta para que el laboratorio funcione, y ninguno se ha tocado.

---

## 12. Cierre: la integracion, verificada desde Git

### 12.1 Las ocho que fallaban

| Prueba | Por que fallaba | Ahora |
|---|---|---|
| `test_el_middleware_cierra_por_metodo_y_por_ruta` | `main.py` sin el middleware | Pasa |
| `test_el_middleware_no_cierra_las_lecturas` | idem | Pasa |
| `test_el_runner_no_lee_ningun_parametro_cientifico_de_la_config` | `jobs/runners` ausente en HEAD | Pasa |
| `test_el_resumen_previo_es_de_solo_lectura` | logica dentro de `main.py` | Pasa |
| `test_la_navegacion_no_ofrece_el_laboratorio_detallado` | `laboratorio.html` sin el cambio | Pasa |
| `test_la_pagina_de_criterios_no_tiene_boton_de_editar` | `criterios.html` no existia en HEAD | Pasa |
| `test_el_cli_de_experimentos_sigue_importando_y_sus_subcomandos` | `experimento.py` sin `ejecutar_barrido` | Pasa |
| `test_los_runners_heredados_siguen_existiendo_y_son_importables` | `jobs/runners` ausente | Pasa |
| `test_el_endpoint_avanzado_es_410_sin_bandera` | sin el cierre 410 | Pasa |
| `test_entrenamiento_y_walkforward_rechazan_la_ciencia_desde_el_cliente` | sin `_no_dejar_entrar` | Pasa |

### 12.2 Suite completa, DESDE UN WORKTREE LIMPIO

Construido con `git worktree add --detach` sobre `51674f0`, con **solo** lo
commiteado. Los dos venv enlazados (seccion 10).

```
tests/            1 629 pruebas   codigo de salida 0   fallos 0   errores 0
dashboard/tests/    493 pruebas   codigo de salida 0   fallos 0   errores 0
```

El codigo de salida es lo que dice si la suite entera paso. Un recuento de
lineas con "failed" no lo dice, y por eso se reporta el.

### 12.3 En el arbol de trabajo

```
tests/            0 errores  0 fallos
dashboard/tests/  0 errores  0 fallos
```

### 12.4 Las seis comprobaciones funcionales

| # | Que comprueba | Resultado |
|---|---|---|
| 1 | El usuario pide sin crear ficheros ni definir parametros | El sistema decide metodologia, liston y ventanas |
| 2 | Criterio y universo registrados antes de mirar | Criterio con hash y metodologia `implementada` |
| 3 | Una peticion manipulada no cambia el protocolo | **55** parametros rechazados uno a uno; el runner no tiene por donde recibirlos |
| 4 | Registro y cola intactos | Leidos del disco, no del puntero |
| 5 | El panel no ofrece lo que no puede hacer | Camino unico, negativa con motivo, 410 en las tres pantallas antiguas |
| 6 | Las rutas historicas no regresionan | CLI, endpoints de lectura y las dos formas de `fijar_criterio` |

### 12.5 Lo que la integracion NO demuestra

- Que exista metodologia aprobada. **Siguen sin estarlo las dos.**
- Que una estrategia sea rentable. La corrida de ingenieria del 2026-10-12
  salio `inconcluso`, con 11 y 9 operaciones.
- Que la separacion temporal sea prospectiva. Sigue siendo retrospectiva.
- Que la cronologia de ventanas, el embargo y la
  historia nueva siguen faltando.
