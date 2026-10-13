# Diffs de lo que queda pendiente, para revisar

**Generado:** 2026-10-12. **RESUELTO el 2026-10-12 — esta archivado como material de revisión histórica.**

> **ESTADO FINAL:** la decisión que este documento esperaba ya se tomó por la
> vía de los hechos, y a favor de la integración completa. Los dos ficheros
> que "no se podían separar con seguridad" (`dashboard/app/main.py` y
> `research/experimento.py`) quedaron commiteados ENTEROS por la sesión de
> integración (commits `ccf3774`, `3e4a40a`, `51674f0` y siguientes; ver
> `INFORME_INTEGRACION.md`, que registra commit a commit qué era propio y qué
> era adoptado), y el resto de ficheros que acompañaban esta revisión se
> commiteó en `5bb0644` (config de la cola, README de la cola, el criterio del
> dueño y el informe de trabajos). El árbol de trabajo quedó limpio el
> 2026-10-12: **ya no hay nada pendiente de revisar por esta vía.**
>
> Los dos `.patch` que acompañan este documento quedan como el expediente de
> aquella decisión: muestran exactamente qué hunks se mezclaban y por qué no
> existía un parche parcial correcto. No describen el estado actual de nada.

---

Lo que sigue es el documento TAL COMO SE ESCRIBIÓ cuando era verdad. Se
conserva íntegro porque borrar la historia de una decisión es la manera de
tomarla otra vez.

Estos dos ficheros NO se pueden separar con seguridad: sus cambios
propios y los de otra sesion estan entrelazados DENTRO de los mismos
hunks, no en trozos distintos del fichero. Por eso no hay un parche
parcial que sea correcto, y por eso estan aqui para que la decision sea
tuya y no mia.

Cada hunk lleva lo que hace y quien parece ser. Es una lectura, no una
garantia: la unica forma de estar seguro es leerlos.

---

## `dashboard/app/main.py`

8000 lineas. Mezcla: integracion de jobs/ (otra sesion) + rutas de investigacion y vallas (mias).

**38 hunks** -> 2 solo mios, 21 solo de `jobs/`, 15 de los dos.

### Solo de esta sesion (criterio, metodologia)

- `@@ -6253,0 +6570,7 @@ def _lanzar_entrenamiento(body: Dict[str, Any]) -> ControlResult:`
- `@@ -6636,0 +6947,7 @@ def _lanzar_walkforward(body: Dict[str, Any]) -> ControlResult:`

### Solo de la integracion de jobs/

- `@@ -78,0 +79 @@ import experimentos as exp_ledger`
- `@@ -3696,14 +4001,5 @@ async def laboratorio_barrer(request: Request, symbol: str = Form(""),`
- `@@ -3754,19 +4050,5 @@ async def laboratorio_confirmar(request: Request, symbol: str = Form(""),`
- `@@ -3846,56 +4126,10 @@ async def laboratorio_pipeline(request: Request, symbol: str = Form(""),`
- `@@ -3955,108 +4186,11 @@ async def laboratorio_probar(request: Request,`
- `@@ -6313,0 +6637,5 @@ def _lanzar_entrenamiento(body: Dict[str, Any]) -> ControlResult:`
- `@@ -6325 +6649 @@ def _lanzar_entrenamiento(body: Dict[str, Any]) -> ControlResult:`
- `@@ -6333,21 +6657,16 @@ def _lanzar_entrenamiento(body: Dict[str, Any]) -> ControlResult:`
- `@@ -6355,12 +6674,4 @@ def _lanzar_entrenamiento(body: Dict[str, Any]) -> ControlResult:`
- `@@ -6368 +6679 @@ def _lanzar_entrenamiento(body: Dict[str, Any]) -> ControlResult:`
- `@@ -6371,5 +6682,3 @@ def _lanzar_entrenamiento(body: Dict[str, Any]) -> ControlResult:`
- `@@ -6380 +6689 @@ def _lanzar_entrenamiento(body: Dict[str, Any]) -> ControlResult:`
- `@@ -6658 +6975,3 @@ def _lanzar_walkforward(body: Dict[str, Any]) -> ControlResult:`
- `@@ -6664 +6983 @@ def _lanzar_walkforward(body: Dict[str, Any]) -> ControlResult:`
- `@@ -6669,10 +6988,12 @@ def _lanzar_walkforward(body: Dict[str, Any]) -> ControlResult:`
- `@@ -6680,12 +7001,3 @@ def _lanzar_walkforward(body: Dict[str, Any]) -> ControlResult:`
- `@@ -6693 +7005 @@ def _lanzar_walkforward(body: Dict[str, Any]) -> ControlResult:`
- `@@ -6696,2 +7008,3 @@ def _lanzar_walkforward(body: Dict[str, Any]) -> ControlResult:`
- `@@ -6699 +7012 @@ def _lanzar_walkforward(body: Dict[str, Any]) -> ControlResult:`
- `@@ -6704,2 +7017,3 @@ def _lanzar_walkforward(body: Dict[str, Any]) -> ControlResult:`
- `@@ -7364,0 +7680,5 @@ async def entrenamiento_lanzar(request: Request, clave: str):`

### De los dos: hay que mirarlos a ojo

- `@@ -2437,12 +2438,19 @@ async def pagina_laboratorio(request: Request,`
- `@@ -2451 +2458,0 @@ async def pagina_laboratorio(request: Request,`
- `@@ -2593,0 +2601,18 @@ async def laboratorio_avanzado(request: Request,`
- `@@ -3676,0 +3702,280 @@ def _cmd_barrer(symbol: str, timeframe: str, tramo: str, workers: int,`
- `@@ -3835,2 +4116,0 @@ async def laboratorio_pipeline(request: Request, symbol: str = Form(""),`
- `@@ -3930,3 +4163,0 @@ async def laboratorio_probar(request: Request,`
- `@@ -4076,0 +4211,150 @@ def _resultado_simplificado(request: Request, symbol: str, timeframe: str,`
- `@@ -6231,0 +6516,32 @@ def _simbolos_de(valor: Any, por_defecto: Optional[List[str]] = None,`
- `@@ -6315,4 +6642,0 @@ def _lanzar_entrenamiento(body: Dict[str, Any]) -> ControlResult:`
- `@@ -6382,2 +6691,2 @@ def _lanzar_entrenamiento(body: Dict[str, Any]) -> ControlResult:`
- `@@ -6409,0 +6719 @@ async def train_model(request: Request):`
- `@@ -6411 +6721 @@ async def train_model(request: Request):`
- `@@ -6719,0 +7034 @@ async def start_walkforward(body: Dict = Body(...)):`
- `@@ -6721 +7036 @@ async def start_walkforward(body: Dict = Body(...)):`
- `@@ -7366 +7686 @@ async def entrenamiento_lanzar(request: Request, clave: str):`

Diff completo: `diff-dashboard-app-main.py.patch`

```diff
diff --git a/dashboard/app/main.py b/dashboard/app/main.py
index c127236..03d381f 100644
--- a/dashboard/app/main.py
+++ b/dashboard/app/main.py
@@ -76,6 +76,7 @@ from fastapi.templating import Jinja2Templates
 import control_channel as cc
 import instrumentos
 import experimentos as exp_ledger
+import jobs_reader as jobs_lector
 import journal_reader as jr
 import log_reader
 import pandas as pd
@@ -2434,21 +2435,27 @@ async def fragmento_experimentos(request: Request):
 @app.get("/laboratorio", response_class=HTMLResponse)
 async def pagina_laboratorio(request: Request,
                              symbol: str = "", timeframe: str = ""):
-    """El laboratorio simplificado: tres pantallas, un botón.
-
-    PANTALLA 1 es la que se ve al llegar: elegir qué probar y pulsar. Las
-    pantallas 2 (elegir periodos) y 3 (resultado) llegan por HTMX al mismo
-    contenedor, asi que el operador nunca sale de esta URL.
-
-    El laboratorio detallado — los 7 pasos con todos sus campos — sigue
-    existiendo en `/laboratorio/avanzado`: el protocolo es el mismo, la
-    disciplina es la misma, y quien quiera tocar cada campo a mano puede.
-    Lo que esta pagina hace es envolver el protocolo en un flujo que un
-    operador de trading lee sin traducir: "elige, marca dos periodos, pulsa
-    probar".
+    """El laboratorio: una pantalla para pedir, una para revisar, una para ver.
+
+    Lo que se ve al llegar es UNA pregunta —qué quieres investigar— y un botón
+    que pide la configuración. La segunda pantalla es el resumen previo, en
+    solo lectura: lo que el sistema va a hacer, con su metodología y su estado,
+    las ventanas que reservará y las limitaciones que hay. Se acepta o se
+    cancela, y no se edita: aceptar una metodología es entenderla.
+
+    La lista de estrategias sale del registro y es OPCIONAL, con
+    «descubrir estrategias automáticamente» por defecto. Ponerla como
+    obligatoria obligaría al operador a saber el nombre de una regla antes de
+    preguntarse qué reglas existen, que es justo lo que el laboratorio puede
+    responder mejor.
+
+    El laboratorio detallado de los siete pasos ya no se ofrece desde aquí: sus
+    formularios dejaban fijar el listón y las ventanas, y eso no forma parte
+    de la experiencia de un operador de trading. Sigue existiendo
+    `/laboratorio/avanzado` para desarrollo (`GTS_LAB_AVANZADO`), y sus
+    endpoints de escritura devuelven 410 aunque la página siga ahí.
     """
     from strategies import list_strategies
-    from strategies.registry import default_params
     return templates.TemplateResponse(
         request, "laboratorio.html",
         {"seccion": "laboratorio",
@@ -2591,6 +2598,24 @@ async def laboratorio_avanzado(request: Request,
     tocar pieza a pieza. El protocolo y el registro son los mismos; esta
     es la vista para quien quiere ver las tuercas.
     """
+    """El laboratorio detallado de siempre: los 7 pasos con todos sus campos.
+
+    NO SE OFRECE desde la navegación, y su escritura esta cerrada (410 en sus
+    endpoints). Sigue entrando aqui para dos cosas:
+
+    * poder COMPARAR el camino viejo con el nuevo, que es la unica forma de
+      saber si el nuevo hace lo mismo que el viejo y mejor;
+    * y que los tests que cubren esas rutas sigan teniendo que probar.
+
+    Se abre con `GTS_LAB_AVANZADO=1`. Sin la bandera, esta pagina dice por que
+    no se enseña en vez de fingir que no existe: una pantalla que desaparece sin
+    explicacion parece un error, y un error hay que poder preguntar.
+    """
+    import guardia as _guardia
+    if not _guardia.avanzado_activo():
+        return HTMLResponse(
+            status_code=410,
+            content=_guardia.expediente_viejo("/laboratorio/avanzado"))
     return templates.TemplateResponse(
         request, "laboratorio_avanzado.html",
         {"seccion": "laboratorio",
@@ -3674,6 +3699,286 @@ def _cmd_barrer(symbol: str, timeframe: str, tramo: str, workers: int,
     return cmd
 
 
+async def _encolar_trabajo(request: Request, tipo: str, experimento_id: str,
+                         config: dict, semilla: int | None = None):
+    """Crea un trabajo en la cola y se asegura de que haya worker vivo.
+
+    Es la mitad nueva de REGLA 8: el "un experimento pesado a la vez" ya no
+    lo vigila la memoria del panel -que una recarga lo perdía- sino la cola
+    persistente, que ejecuta UNO a la vez y guarda el turno de los demas.
+
+    Lo que devuelve es UNA respuesta con el enlace a la pagina del trabajo:
+    el resultado ya no llega dentro de esta peticion HTTP, y fingir lo
+    contrario seria la peticion eterna de antes con otro nombre.
+
+    Lo que toca disco y procesos (crear la fila, sondear si la cola ya
+    existe, lanzar el worker) se espera EN UN HILO: son subprocess y sleeps
+    de hasta 10 s la primera vez, y dentro del bucle de eventos eso es el
+    panel muerto que test_no_bloquea_el_panel existe para impedir.
+    """
+    from starlette.concurrency import run_in_threadpool
+
+    def _preparar_cola():
+        if not jobs_lector.existe_cola():
+            jobs_lector.lanzar_worker_si_hace_falta()
+            for _ in range(40):
+                if jobs_lector.existe_cola():
+                    break
+                time.sleep(0.25)
+
+    await run_in_threadpool(_preparar_cola)
+    try:
+        trabajo = await run_in_threadpool(
+            jobs_lector.crear_job, tipo, experimento_id, config,
+            semilla=semilla)
+    except Exception as exc:  # noqa: BLE001 - la cola rota es fallo del panel
+        return _fallido(
+            request, "Cola de trabajos",
+            f"no se pudo crear el trabajo: {exc}. Si acaba de arrancarse el "
+            f"worker por primera vez, espera unos segundos y vuelve a "
+            f"intentarlo; si persiste, el detalle esta en "
+            f"data/logs/jobs-worker.log")
+    ok_worker, msg_worker = await run_in_threadpool(
+        jobs_lector.lanzar_worker_si_hace_falta)
+    aviso = ("El trabajo corre fuera del navegador: puedes cerrar esta "
+             "pestaña y volver; la ejecucion sigue. Un trabajo pesado a la "
+             "vez -los demas esperan su turno en la cola-. La ventana que "
+             "va a mirar este trabajo queda GASTADA aunque canceles despues.")
+    if not ok_worker:
+        aviso += " AVISO: " + msg_worker
+    return _hecho(
+        request, "Trabajo en cola",
+        f"Trabajo {trabajo['id']} ({tipo}) en cola para el experimento "
+        f"{experimento_id}. Progreso y resultado: /trabajos/{trabajo['id']}",
+        aviso=aviso)
+
+
+@app.get("/laboratorio/investigar/preparar", response_class=HTMLResponse)
+async def laboratorio_investigar_preparar(request: Request,
+                                          symbol: str = "",
+                                          timeframe: str = "",
+                                          estrategia: str = ""):
+    """EL RESUMEN PREVIO: lo que el sistema va a hacer, en solo lectura.
+
+    No encola nada, no declara nada, no gasta nada. Solo comprueba —que haya
+    datos, que haya metodología, que las ventanas que la metodología propone no
+    pisen nada declarado— y lo cuenta con palabras. Es la pantalla donde el
+    usuario ACEPTA, y por eso no puede haber un campo editable: aceptar una
+    metodología es entenderla, y entenderla exige verla entera.
+
+    Si no hay metodología aprobada, esto NO dice «cambia un campo»: dice que no
+    hay metodología validada y qué falta. Un test verde no crea evidencia, y
+    esta pantalla no puede sugerir lo contrario.
+    """
+    from starlette.concurrency import run_in_threadpool
+    preparado = await run_in_threadpool(
+        _resumen_previo, symbol, timeframe, estrategia)
+    return templates.TemplateResponse(
+        request, "_lab_preparar.html",
+        {"symbol": symbol, "timeframe": timeframe, "estrategia": estrategia,
+         "preparado": preparado, "ahora": _utc_ahora(),
+         "asset_v": _asset_version()})
+
+
+def _resumen_previo(simbolo: str, temporalidad: str,
+                    estrategia: str = "") -> dict[str, Any]:
+    """Comprobaciones de solo lectura para la pantalla de inicio. Nunca escribe.
+
+    Usa `preinscripcion.preparar`, que es exactamente lo mismo que hará el
+    trabajo cuando arranque, y por eso la pantalla no puede prometer una cosa y
+    el trabajo hacer otra: si esto pasa, el trabajo pasa.
+    """
+    from research import preinscripcion as pre
+
+    out: dict[str, Any] = {"simbolo": simbolo, "temporalidad": temporalidad,
+                           "estrategia": estrategia, "puede": False,
+                           "motivo": "", "titulo": ""}
+    if not (simbolo and temporalidad):
+        return out
+    try:
+        prep = pre.preparar(simbolo, temporalidad,
+                            estrategia=estrategia or None)
+    except pre.PreflightError as exc:
+        texto = str(exc)
+        # Se distingue «no hay metodología» de «el par no está listo» porque
+        # son dos problemas con dos acciones distintas, y con dos mensajes
+        # distintos.
+        out["titulo"] = ("No hay metodología para esto"
+                         if "metodolog" in texto.lower()
+                         else "Este par todavía no está listo")
+        out["motivo"] = texto
+        return out
+
+    uni = prep.universo
+    out.update({
+        "puede": True,
+        "titulo": "Configuración preparada",
+        "metodologia": prep.perfil.nombre,
+        "metodologia_estado": prep.perfil.estado,
+        "metodologia_uso": prep.perfil.uso,
+        "aprobada": prep.perfil.aprobado,
+        "descubrimiento": not uni.get("objetivo"),
+        "estrategias": uni.get("estrategias", []),
+        "n_configuraciones": uni.get("n_configuraciones", 0),
+        "descartadas": uni.get("descartadas", []),
+        "umbral_abs_t": prep.perfil.umbral_abs_t,
+        "min_operaciones": prep.perfil.min_operaciones,
+        "correccion": prep.perfil.correccion.get("metodo"),
+        "broker": prep.perfil.broker,
+        "ventanas": [(v.nombre, v.tipo, v.desde, v.hasta) for v in prep.ventanas],
+        "velas": prep.datos.velas,
+        "periodo": (prep.datos.primera, prep.datos.ultima),
+    })
+    return out
+
+
+async def _cuerpo_del_formulario(request: Request) -> dict[str, Any]:
+    """El formulario entero, tal cual lo mandó el cliente.
+
+    Existe para poder decir «has enviado X» en vez de «falta Y»: una valla que
+    solo sabe lo que espera no puede enseñar lo que le mandaron de más, y quien
+    la salta necesita entender por qué no entró.
+    """
+    try:
+        form = await request.form()
+    except Exception:  # noqa: BLE001 - sin cuerpo no hay nada que reprobar
+        return {}
+    return {str(k): v for k, v in form.items()}
+
+
+@app.post("/laboratorio/investigar", response_class=HTMLResponse)
+async def laboratorio_investigar(request: Request, symbol: str = Form(""),
+                                timeframe: str = Form(""),
+                                estrategia: str = Form(""),
+                                workers: int = Form(4)):
+    """EL BOTÓN ÚNICO. Pide una investigación; el sistema decide cómo.
+
+    Lo que acepta esta ruta es lo que el usuario decide de verdad: el activo, la
+    temporalidad y, si quiere, una estrategia concreta. Nada más.
+
+    Lo que NO acepta, y se rechaza aquí y no más abajo, es todo lo demás: el
+    listón, el mínimo de operaciones, las rejillas, las ventanas, el número de
+    ensayos, los controles estadísticos. La valla está en
+    `jobs_reader.config_laboratorio`, que ve TODO trabajo que crea el panel; y
+    aunque se pasara, el runner saca esos números del criterio y no de la
+    petición. Dos capas, y la segunda es la que de verdad protege.
+
+    `estrategia` vacía significa DESCUBRIR: el universo entero del catálogo
+    intersecado con la metodología. Es el flujo predeterminado, y por eso no
+    hay ningún desplegable de «qué reglas conoces».
+    """
+    cuerpo = await _cuerpo_del_formulario(request)
+    try:
+        jobs_lector.config_laboratorio({
+            "symbol": symbol, "timeframe": timeframe,
+            "estrategia": estrategia, "modo": "investigar",
+            "workers": workers})
+    except jobs_lector.ConfigCientificaError as exc:
+        return _fallido(request, "No se puede", str(exc))
+
+    rechazadas = sorted(set(cuerpo) & set(jobs_lector.PARAMETROS_CIENTIFICOS))
+    if rechazadas:
+        return _fallido(
+            request, "No se puede",
+            f"has enviado {', '.join(rechazadas)}, que son parámetros "
+            f"científicos. La metodología no se fija desde el panel: la elige "
+            f"el sistema de config/metodologias/ y la deja escrita en el "
+            f"criterio antes de mirar. Esta pantalla elige QUÉ investigar; el "
+            f"CÓMO no se toca.")
+
+    if not (symbol and timeframe):
+        return _resultado_simplificado(
+            request, symbol, timeframe, estrategia,
+            titulo="Faltan datos",
+            detalle="elige un símbolo y una temporalidad antes de empezar",
+            exito=False)
+
+    experimento = (f"lab-{symbol}-{timeframe}" if not estrategia
+                   else f"lab-{symbol}-{timeframe}-{estrategia}")
+    return await _encolar_trabajo(
+        request, "pipeline", experimento,
+        {"modo": "investigar", "symbol": symbol, "timeframe": timeframe,
+         "estrategia": estrategia, "workers": workers})
+
+
+@app.get("/investigacion/criterios", response_class=HTMLResponse)
+async def pagina_criterios(request: Request):
+    """Los criterios escritos, en SOLO LECTURA, con su hash y su estado.
+
+    Es la respuesta a «¿en qué metodología se corrió esto?» sin tener que ir a
+    buscar un fichero: cada fila dice el perfil, su estado, el universo que se
+    iba a medir y el hash del contenido.
+    """
+    from starlette.concurrency import run_in_threadpool
+    filas = await run_in_threadpool(_criterios_para_pantalla)
+    return templates.TemplateResponse(
+        request, "criterios.html",
+        {"seccion": "investigacion", "criterios": filas,
+         "ahora": _utc_ahora(), "asset_v": _asset_version()})
+
+
+def _criterios_para_pantalla() -> list[dict[str, Any]]:
+    from research.criterios import listar, resumen
+
+    out: list[dict[str, Any]] = []
+    for cid in listar():
+        try:
+            out.append(resumen(cid))
+        except Exception:  # noqa: BLE001 - un criterio roto no tumba la pagina
+            out.append({"criterio_id": cid, "ilegible": True})
+    return out
+
+
+@app.get("/laboratorio/criterio/{criterio_id}", response_class=HTMLResponse)
+async def laboratorio_criterio_leer(request: Request, criterio_id: str):
+    """El documento de auditoría de un criterio, generado desde su JSON."""
+    from research.criterios import leer
+    try:
+        criterio = leer(criterio_id)
+    except Exception as exc:  # noqa: BLE001 - el motivo va a pantalla
+        return _fallido(request, "Criterio", str(exc))
+    return templates.TemplateResponse(
+        request, "_criterio_lectura.html",
+        {"criterio": criterio, "texto": criterio.a_markdown(),
+         "criterio_id": criterio_id, "ahora": _utc_ahora(),
+         "asset_v": _asset_version()})
+
+
+@app.middleware("http")
+async def _retirar_escritura_metodologica(request: Request, call_next):
+    """La puerta unica que cierra las escrituras de metodologia del navegador.
+
+    Un MIDDLEWARE y no una comprobacion en cada endpoint, por una razon que no es
+    estetica: con ocho endpoints habria ocho sitios donde olvidarse uno, y el que
+    se olvide es exactamente el que nadie vuelve a mirar. Aqui no hay ocho
+    sitios: hay uno. Y se aplica al METODO y a la RUTA, asi que anadir un
+    endpoint nuevo con un nombre viejo tampoco la salta.
+
+    Lo que NO se retira con esto, y es lo importante: la linea de comandos
+    (`research/experimento.py`), los trabajos historicos ya en la cola, y los
+    runners que siguen haciendo falta para unos y otros. Lo que se cierra es la
+    puerta del navegador, no la capacidad.
+
+    La respuesta es 410 Gone y no 403: «no tienes permiso» implicaria que hay
+    alguien que si lo tiene, y aqui no lo hay aunque se tenga el mayor permiso
+    del mundo. 410 dice «esto se retiro», que es lo que paso.
+
+    Y a una peticion de HTMX el cuerpo vuelve con 200, porque htmx no
+    intercambia respuestas de error y el operador se quedaria mirando una
+    pantalla que no se mueve. Un `curl` -que es la forma de saltarse un
+    formulario- si se lleva el 410, y ese es el que importa.
+    """
+    import guardia as _guardia
+
+    if (request.method == "POST"
+            and request.url.path in _guardia.ENDPOINTS_ESCRITURA_RETIRADOS
+            and not _guardia.avanzado_activo()):
+        cuerpo = _guardia.expediente_viejo(request.url.path)
+        es_htmx = request.headers.get("hx-request") == "true"
+        return HTMLResponse(status_code=200 if es_htmx else 410, content=cuerpo)
+    return await call_next(request)
+
+
 @app.post("/laboratorio/barrer", response_class=HTMLResponse)
 async def laboratorio_barrer(request: Request, symbol: str = Form(""),
                              timeframe: str = Form(""), tramo: str = Form(""),
@@ -3693,20 +3998,11 @@ async def laboratorio_barrer(request: Request, symbol: str = Form(""),
     if not (symbol and timeframe and tramo):
         return _fallido(request, "Barrer", "elige el par y escribe el nombre EXACTO de la "
                                            "ventana de seleccion")
-    ocupado = _experimento_ocupado("barrido")
-    if ocupado:
-        return _fallido(request, "Barrer", ocupado)
-    try:
-        codigo, texto = await _correr_experimento(
-            _cmd_barrer(symbol, timeframe, tramo, workers, limit_bars, solo_regla))
-    finally:
-        _liberar_experimento()
-    if codigo != 0:
-        return _fallido(request, "Barrer", texto)
-    return _hecho(request, "Barrido terminado",
-                  texto or f"el barrido sobre '{tramo}' ha terminado y la ventana queda gastada",
-                  aviso="Que salga bien NO significa que haya ventaja: solo que el barrido se "
-                        "ha ejecutado. Lo que decide de verdad es la confirmacion del paso 6.")
+    return await _encolar_trabajo(
+        request, "barrido", f"barrido-{symbol}-{timeframe}-{tramo}",
+        {"symbol": symbol, "timeframe": timeframe, "tramo": tramo,
+         "workers": workers, "limit_bars": limit_bars,
+         "solo_regla": [solo_regla] if solo_regla else None})
 
 
 @app.post("/laboratorio/confirmar", response_class=HTMLResponse)
@@ -3751,25 +4047,11 @@ async def laboratorio_confirmar(request: Request, symbol: str = Form(""),
                         'los parametros tienen que ser un objeto JSON ({"clave": valor}), '
                         'no una lista ni un numero')
 
-    cmd = [sys.executable, "-m", "research.experimento", "confirmar",
-           "--symbol", symbol, "--timeframe", timeframe, "--tramo", tramo,
-           "--umbral-t", str(umbral_t), "--min-ops", str(min_ops),
-           "--config", json.dumps({"regla": regla, "params": ajustes, "motivo": motivo},
-                                  ensure_ascii=False)]
-    ocupado = _experimento_ocupado("confirmacion")
-    if ocupado:
-        return _fallido(request, "Confirmar", ocupado)
-    try:
-        codigo, texto = await _correr_experimento(cmd)
-    finally:
-        _liberar_experimento()
-    if codigo != 0:
-        return _fallido(request, "Confirmar", texto)
-    return _hecho(request, "Confirmacion terminada",
-                  texto or f"'{regla}' medida sobre '{tramo}', que queda gastada",
-                  aviso="Aguanta o refutada esta en el registro y en /experimentos. Las dos "
-                        "son resultados utiles: una refutacion con la ventana gastada es lo "
-                        "que hace que la siguiente busqueda sea una busqueda nueva.")
+    return await _encolar_trabajo(
+        request, "confirmacion", f"confirmacion-{symbol}-{timeframe}-{tramo}",
+        {"symbol": symbol, "timeframe": timeframe, "tramo": tramo,
+         "config": [{"regla": regla, "params": ajustes, "motivo": motivo}],
+         "umbral_t": umbral_t, "min_ops": min_ops})
 
 
 def _elegir_mejora(filas: list[dict]) -> dict | None:
@@ -3832,8 +4114,6 @@ async def laboratorio_pipeline(request: Request, symbol: str = Form(""),
     puede escribir sin mirar. Queda dicho en el registro, que es lo que permite
     que alguien lo note mas adelante.
     """
-    import json
-    import sys
     if not (symbol and timeframe and tramo_barrer and tramo_confirmar):
         return _fallido(request, "Pipeline",
                         "elige el par y los nombres de las DOS ventanas")
@@ -3843,62 +4123,16 @@ async def laboratorio_pipeline(request: Request, symbol: str = Form(""),
                         f"confirmacion tienen que ser dos tramos distintos: si fueran el "
                         f"mismo se estaria eligiendo y confirmando sobre lo ya mirado")
 
-    # REGLA 8: el pipeline ocupa el sitio de experimento el tiempo que duren
-    # las DOS fases. Si un barrido o una confirmacion suelta esta corriendo,
-    # se niega aqui; y si el pipeline mismo esta corriendo, un segundo POST
-    # no puede colarse entre sus dos fases.
-    ocupado = _experimento_ocupado("pipeline")
-    if ocupado:
-        return _fallido(request, "Pipeline", ocupado)
-    try:
-        # -- fase 1: barrer la ventana de seleccion --------------------- #
-        codigo, texto = await _correr_experimento(
-            _cmd_barrer(symbol, timeframe, tramo_barrer, workers, limit_bars, solo_regla))
-        if codigo != 0:
-            return _fallido(request, "Pipeline",
-                            f"el barrido ha fallado y '{tramo_barrer}' HA QUEDADO GASTADA:\n{texto}")
-
-        try:
-            filas = _leer_barrido(symbol, timeframe, tramo_barrer)
-        except (OSError, ValueError) as fallo:
-            return _fallido(request, "Pipeline",
-                            f"el barrido ha corrido pero no se ha podido leer su resultado "
-                            f"({fallo}). '{tramo_barrer}' ha quedado gastada igualmente")
-
-        mejor = _elegir_mejora(filas)
-        if mejor is None:
-            return _fallido(request, "Pipeline",
-                            f"el barrido sobre '{tramo_barrer}' no ha dejado ninguna "
-                            f"configuracion. La ventana ha quedado gastada: revisa el tramo, o "
-                            f"declara los ensayos antes de volver a intentarlo")
-
-        # -- fase 2: confirmar la elegida sobre la otra ventana --------- #
-        t_elegido = float(mejor.get("t_b") or 0)
-        config = {"regla": mejor.get("rule", ""), "params": mejor.get("params", {}),
-                  "motivo": f"elegida del barrido sobre {tramo_barrer}: |t|={t_elegido:.2f}"}
-        codigo, texto_conf = await _correr_experimento([
-            sys.executable, "-m", "research.experimento", "confirmar",
-            "--symbol", symbol, "--timeframe", timeframe, "--tramo", tramo_confirmar,
-            "--umbral-t", str(umbral_t), "--min-ops", str(min_ops),
-            "--config", json.dumps(config, ensure_ascii=False)])
-        if codigo != 0:
-            return _fallido(request, "Pipeline",
-                            f"la eleccion se ha hecho ({config['regla']}) pero la confirmacion "
-                            f"sobre '{tramo_confirmar}' ha fallado. LAS DOS VENTANAS HAN "
-                            f"QUEDADO GASTADAS:\n{texto_conf}")
-
-        return _hecho(
-            request, "Pipeline terminado",
-            f"elegida {config['regla']} del barrido (|t|={t_elegido:.2f}) y confirmada sobre "
-            f"'{tramo_confirmar}'.\n{texto_conf[-600:]}",
-            aviso="Las dos ventanas quedan gastadas. Un reintento sobre las mismas es "
-                  "imposible, y no es un fallo: es lo que hace que lo que se mire despues no "
-                  "esté elegido por lo de antes.")
-    finally:
-        # Se libera SIEMPRE: los return de arriba son cinco caminos de salida
-        # y este es el sexto. Sin finally, un barrido que falla dejaria el
-        # sitio ocupado y el laboratorio bloqueado hasta reiniciar el panel.
-        _liberar_experimento()
+    # El pipeline entero corre como UN trabajo: la cola serializa, las DOS
+    # ventanas se gastan dentro del trabajo, y el operador sigue el progreso
+    # en la pagina del trabajo en vez de en una peticion eterna.
+    return await _encolar_trabajo(
+        request, "pipeline", f"pipeline-{symbol}-{timeframe}-{tramo_barrer}",
+        {"modo": "pipeline", "symbol": symbol, "timeframe": timeframe,
+         "tramo_barrer": tramo_barrer, "tramo_confirmar": tramo_confirmar,
+         "workers": workers, "limit_bars": limit_bars,
+         "solo_regla": [solo_regla] if solo_regla else None,
+         "umbral_t": umbral_t, "min_ops": min_ops})
 
 
 @app.post("/laboratorio/probar", response_class=HTMLResponse)
@@ -3927,9 +4161,6 @@ async def laboratorio_probar(request: Request,
 
     REGLA 8: usa el mismo lock que el resto de experimentos.
     """
-    import json as _json
-    import sys as _sys
-
     if not (symbol and timeframe and estrategia):
         return _resultado_simplificado(
             request, symbol, timeframe, estrategia,
@@ -3952,114 +4183,17 @@ async def laboratorio_probar(request: Request,
             titulo="Las fechas no valen",
             detalle=problema, exito=False)
 
-    # REGLA 8: un experimento a la vez.
-    ocupado = _experimento_ocupado("prueba completa")
-    if ocupado:
-        return _resultado_simplificado(
-            request, symbol, timeframe, estrategia,
-            titulo="Ya hay una prueba en marcha",
-            detalle=ocupado, exito=False)
-
-    try:
-        # 1. Declarar las dos ventanas con nombres automáticos.
-        import research.registro as registro
-        reg = registro.cargar()
-        par = registro.par(reg, symbol, timeframe) or {}
-        existentes = {t.get("nombre", "") for t in par.get("tramos", [])}
-        n = 1
-        while f"entrenar-{n}" in existentes or f"examinar-{n}" in existentes:
-            n += 1
-        nombre_entrenar = f"entrenar-{n}"
-        nombre_examinar = f"examinar-{n}"
-
-        registro.declarar_tramo(reg, symbol, timeframe,
-                                nombre=nombre_examinar, tipo="confirmacion",
-                                desde=examinar_desde, hasta=examinar_hasta,
-                                motivo=f"período de examinar: {estrategia} en {symbol} {timeframe}")
-        registro.declarar_tramo(reg, symbol, timeframe,
-                                nombre=nombre_entrenar, tipo="seleccion",
-                                desde=entrenar_desde, hasta=entrenar_hasta,
-                                motivo=f"período de entrenar: {estrategia} en {symbol} {timeframe}")
-        registro.guardar(reg)
-
-        # 2. Barrer sobre el periodo de entrenar.
-        codigo, texto = await _correr_experimento(
-            _cmd_barrer(symbol, timeframe, nombre_entrenar, workers, None, estrategia))
-        if codigo != 0:
-            return _resultado_simplificado(
-                request, symbol, timeframe, estrategia,
-                titulo="El barrido falló",
-                detalle=(f"el periodo de entrenar ({entrenar_desde} a {entrenar_hasta}) "
-                         f"ya queda gastado.\n{texto}"),
-                exito=False)
-
-        # 3. Elegir la mejor configuración del barrido.
-        try:
-            filas = _leer_barrido(symbol, timeframe, nombre_entrenar)
-        except (OSError, ValueError) as fallo:
-            return _resultado_simplificado(
-                request, symbol, timeframe, estrategia,
-                titulo="El barrido corrió pero no se pudo leer",
-                detalle=f"{fallo}. El periodo de entrenar queda gastado igualmente.",
-                exito=False)
-
-        mejor = _elegir_mejora(filas)
-        if mejor is None:
-            return _resultado_simplificado(
-                request, symbol, timeframe, estrategia,
-                titulo="El barrido no encontró ninguna configuración",
-                detalle="el periodo de entrenar queda gastado. Prueba con fechas más largas.",
-                exito=False)
-
-        t_elegido = float(mejor.get("t_b") or 0)
-        config = {"regla": mejor.get("rule", estrategia),
-                  "params": mejor.get("params", {}),
-                  "motivo": f"elegida del barrido sobre {nombre_entrenar}: |t|={t_elegido:.2f}"}
-
-        # 4. Confirmar la elegida sobre el periodo de examinar.
-        codigo, texto_conf = await _correr_experimento([
-            _sys.executable, "-m", "research.experimento", "confirmar",
-            "--symbol", symbol, "--timeframe", timeframe, "--tramo", nombre_examinar,
-            "--umbral-t", "2.0", "--min-ops", "30",
-            "--config", _json.dumps(config, ensure_ascii=False)])
-
-        if codigo != 0:
-            return _resultado_simplificado(
-                request, symbol, timeframe, estrategia,
-                titulo="La confirmación falló",
-                detalle=(f"se probó {config['regla']} y se examinó sobre el periodo "
-                         f"de examinar ({examinar_desde} a {examinar_hasta}), "
-                         f"pero el examen falló:\n{texto_conf}"),
-                exito=False)
-
-        # 5. Leer el veredicto del registro.
-        reg_despues = registro.cargar()
-        par_despues = registro.par(reg_despues, symbol, timeframe) or {}
-        veredicto = ""
-        for r in (par_despues.get("resultados") or []):
-            if r.get("tramo") == nombre_examinar:
-                veredicto = r.get("veredicto") or r.get("resultado") or ""
-                break
-
-        return _resultado_simplificado(
-            request, symbol, timeframe, config["regla"],
-            titulo="Supera el examen" if "confirma" in veredicto.lower()
-                   or "aguanta" in veredicto.lower() else "No supera el examen",
-            detalle=(f"Se probaron {len(filas)} combinaciones sobre el periodo de "
-                     f"entrenar y la mejor ({config['regla']}, |t|={t_elegido:.2f}) "
-                     f"se examinó sobre el periodo de examinar.\n"
-                     f"Veredicto: {veredicto or texto_conf[-400:]}"),
-            exito="confirma" in veredicto.lower() or "aguanta" in veredicto.lower(),
-            params=config.get("params", {}),
-            t_entrenar=t_elegido)
-
-    except Exception as fallo:
-        return _resultado_simplificado(
-            request, symbol, timeframe, estrategia,
-            titulo="La prueba falló",
-            detalle=str(fallo), exito=False)
-    finally:
-        _liberar_experimento()
+    # La prueba completa corre como UN trabajo de la cola (modo "probar"):
+    # declara las ventanas, barre, elige y confirma, con progreso y
+    # cancelacion reales, y sin mantener esta peticion HTTP viva durante
+    # horas. El veredicto aparece en la pagina del trabajo y en el registro.
+    return await _encolar_trabajo(
+        request, "pipeline", f"prueba-{symbol}-{timeframe}",
+        {"modo": "probar", "symbol": symbol, "timeframe": timeframe,
+         "estrategia": estrategia,
+         "examinar_desde": examinar_desde, "examinar_hasta": examinar_hasta,
+         "entrenar_desde": entrenar_desde, "entrenar_hasta": entrenar_hasta,
+         "workers": workers})
 
 
 def _resultado_simplificado(request: Request, symbol: str, timeframe: str,
@@ -4074,6 +4208,156 @@ def _resultado_simplificado(request: Request, symbol: str, timeframe: str,
          "params": params or {}, "t_entrenar": t_entrenar,
          "ahora": _utc_ahora(), "asset_v": _asset_version()})
 
+# --------------------------------------------------------------------------- #
+# Trabajos: la cola de investigación (espejo de jobs/store.py).
+#
+# El panel CREA trabajos, PIDE cancelaciones y MIRA; ejecuta el worker
+# (python -m jobs.worker), que es un proceso aparte y desasociado. Nada de
+# esta seccion corre calculo pesado: una peticion HTTP que dura horas es
+# justamente lo que esta arquitectura existe para quitar.
+#
+# El progreso se sigue con HTMX re-pidiendo fragmentos (every 3-5 s). Se
+# eligio polling y no SSE a proposito: la fuente de verdad es la SQLite en
+# WAL, una consulta barata, y un evento que se pierde no deja la pagina
+# media muerta - el siguiente refresco la completa. Simple y verificable.
+# --------------------------------------------------------------------------- #
+
+
+@app.get("/investigacion", response_class=HTMLResponse)
+async def pagina_investigacion(request: Request):
+    """El centro: accesos a laboratorio/entrenamiento/registro/trabajos y el
+    resumen de lo que corre, lo que acabo y lo que necesita atencion."""
+    from starlette.concurrency import run_in_threadpool
+    resumen = await run_in_threadpool(jobs_lector.resumen)
+    return templates.TemplateResponse(
+        request, "investigacion.html",
+        {"seccion": "investigacion", "resumen": resumen,
+         "ahora": _utc_ahora(), "asset_v": _asset_version()})
+
+
+@app.get("/investigacion/resumen", response_class=HTMLResponse)
+async def investigacion_resumen(request: Request):
+    """El resumen del centro, para el refresco solo de esa mitad."""
+    from starlette.concurrency import run_in_threadpool
+    resumen = await run_in_threadpool(jobs_lector.resumen)
+    return templates.TemplateResponse(
+        request, "_investigacion_resumen.html", {"resumen": resumen})
+
+
+@app.get("/trabajos", response_class=HTMLResponse)
+async def pagina_trabajos(request: Request):
+    from starlette.concurrency import run_in_threadpool
+    # Sin cola todavia (worker nunca arrancado) la pagina se pinta igual con
+    # la lista vacia: un SELECT sobre una BD sin tablas es un 500, y una
+    # pagina que explota antes de que exista nada que listar es la peor
+    # primera impresion posible.
+    hay_cola = await run_in_threadpool(jobs_lector.existe_cola)
+    trabajos = (await run_in_threadpool(jobs_lector.listar, limite=100)
+                if hay_cola else [])
+    vivo = await run_in_threadpool(jobs_lector.worker_vivo)
+    return templates.TemplateResponse(
+        request, "trabajos.html",
+        {"seccion": "trabajos", "trabajos": trabajos,
+         "worker_vivo": vivo,
+         "ahora": _utc_ahora(), "asset_v": _asset_version()})
+
+
+@app.get("/trabajos/fragmento", response_class=HTMLResponse)
+async def trabajos_fragmento(request: Request):
+    trabajos = jobs_lector.listar(limite=100) if jobs_lector.existe_cola() else []
+    return templates.TemplateResponse(
+        request, "_trabajos_lista.html", {"trabajos": trabajos})
+
+
+def _trabajo_o_404(job_id: str) -> dict:
+    trabajo = jobs_lector.leer(job_id)
+    if trabajo is None:
+        raise HTTPException(404, f"no hay ningun trabajo {job_id!r}")
+    return trabajo
+
+
+def _contexto_detalle(t: dict) -> dict:
+    eventos = jobs_lector.leer_eventos(t["id"], limite=200) \
+        if jobs_lector.existe_cola() else []
+    resultado = t.get("resultado_json") if isinstance(
+        t.get("resultado_json"), dict) else None
+    artefactos = t.get("artefactos_json") if isinstance(
+        t.get("artefactos_json"), dict) else {}
+    config_resuelta = t.get("config_resuelta_json") if isinstance(
+        t.get("config_resuelta_json"), dict) else None
+    return {"t": t, "eventos": eventos, "resultado": resultado,
+            "artefactos": artefactos, "config_resuelta": config_resuelta}
+
+
+@app.get("/trabajos/{job_id}", response_class=HTMLResponse)
+async def pagina_trabajo(request: Request, job_id: str):
+    t = _trabajo_o_404(job_id)
+    return templates.TemplateResponse(
+        request, "trabajo_detalle.html",
+        {"seccion": "trabajos", **_contexto_detalle(t),
+         "ahora": _utc_ahora(), "asset_v": _asset_version()})
+
+
+@app.get("/trabajos/{job_id}/fragmento", response_class=HTMLResponse)
+async def trabajo_fragmento(request: Request, job_id: str):
+    t = _trabajo_o_404(job_id)
+    return templates.TemplateResponse(
+        request, "_trabajo_detalle.html", _contexto_detalle(t))
+
+
+@app.get("/trabajos/{job_id}/log")
+async def trabajo_log(job_id: str):
+    """El log tecnico del trabajo: lo que dijo el motor (barrido) o el
+    proceso (entrenamiento/walk-forward), y la traza si fallo. Texto plano,
+    cola del final, mismo criterio que el visor de logs."""
+    from fastapi.responses import PlainTextResponse
+    t = _trabajo_o_404(job_id)
+    dir_log = PROJECT_ROOT / "data" / "logs" / "jobs" / t["id"]
+    lineas: list[str] = []
+    for nombre in ("motor.log", "salida.log", "error.txt"):
+        ruta = dir_log / nombre
+        if ruta.exists():
+            try:
+                texto = ruta.read_text(encoding="utf-8", errors="replace")
+            except OSError:
+                continue
+            lineas.append(f"===== {nombre} (final) =====")
+            lineas.extend(texto.splitlines()[-200:])
+    cuerpo = "\n".join(lineas) or ("sin log todavia; el worker escribe "
+                                    "data/logs/jobs/" + t["id"] + "/")
+    return PlainTextResponse(cuerpo, media_type="text/plain; charset=utf-8")
+
+
+@app.post("/trabajos/{job_id}/cancelar", response_class=HTMLResponse)
+async def trabajo_cancelar(request: Request, job_id: str):
+    """La cancelacion COOPERATIVA: se pide, no se mata. El worker para el
+    trabajo en su punto seguro y lo confirma; la respuesta es el detalle
+    actualizado, no una promesa."""
+    _trabajo_o_404(job_id)
+    ok, mensaje = jobs_lector.solicitar_cancelacion(job_id)
+    t = jobs_lector.leer(job_id)
+    contexto = _contexto_detalle(t)  # type: ignore[arg-type]
+    contexto["aviso_cancelacion"] = mensaje if not ok else (
+        "Cancelacion pedida. El trabajo para en su punto seguro (entre "
+        "configuraciones/epocas/ventanas); el estado pasa a cancelled cuando "
+        "el worker lo confirma, y lo que quedo hecho se conserva marcado "
+        "como incompleto.")
+    return templates.TemplateResponse(
+        request, "_trabajo_detalle.html", contexto)
+
+
+@app.post("/trabajos/{job_id}/duplicar", response_class=HTMLResponse)
+async def trabajo_duplicar(request: Request, job_id: str):
+    """Repetir con la MISMA configuracion: un trabajo nuevo con su propio
+    identificador. El original no se toca - repetir no es reescribir la
+    historia del experimento anterior."""
+    _trabajo_o_404(job_id)
+    nuevo = jobs_lector.duplicar(job_id)
+    if nuevo is None:
+        raise HTTPException(409, "no se pudo duplicar el trabajo: su "
+                                 "configuracion ya no se puede leer")
+    return RedirectResponse(f"/trabajos/{nuevo['id']}", status_code=303)
+
 # --------------------------------------------------------------------------- #
 # Logs: la primera sección del panel clásico migrada al stack nuevo.
 #
@@ -6229,6 +6513,38 @@ def _simbolos_de(valor: Any, por_defecto: Optional[List[str]] = None,
     return limpio
 
 
+def _no_dejar_entrar(body: Dict[str, Any], claves: tuple[str, ...],
+                     operacion: str) -> None:
+    """La puerta del entrenamiento y del walk-forward: la ciencia no entra
+    desde la petición.
+
+    Se responde con `ParamError` y no con un 403 porque quien llama a estas
+    funciones es a veces el API (que devuelve 422 con el texto) y a veces la
+    página (que pone el motivo escrito dentro y devuelve 200). Si levantara
+    `HTTPException`, la página se quedaría en blanco justo en el mensaje que el
+    operador necesita leer.
+
+    Y no es un detalle que el runner lo ignore: `jobs/runners/entrenamiento.py`
+    vuelve a validar los rangos con `jobs/config.py`, pero un valor DENTRO del
+    rango no es un error de rango. Lo que se rechaza aquí no está fuera de
+    rango: está fuera de sitio.
+
+    Lo que sí se acepta son las decisiones del usuario —qué símbolos, qué
+    temporalidad— porque esas no son ciencia: son la pregunta.
+    """
+    mandadas = set(claves) | set(jobs_lector.PARAMETROS_CIENTIFICOS)
+    hated = sorted(str(k) for k in body if k in mandadas)
+    if not hated:
+        return
+    raise ParamError(
+        f"la configuracion cientifica del {operacion} no se fija desde el panel: "
+        f"has enviado {', '.join(hated)}. El sistema resuelve una configuracion "
+        f"autorizada (jobs/config.py y la metodologia de config/metodologias/) "
+        f"y aplica sus limites; para cambiar una se declara una metodologia "
+        f"nueva y se ejecuta el trabajo con ella. Lo que si se puede indicar es "
+        f"QUE: los simbolos y la temporalidad.")
+
+
 def _lanzar_entrenamiento(body: Dict[str, Any]) -> ControlResult:
     """Lanza un entrenamiento como proceso desasgado. SIN SABER NADA DE HTTP.
 
@@ -6251,6 +6567,13 @@ def _lanzar_entrenamiento(body: Dict[str, Any]) -> ControlResult:
     servidor: ahi el motivo es real y lo que se quiere es un 500, no un
     mensaje de campo.
     """
+    _no_dejar_entrar(body, ("epochs", "batch_size", "accumulation_steps",
+                            "lookback", "stride", "pt_sl", "vol_target",
+                            "max_holding_bars", "d_model", "n_layers",
+                            "model_size", "compile", "primary_model_threshold",
+                            "label_smoothing", "vol_lookback",
+                            "min_holding_bars", "use_meta_labeling"),
+                     "entrenamiento")
     try:
         epochs = int(body.get("epochs", 10))
         batch_size = int(body.get("batch_size", 16))
@@ -6311,18 +6634,19 @@ def _lanzar_entrenamiento(body: Dict[str, Any]) -> ControlResult:
         raise ParamError(
             "ya hay un entrenamiento en marcha", codigo=409)
 
+    # A LA COLA. El entrenamiento corre como trabajo del worker y no como
+    # hijo del panel: sobrevive a reinicios del panel, se cancela
+    # cooperativamente (fichero-bandera, una vez por epoca) y la cola evita
+    # dos a la vez. El fichero de estado canonico se siembra como siempre
+    # para que la pagina tenga algo truthful ANTES de que el worker reclame.
     TRAINING_STATE_FILE.parent.mkdir(parents=True, exist_ok=True)
-    TRAINING_LOG_FILE.parent.mkdir(parents=True, exist_ok=True)
-
-    # Seeded so the panel has something truthful to render during the seconds
-    # before the trainer's first write. The trainer overwrites it.
     seed = {
         "status": "starting",
         "epoch": 0,
         "total_epochs": epochs,
         "train_loss": None,
         "val_loss": None,
-        "message": "arrancando el proceso de entrenamiento",
+        "message": "en cola: esperando al worker de trabajos",
         "pid": None,
         "started_at": datetime.now(timezone.utc).isoformat(),
         "finished_at": None,
@@ -6330,57 +6654,42 @@ def _lanzar_entrenamiento(body: Dict[str, Any]) -> ControlResult:
     with open(TRAINING_STATE_FILE, "w") as fh:
         json.dump(seed, fh)
 
-    cmd = [
-        str(_bot_python()), str(TRAINING_SCRIPT),
-        "--epochs", str(epochs),
-        "--batch-size", str(batch_size),
-        "--accumulation-steps", str(accumulation_steps),
-        "--lookback", str(lookback),
-        "--stride", str(stride),
-        "--pt-sl", str(pt_sl),
-        "--max-holding-bars", str(max_holding_bars),
-        "--vol-target", str(vol_target),
-        "--symbols", *symbols,
-        *_training_model_size(),
-        "--progress-file", str(TRAINING_STATE_FILE),
-        "--checkpoint-dir", str(TRAINING_CHECKPOINT_DIR),
-    ]
-    if compile_model:
-        cmd.append("--compile")
-
-    # The script by path, not ``-m training.train_universal``: that form only
-    # resolves when the repo root happens to be the working directory, and the
-    # failure is an ImportError in a detached process with no terminal.
+    config_job = {
+        "epochs": epochs, "batch_size": batch_size,
+        "accumulation_steps": accumulation_steps, "lookback": lookback,
+        "stride": stride, "pt_sl": pt_sl, "vol_target": vol_target,
+        "max_holding_bars": max_holding_bars, "symbols": symbols,
+        "compile": compile_model,
+        "progress_file": str(TRAINING_STATE_FILE),
+        "checkpoint_dir": str(TRAINING_CHECKPOINT_DIR),
+        "model_size": list(_training_model_size()),
+    }
+    if not jobs_lector.existe_cola():
+        jobs_lector.lanzar_worker_si_hace_falta()
+        for _ in range(40):
+            if jobs_lector.existe_cola():
+                break
+            time.sleep(0.25)
     try:
-        with open(TRAINING_LOG_FILE, "w") as log_fh:
-            proc = subprocess.Popen(
-                cmd, cwd=str(PROJECT_ROOT),
-                stdin=subprocess.DEVNULL, stdout=log_fh, stderr=subprocess.STDOUT,
-                close_fds=True,
-                creationflags=(
-                    getattr(subprocess, "DETACHED_PROCESS", 0)
-                    | getattr(subprocess, "CREATE_NEW_PROCESS_GROUP", 0)
-                    | getattr(subprocess, "CREATE_NO_WINDOW", 0)
-                ),
-            )
-    except OSError as exc:
+        trabajo = jobs_lector.crear_job(
+            "entrenamiento", f"entrenamiento-{len(symbols)}p-{epochs}e",
+            config_job)
+    except Exception as exc:  # noqa: BLE001 - la cola rota es fallo del panel
         seed.update(status="failed",
-                  message=f"no se pudo arrancar el entrenamiento: {exc}")
+                    message=f"no se pudo encolar el entrenamiento: {exc}")
         with open(TRAINING_STATE_FILE, "w") as fh:
             json.dump(seed, fh)
-        raise ParamError(f"no se pudo arrancar el entrenamiento: {exc}", del_servidor=True)
-
-    seed.update(pid=proc.pid, status="running", message="training process started")
-    with open(TRAINING_STATE_FILE, "w") as fh:
-        json.dump(seed, fh)
+        raise ParamError(f"no se pudo encolar el entrenamiento: {exc}",
+                         del_servidor=True) from exc
+    jobs_lector.lanzar_worker_si_hace_falta()
 
     return ControlResult(
         requested="train", seq=1, requested_at=datetime.now(timezone.utc).isoformat(),
         message=(
-            f"entrenamiento lanzado como pid {proc.pid}: {epochs} épocas sobre "
+            f"entrenamiento en cola como {trabajo['id']}: {epochs} épocas sobre "
             f"{len(symbols)} símbolo(s), {lookback} velas de mirada. El progreso se "
-            f"ve en este panel; la salida, con cualquier error si lo hay, se añade "
-            f"a data/logs/training.log"
+            f"ve en este panel y en /trabajos/{trabajo['id']}; la salida del "
+            f"proceso, en data/logs/jobs/{trabajo['id']}/salida.log"
         ),
     )
 
@@ -6407,8 +6716,9 @@ async def train_model(request: Request):
         body = {}
     if not isinstance(body, dict):
         body = {}
+    from starlette.concurrency import run_in_threadpool
     try:
-        return _lanzar_entrenamiento(body)
+        return await run_in_threadpool(_lanzar_entrenamiento, body)
     except ParamError as exc:
         if exc.del_servidor:
             raise HTTPException(500, exc.mensaje)
@@ -6634,6 +6944,13 @@ def _lanzar_walkforward(body: Dict[str, Any]) -> ControlResult:
     un símbolo que el modelo no vio no da un número malo, da un número que no
     significa nada.
     """
+    # La misma puerta que en el entrenamiento: el número de cortes, el tamaño de
+    # la ventana de entrenamiento y el del test son metodología de validación,
+    # no una preferencia de quien pulsa. Los decide `jobs/config.py` y, si en el
+    # futuro se metodologiza, la metodología versionada; no este formulario.
+    _no_dejar_entrar(body, ("n_splits", "wf_train", "wf_test", "wf_step",
+                            "lookback", "conservative", "model_path"),
+                     "walk-forward")
     # Input validation
     symbols = _simbolos_de(body.get("symbols"), ["EURUSD"])
 
@@ -6655,54 +6972,51 @@ def _lanzar_walkforward(body: Dict[str, Any]) -> ControlResult:
             and _process_alive(estado.get("pid"))):
         raise ParamError("ya hay un walk-forward en marcha", codigo=409)
 
-    # Seed state so the panel has something to show immediately
+    # A LA COLA, igual que el entrenamiento: trabajo del worker, cancelacion
+    # cooperativa entre folds, y el fichero canonico sembrado para que la
+    # pagina muestre algo truthful antes de que el worker reclame.
     seed = {
         "status": "starting",
         "symbols": symbols,
         "lookback": lookback,
         "n_splits": n_splits,
-        "message": "arrancando...",
+        "message": "en cola: esperando al worker de trabajos",
     }
     with open(WALKFORWARD_STATE_FILE, "w") as fh:
         json.dump(seed, fh)
 
-    cmd = [
-        str(_bot_python()), str(WALKFORWARD_SCRIPT),
-        "--symbols", ",".join(symbols),
-        "--timeframe", "H1",
-        "--lookback", str(lookback),
-        "--threshold", str(body.get("threshold", 0.26)),
-        "--progress-file", str(WALKFORWARD_STATE_FILE),
-        "--model-path", str(PROJECT_ROOT / "checkpoints_training" / "best_model.pt"),
-    ]
-
+    config_job = {
+        "symbols": symbols, "lookback": lookback, "n_splits": n_splits,
+        "threshold": float(body.get("threshold", 0.26)),
+        "model_path": str(PROJECT_ROOT / "checkpoints_training" / "best_model.pt"),
+        "progress_file": str(WALKFORWARD_STATE_FILE),
+    }
+    if not jobs_lector.existe_cola():
+        jobs_lector.lanzar_worker_si_hace_falta()
+        for _ in range(40):
+            if jobs_lector.existe_cola():
+                break
+            time.sleep(0.25)
     try:
-        with open(WALKFORWARD_LOG_FILE, "w") as log_fh:
-            proc = subprocess.Popen(
-                cmd, cwd=str(PROJECT_ROOT),
-                stdin=subprocess.DEVNULL, stdout=log_fh, stderr=subprocess.STDOUT,
-                close_fds=True,
-                creationflags=(
-                    getattr(subprocess, "DETACHED_PROCESS", 0)
-                    | getattr(subprocess, "CREATE_NEW_PROCESS_GROUP", 0)
-                    | getattr(subprocess, "CREATE_NO_WINDOW", 0)
-                ),
-            )
-    except OSError as exc:
+        trabajo = jobs_lector.crear_job(
+            "walkforward", f"walkforward-{'-'.join(symbols)}", config_job)
+    except Exception as exc:  # noqa: BLE001 - la cola rota es fallo del panel
         seed.update(status="failed",
-                  message=f"no se pudo arrancar el walk-forward: {exc}")
+                    message=f"no se pudo encolar el walk-forward: {exc}")
         with open(WALKFORWARD_STATE_FILE, "w") as fh:
             json.dump(seed, fh)
-        raise ParamError(f"no se pudo arrancar el walk-forward: {exc}",
-                          del_servidor=True)
+        raise ParamError(f"no se pudo encolar el walk-forward: {exc}",
+                          del_servidor=True) from exc
+    jobs_lector.lanzar_worker_si_hace_falta()
 
-    logger.info(f"Walk-forward evaluation launched as pid {proc.pid}")
+    logger.info("Walk-forward encolado como %s", trabajo["id"])
     return ControlResult(
         requested="walkforward",
         seq=1,
         requested_at=datetime.now(timezone.utc).isoformat(),
-        message=(f"walk-forward lanzado como pid {proc.pid}. El progreso se ve en "
-                 f"este panel; el log en data/logs/walkforward.log"),
+        message=(f"walk-forward en cola como {trabajo['id']}. El progreso se ve en "
+                 f"este panel y en /trabajos/{trabajo['id']}; el log del "
+                 f"proceso, en data/logs/jobs/{trabajo['id']}/salida.log"),
     )
 
 
@@ -6717,8 +7031,9 @@ async def start_walkforward(body: Dict = Body(...)):
     ES UN SHELL. La lógica entera vive en :func:`_lanzar_walkforward`, que la
     página de Entrenamiento también llama. Contrato por HTTP sin cambios.
     """
+    from starlette.concurrency import run_in_threadpool
     try:
-        return _lanzar_walkforward(body)
+        return await run_in_threadpool(_lanzar_walkforward, body)
     except ParamError as exc:
         if exc.del_servidor:
             raise HTTPException(500, exc.mensaje)
@@ -7362,8 +7677,13 @@ async def entrenamiento_lanzar(request: Request, clave: str):
 
     mensaje = ""
     error = False
+    # La puerta (encolar) toca disco y procesos: tasklist para ver si el
+    # worker vive, un Popen para arrancarlo, y la primera vez hasta 10 s
+    # esperando a que la cola exista. Todo eso en un hilo del pool, que en
+    # el bucle de eventos es el panel muerto que la valla mide.
+    from starlette.concurrency import run_in_threadpool
     try:
-        resultado = PUERTAS_DE_ACCION[clave](cuerpo)
+        resultado = await run_in_threadpool(PUERTAS_DE_ACCION[clave], cuerpo)
         mensaje = resultado.message
     except ParamError as exc:
         error = not exc.del_servidor
```

---

## `research/experimento.py`

Criterio y trazabilidad (mios) + ganchos de progreso y cancelacion (andamiaje de jobs/, otra sesion).

**27 hunks** -> 6 solo mios, 15 solo de `jobs/`, 6 de los dos.

### Solo de esta sesion (criterio, metodologia)

- `@@ -386,0 +422,4 @@ def _cmd_barrer(args: argparse.Namespace) -> int:`
- `@@ -398 +442 @@ def _cmd_barrer(args: argparse.Namespace) -> int:`
- `@@ -404,0 +449,4 @@ def _cmd_barrer(args: argparse.Namespace) -> int:`
- `@@ -451,0 +506,5 @@ def _cmd_barrer(args: argparse.Namespace) -> int:`
- `@@ -569,0 +737,5 @@ def _cmd_barrer(args: argparse.Namespace) -> int:`
- `@@ -690,0 +953,2 @@ def _cmd_confirmar(args: argparse.Namespace) -> int:`

### Solo de la integracion de jobs/

- `@@ -366,0 +367,11 @@ def _cmd_barrer(args: argparse.Namespace) -> int:`
- `@@ -373,0 +385,8 @@ def _cmd_barrer(args: argparse.Namespace) -> int:`
- `@@ -375,0 +395 @@ def _cmd_barrer(args: argparse.Namespace) -> int:`
- `@@ -384,0 +405,15 @@ def _cmd_barrer(args: argparse.Namespace) -> int:`
- `@@ -389,0 +429,5 @@ def _cmd_barrer(args: argparse.Namespace) -> int:`
- `@@ -434,0 +483,6 @@ def _cmd_barrer(args: argparse.Namespace) -> int:`
- `@@ -536,0 +601,4 @@ def _cmd_barrer(args: argparse.Namespace) -> int:`
- `@@ -542,0 +611,20 @@ def _cmd_barrer(args: argparse.Namespace) -> int:`
- `@@ -556 +711,13 @@ def _cmd_barrer(args: argparse.Namespace) -> int:`
- `@@ -621,0 +800,6 @@ def _cmd_confirmar(args: argparse.Namespace) -> int:`
- `@@ -626,0 +811,5 @@ def _cmd_confirmar(args: argparse.Namespace) -> int:`
- `@@ -629,0 +819 @@ def _cmd_confirmar(args: argparse.Namespace) -> int:`
- `@@ -631,0 +822,13 @@ def _cmd_confirmar(args: argparse.Namespace) -> int:`
- `@@ -636,0 +840,5 @@ def _cmd_confirmar(args: argparse.Namespace) -> int:`
- `@@ -674,0 +883,2 @@ def _cmd_confirmar(args: argparse.Namespace) -> int:`

### De los dos: hay que mirarlos a ojo

- `@@ -500,0 +560,5 @@ def _cmd_barrer(args: argparse.Namespace) -> int:`
- `@@ -544,10 +632,77 @@ def _cmd_barrer(args: argparse.Namespace) -> int:`
- `@@ -617,0 +790,6 @@ def _cmd_barrer(args: argparse.Namespace) -> int:`
- `@@ -678,0 +889,4 @@ def _cmd_confirmar(args: argparse.Namespace) -> int:`
- `@@ -680,4 +894,52 @@ def _cmd_confirmar(args: argparse.Namespace) -> int:`
- `@@ -723,0 +988,6 @@ def _cmd_confirmar(args: argparse.Namespace) -> int:`

Diff completo: `diff-research-experimento.py.patch`

```diff
diff --git a/research/experimento.py b/research/experimento.py
index f657c01..e3e4e9c 100644
--- a/research/experimento.py
+++ b/research/experimento.py
@@ -364,6 +364,17 @@ def _requerir_par_prometido(reg: dict[str, Any], args: argparse.Namespace,
 
 
 def _cmd_barrer(args: argparse.Namespace) -> int:
+    """El subcomando de consola: la ejecucion de siempre, sin enganches.
+
+    La logica vive en :func:`ejecutar_barrido` para que el worker la llame
+    con progreso y cancelacion reales sin que la linea de comandos cambie
+    un byte de su salida.
+    """
+    return ejecutar_barrido(args)
+
+
+def ejecutar_barrido(args: argparse.Namespace, *,
+                      progreso: Any = None, cancelado: Any = None) -> int:
     """El barrido sobre un tramo de seleccion: puerta, pin, gasto, busqueda.
 
     El orden no es decorativo. Todo lo barato primero (validar, recortar,
@@ -371,8 +382,17 @@ def _cmd_barrer(args: argparse.Namespace) -> int:
     los ensayos, GASTAR el tramo y guardar. Despues la busqueda, que es lo
     caro. Si el proceso muere a mitad del barrido, el registro ya dice la
     verdad; si muriera antes, el tramo sigue limpio de verdad.
+
+    `progreso` (evento, carga) y `cancelado` () -> bool son los enganches
+    de la fase D que usa el worker de trabajos; la linea de comandos no los
+    usa y su salida no cambia un byte. Los puntos de cancelacion son: antes
+    de declarar ensayos y gastar el tramo (el tramo queda LIMPIO si se
+    cancela ahi), y entre configuraciones de la busqueda (el tramo ya esta
+    gastado; lo medido se escribe en el JSON MARCADO incompleto, nunca como
+    si estuviera completo).
     """
     from research.sweep import (
+        CancelacionSolicitada,
         _grids,
         barrido_paralelo,
         reality_check_over,
@@ -382,11 +402,35 @@ def _cmd_barrer(args: argparse.Namespace) -> int:
     from research.window import split_csv, verify_window
     from strategies.registry import build_strategy
 
+    def _decir(tipo_evento: str, fase: str | None = None,
+               mensaje: str | None = None, hecho: int | None = None,
+               total: int | None = None, detalle: dict[str, Any] | None = None
+               ) -> None:
+        """Un evento de progreso, si hay a quien contarselo. La consola no
+        escucha: sus prints siguen siendo los de siempre."""
+        if progreso is not None:
+            progreso(tipo_evento, {"fase": fase, "mensaje": mensaje,
+                                   "hecho": hecho, "total": total,
+                                   "detalle": detalle})
+
+    def _cancelar_si_piden(mensaje: str) -> None:
+        if cancelado is not None and cancelado():
+            raise CancelacionSolicitada(mensaje)
+
     reg = registro.cargar()
     p, tramo = _requerir_par_prometido(reg, args, tipo_esperado="seleccion")
+    # La trazabilidad del criterio viaja DENTRO del resultado, no al lado: un
+    # JSON de barrido sin el identificador del criterio que lo autorizo es un
+    # numero del que no se puede decir quien lo pidio ni con que metodologia.
+    _criterio = p.get("criterio") or {}
     # El perfil del broker DECLARADO cuyos costos cobra esta evaluacion. Lo
     # declara la linea de comandos (-brokers/<broker>.yaml); no hay adivinanza.
     profile = load_broker(args.broker)
+    _decir("puerta", fase="puerta", mensaje=(
+        f"tramo '{args.tramo}' reclamaido: {tramo['desde']} -> {tramo['hasta']}"
+    ), detalle={"simbolo": args.symbol, "temporalidad": args.timeframe,
+                "tipo": tramo["tipo"], "broker": profile.name})
+    _cancelar_si_piden("cancelado en la puerta, el tramo queda limpio")
 
     if args.limit_bars and args.workers < 2:
         raise registro.RegistroError(
@@ -395,13 +439,17 @@ def _cmd_barrer(args: argparse.Namespace) -> int:
             "Un smoke que mide mas de lo que dice es peor que no hacer smoke; "
             "usa --workers 2, o quita --limit-bars y mide de verdad.")
 
-    grids = _grids()
+    grids = getattr(args, "grids", None) or _grids()
     if args.solo_regla:
         desconocidas = [r for r in args.solo_regla if r not in grids]
         if desconocidas:
             raise registro.RegistroError(
                 f"reglas desconocidas {desconocidas}; las que hay: {sorted(grids)}")
         grids = {r: grids[r] for r in args.solo_regla}
+    if not grids:
+        raise registro.RegistroError(
+            "el criterio pre-inscrito no declara ninguna configuracion que "
+            "medir: hay una promesa sin contenido, que no es una promesa")
     # Validar barato antes de gastar: un typo en una regla no puede quemar un tramo.
     for rule_id, plans in grids.items():
         for params in plans:
@@ -432,6 +480,12 @@ def _cmd_barrer(args: argparse.Namespace) -> int:
         print(f"  CPCV: {cv_config.n_splits} splits, {cv_config.n_test_splits} test groups, "
               f"embargo={cv_config.embargo_pct:.1%}")
         
+        # Punto de cancelacion antes del gasto, igual que el camino 50/50.
+        # El barrido CPCV aun no tiene cancelacion fina dentro de purged_barrer:
+        # parar aqui deja el tramo limpio; parar despues deja el JSON sin
+        # escribir y el tramo gastado (recuperable solo re-declarando).
+        _cancelar_si_piden(
+            "cancelado antes de declarar ensayos (CPCV): el tramo queda limpio")
         # Gasto y ensayos ANTES de la búsqueda
         registro.acumular_ensayos(reg, args.symbol, args.timeframe, n_grid,
                                   f"barrido CPCV de {n_grid} configuraciones sobre '{args.tramo}'")
@@ -449,6 +503,11 @@ def _cmd_barrer(args: argparse.Namespace) -> int:
         out_data.update({
             "symbol": args.symbol, "timeframe": args.timeframe,
             "broker": profile.name,
+            "criterio": {"id": _criterio.get("criterio_id", ""),
+                         "hash": _criterio.get("hash", ""),
+                         "metodologia": _criterio.get("metodologia", {}),
+                         "estado": _criterio.get("metodologia_estado", ""),
+                         "uso": _criterio.get("metodologia_uso", "")},
             "tramo": {"nombre": tramo["nombre"], "desde": tramo["desde"],
                       "hasta": tramo["hasta"], "tipo": tramo["tipo"]},
             "pin": {"archivo": pin.name, "sha256": digest, "velas": len(recorte.bars)},
@@ -498,6 +557,11 @@ def _cmd_barrer(args: argparse.Namespace) -> int:
             from research.reality_check import format_verdict
             print()
             print(format_verdict(rc, symbol=args.symbol, timeframe=args.timeframe))
+        _decir("salida", fase="escritura",
+               mensaje=(f"barrido CPCV terminado: {pasan}/{n_trials} "
+                        f"sobreviven; escrito en {out.name}"),
+               detalle={"n_trials": n_trials, "pasan": pasan,
+                        "salida": out.name})
         return 0
     
     # El camino "VECTORBT" se retiró el 2026-10-08, tras la auditoría del
@@ -534,26 +598,129 @@ def _cmd_barrer(args: argparse.Namespace) -> int:
         print(f"  LIMIT-BARS {args.limit_bars}: comprobacion del camino, NO una "
               "medicion citable; el tramo se gasta IGUAL -se ha mirado-")
 
+    # Punto de cancelacion ANTES del gasto: parar aqui deja el tramo limpio
+    # de verdad y no debe nada a nadie.
+    _cancelar_si_piden(
+        "cancelado antes de declarar ensayos: el tramo queda limpio")
     # Gasto y ensayos ANTES de la busqueda, en el mismo guardado.
     registro.acumular_ensayos(reg, args.symbol, args.timeframe, n_grid,
                               f"barrido de {n_grid} configuraciones sobre '{args.tramo}'")
     registro.gastar_tramo(reg, args.symbol, args.timeframe, args.tramo,
                           f"barrido de {n_grid} configuraciones eligió leyendo este tramo")
     registro.guardar(reg)
+    _decir("gasto", fase="busqueda",
+           mensaje=(f"{n_grid} configuraciones declaradas y tramo "
+                    f"'{args.tramo}' gastado; empieza la busqueda"),
+           hecho=0, total=n_grid)
+
+    # Los enganches de progreso acumulan aqui lo medido, para que una
+    # cancelacion tenga las filas que ya estan sin rebuscarlas.
+    filas_hechas: list[dict[str, Any]] = []
+
+    def _nota_configuracion(regla: str, fila: dict[str, Any],
+                            hecho: int, total: int) -> None:
+        filas_hechas.append({**fila, "rule": regla})
+        _decir("configuracion", fase="busqueda",
+               mensaje=(f"configuracion {hecho}/{total}: {regla} "
+                        f"t_b={fila.get('t_b', 0):.2f}"),
+               hecho=hecho, total=total,
+               detalle={"regla": regla, "params": fila.get("params"),
+                        "t_b": fila.get("t_b"),
+                        "trades_b": fila.get("trades_b"),
+                        "pasa": bool(fila.get("survives_b"))})
 
-    if args.workers > 1:
-        results = barrido_paralelo(grids, workers=args.workers, csv=str(pin),
-                                   train=train, test=test,
-                                   timeframe=args.timeframe,
-                                   limit_bars=args.limit_bars, split=corte,
-                                   profile=profile)
-    else:
-        results = [sweep(rule, plans, train, test, args.timeframe,
-                         profile=profile)
-                   for rule, plans in grids.items()]
+    try:
+        if args.workers > 1:
+            results = barrido_paralelo(grids, workers=args.workers, csv=str(pin),
+                                       train=train, test=test,
+                                       timeframe=args.timeframe,
+                                       limit_bars=args.limit_bars, split=corte,
+                                       profile=profile,
+                                       al_completar=lambda fila, hecho, total:
+                                           _nota_configuracion(fila["rule"], fila,
+                                                               hecho, total),
+                                       cancelado=cancelado)
+        else:
+            results = []
+            for regla, planes in grids.items():
+                _cancelar_si_piden(
+                    "cancelado antes de la regla " + regla)
+
+                def _hook(fila: dict[str, Any], indice: int, total: int,
+                          _regla: str = regla, _planes: list = planes) -> None:
+                    _nota_configuracion(_regla, fila, indice + 1, len(_planes))
+                    _cancelar_si_piden(
+                        "cancelado a mitad de la regla " + _regla)
+
+                results.append(sweep(regla, planes, train, test,
+                                     args.timeframe, profile=profile,
+                                     al_fila=_hook))
+    except CancelacionSolicitada as exc:
+        # El operador paro el barrido. El tramo YA esta gastado, asi que lo
+        # unico honesto es escribir lo medido MARCADO incompleto, anotarlo en
+        # el registro como lo que es, y re-lanzar para que el worker ponga el
+        # trabajo en cancelled. Un JSON incompleto no puede usarse para
+        # elegir configuracion: el panel lo rechaza.
+        parciales = (exc.parcial if isinstance(exc.parcial, list) and exc.parcial
+                     else filas_hechas)
+        out = (registro.PROJECT_ROOT / "data" / "research"
+               / f"barrido-{args.symbol}-{args.timeframe}-{args.tramo}.json")
+        _atomic_write(out, json.dumps({
+            "symbol": args.symbol, "timeframe": args.timeframe,
+            "broker": profile.name,
+            "criterio": {"id": _criterio.get("criterio_id", ""),
+                         "hash": _criterio.get("hash", ""),
+                         "metodologia": _criterio.get("metodologia", {}),
+                         "estado": _criterio.get("metodologia_estado", ""),
+                         "uso": _criterio.get("metodologia_uso", "")},
+            "tramo": {"nombre": tramo["nombre"], "desde": tramo["desde"],
+                      "hasta": tramo["hasta"], "tipo": tramo["tipo"]},
+            "pin": {"archivo": pin.name, "sha256": digest,
+                    "velas": len(recorte.bars)},
+            "corte": corte,
+            "mitades": {"a": len(train.bars), "b": len(test.bars)},
+            "n_declarado": n_grid,
+            "n_completadas": len(parciales),
+            "limit_bars": args.limit_bars,
+            "incompleto": True,
+            "motivo_interrupcion": "cancelado por el operador",
+            "configurations": [
+                {"rule": f["rule"], "params": f["params"],
+                 "trades_b": f["trades_b"], "t_b": f["t_b"],
+                 "survives_b": f["survives_b"],
+                 "trade_returns_b": f.get("trade_returns_b")}
+                for f in parciales],
+            "reality_check": None,
+        }, ensure_ascii=False, indent=2))
+        reglas_vistas = sorted({f["rule"] for f in parciales})
+        registro.anotar_resultado(reg, args.symbol, args.timeframe,
+                                  tipo="barrido",
+                                  veredicto=(f"incompleto (cancelado): "
+                                             f"{len(parciales)}/{n_grid} "
+                                             f"configuraciones"),
+                                  detalle=f"{out.name}; pin {pin.name}",
+                                  regla=", ".join(reglas_vistas),
+                                  params={})
+        registro.guardar(reg)
+        _decir("cancelado", fase="cancelado",
+               mensaje=(f"barrido cancelado con {len(parciales)}/{n_grid} "
+                        f"configuraciones; parcial escrito en {out.name}"))
+        raise
 
     n_trials = universe_size(results)
-    rc = reality_check_over(results)
+    _cancelar_si_piden("cancelado antes de la correccion por pruebas multiples")
+
+    def _progreso_rc(hechas: int, total: int, n_boot: int, fase: str) -> None:
+        _decir("reality_check", fase="reality_check",
+               mensaje=(f"White Reality Check ({fase}): {hechas}/{total} "
+                        f"configuraciones, {n_boot} remuestreos por "
+                        f"configuracion"),
+               hecho=hechas, total=total,
+               detalle={"n_boot": n_boot, "fase": fase})
+
+    _decir("fase", fase="reality_check",
+           mensaje=f"White Reality Check sobre {n_trials} configuraciones")
+    rc = reality_check_over(results, progreso=_progreso_rc)
     pasan = sum(int(r["passing"]) for r in results)
     configurations = [
         {"rule": r["rule"], "params": row["params"],
@@ -567,6 +734,11 @@ def _cmd_barrer(args: argparse.Namespace) -> int:
     _atomic_write(out, json.dumps({
         "symbol": args.symbol, "timeframe": args.timeframe,
         "broker": profile.name,
+        "criterio": {"id": _criterio.get("criterio_id", ""),
+                     "hash": _criterio.get("hash", ""),
+                     "metodologia": _criterio.get("metodologia", {}),
+                     "estado": _criterio.get("metodologia_estado", ""),
+                     "uso": _criterio.get("metodologia_uso", "")},
         "tramo": {"nombre": tramo["nombre"], "desde": tramo["desde"],
                   "hasta": tramo["hasta"], "tipo": tramo["tipo"]},
         "pin": {"archivo": pin.name, "sha256": digest, "velas": len(recorte.bars)},
@@ -615,25 +787,61 @@ def _cmd_barrer(args: argparse.Namespace) -> int:
         from research.reality_check import format_verdict
         print()
         print(format_verdict(rc, symbol=args.symbol, timeframe=args.timeframe))
+    _decir("salida", fase="escritura",
+           mensaje=(f"barrido terminado: {pasan}/{n_trials} sobreviven B; "
+                    f"escrito en {out.name}"),
+           detalle={"p_value": (rc or {}).get("p_value"),
+                    "n_trials": n_trials, "pasan": pasan,
+                    "salida": out.name})
     return 0
 
 
 def _cmd_confirmar(args: argparse.Namespace) -> int:
+    """El subcomando de consola: la ejecucion de siempre, sin enganches."""
+    return ejecutar_confirmar(args)
+
+
+def ejecutar_confirmar(args: argparse.Namespace, *,
+                        progreso: Any = None, cancelado: Any = None) -> int:
     """La confirmacion sobre un tramo de confirmacion: puerta, pin, gasto, medida.
 
     Mide lo YA elegido -no es una busqueda, no declara ensayos nuevos- y el
     liston viene del criterio del par, escrito antes de mirar. Todo lo barato
     primero; el gasto, guardado, antes de medir.
+
+    `progreso` y `cancelado` son los enganches del worker (fase D); la
+    consola no los usa. Puntos de cancelacion: antes de gastar el tramo
+    (queda limpio) y entre configuraciones de la medicion (lo medido se
+    escribe MARCADO incompleto, igual que el barrido).
     """
     from research.confirmacion import medir
     from research.replay import load_recording_from_csv
+    from research.sweep import CancelacionSolicitada
     from strategies.registry import build_strategy
 
+    def _decir(tipo_evento: str, fase: str | None = None,
+               mensaje: str | None = None, hecho: int | None = None,
+               total: int | None = None, detalle: dict[str, Any] | None = None
+               ) -> None:
+        if progreso is not None:
+            progreso(tipo_evento, {"fase": fase, "mensaje": mensaje,
+                                   "hecho": hecho, "total": total,
+                                   "detalle": detalle})
+
+    def _cancelar_si_piden(mensaje: str) -> None:
+        if cancelado is not None and cancelado():
+            raise CancelacionSolicitada(mensaje)
+
     reg = registro.cargar()
     p, tramo = _requerir_par_prometido(reg, args, tipo_esperado="confirmacion")
     # El perfil del broker DECLARADO, igual que en `barrer`: los costos de la
     # medicion son los de su tabla, y la adivinanza no vuelve.
     profile = load_broker(args.broker)
+    _decir("puerta", fase="puerta", mensaje=(
+        f"tramo '{args.tramo}' reclamaido: {tramo['desde']} -> {tramo['hasta']}"
+    ), detalle={"simbolo": args.symbol, "temporalidad": args.timeframe,
+                "tipo": tramo["tipo"], "broker": profile.name})
+    _cancelar_si_piden("cancelado en la puerta, el tramo queda limpio")
 
     configs: list[tuple[str, dict[str, int], str]] = []
     for i, texto in enumerate(args.config, start=1):
@@ -672,15 +880,69 @@ def _cmd_confirmar(args: argparse.Namespace) -> int:
           "escrito en el criterio antes de mirar")
     print(f"  {len(configs)} configuracion(es)")
 
+    _cancelar_si_piden(
+        "cancelado antes de gastar el tramo de confirmacion: queda limpio")
     registro.gastar_tramo(reg, args.symbol, args.timeframe, args.tramo,
                           f"confirmacion de {len(configs)} configuraciones leyo "
                           "este tramo")
     registro.guardar(reg)
+    _decir("gasto", fase="medicion",
+           mensaje=(f"tramo '{args.tramo}' gastado; midiendo "
+                    f"{len(configs)} configuracion(es)"),
+           hecho=0, total=len(configs))
 
-    filas = [medir(pin_rec, rule_id, params, motivo,
-                   umbral_t=args.umbral_t, min_ops=args.min_ops,
-                   profile=profile)
-             for rule_id, params, motivo in configs]
+    filas: list[dict[str, Any]] = []
+    try:
+        for indice, (rule_id, params, motivo) in enumerate(configs, start=1):
+            _cancelar_si_piden(
+                "cancelado antes de medir la configuracion " + rule_id)
+            fila = medir(pin_rec, rule_id, params, motivo,
+                         umbral_t=args.umbral_t, min_ops=args.min_ops,
+                         profile=profile)
+            filas.append(fila)
+            _decir("configuracion", fase="medicion",
+                   mensaje=(f"medida {indice}/{len(configs)}: {rule_id} "
+                            f"|t|={fila['t_stat']:.2f}, {fila['trades']} ops"),
+                   hecho=indice, total=len(configs),
+                   detalle={"regla": rule_id, "params": params,
+                            "t_stat": fila["t_stat"],
+                            "trades": fila["trades"],
+                            "veredicto": fila["veredicto"]})
+    except CancelacionSolicitada:
+        # Lo medido se escribe MARCADO incompleto. El tramo ya esta gastado y
+        # eso no se des-hace; un veredicto de confirmacion sobre datos a
+        # medias no es ni aguanta ni refutada: es incompleto.
+        out = (registro.PROJECT_ROOT / "data" / "research"
+               / f"confirmacion-{args.symbol}-{args.timeframe}-{args.tramo}.json")
+        _atomic_write(out, json.dumps({
+            "symbol": args.symbol, "timeframe": args.timeframe,
+            "broker": profile.name,
+            "criterio": {"id": (p.get("criterio") or {}).get("criterio_id", ""),
+                         "hash": (p.get("criterio") or {}).get("hash", "")},
+            "tramo": {"nombre": tramo["nombre"], "desde": tramo["desde"],
+                      "hasta": tramo["hasta"], "tipo": tramo["tipo"]},
+            "pin": {"archivo": pin.name, "sha256": digest,
+                    "velas": len(pin_rec.bars)},
+            "liston": {"umbral_abs_t": args.umbral_t,
+                       "min_operaciones": args.min_ops},
+            "incompleto": True,
+            "motivo_interrupcion": "cancelado por el operador",
+            "n_completadas": len(filas),
+            "filas": filas,
+            "veredicto": {"resultado": "incompleto",
+                          "porque": "cancelado por el operador"},
+        }, ensure_ascii=False, indent=2))
+        registro.anotar_resultado(reg, args.symbol, args.timeframe,
+                                  tipo="confirmacion",
+                                  veredicto="incompleto (cancelado)",
+                                  detalle=out.name,
+                                  regla=configs[0][0] if configs else "",
+                                  params=configs[0][1] if configs else {})
+        registro.guardar(reg)
+        _decir("cancelado", fase="cancelado",
+               mensaje=(f"confirmacion cancelada con {len(filas)}/{len(configs)} "
+                        f"medidas; parcial en {out.name}"))
+        raise
     global_ = _veredicto_confirmacion(filas)
 
     out = (registro.PROJECT_ROOT / "data" / "research"
@@ -688,6 +950,8 @@ def _cmd_confirmar(args: argparse.Namespace) -> int:
     _atomic_write(out, json.dumps({
         "symbol": args.symbol, "timeframe": args.timeframe,
         "broker": profile.name,
+        "criterio": {"id": (p.get("criterio") or {}).get("criterio_id", ""),
+                     "hash": (p.get("criterio") or {}).get("hash", "")},
         "tramo": {"nombre": tramo["nombre"], "desde": tramo["desde"],
                   "hasta": tramo["hasta"], "tipo": tramo["tipo"]},
         "pin": {"archivo": pin.name, "sha256": digest, "velas": len(pin_rec.bars)},
@@ -721,6 +985,12 @@ def _cmd_confirmar(args: argparse.Namespace) -> int:
     print(f"  VEREDICTO: {global_['resultado'].upper()}")
     print(f"  {global_['porque']}")
     print(f"\n  escrito en {out}")
+    _decir("salida", fase="escritura",
+           mensaje=(f"confirmacion terminada: {global_['resultado']}; "
+                    f"escrito en {out.name}"),
+           detalle={"resultado": global_["resultado"],
+                    "porque": global_.get("porque"),
+                    "salida": out.name})
     return 0
```

---
