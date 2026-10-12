# RUNBOOK DEMO — Primera orden real

> **Para Jose. Para leer ANTES de tocar nada.**
>
> Este runbook no lo ejecuta la IA. La primera orden real la envías tú, con el
> terminal delante, cuando tú decidas. Nada de lo que hay aquí es automático:
> cada paso es una comprobación que tú haces y tú ves.

---

## Lo que va a pasar

GTS va a enviar **una orden de mercado a una cuenta demo**. No es dinero real.
La orden saldrá por `live/terminal.py`, que es el único módulo del árbol que
puede escribir en el broker, y que **se niega en seco si la cuenta no es demo**.

Si en algún punto no estás seguro: **no continues**. El bot no tiene prisa y
una cuenta demo no la justifica.

---

## Antes de arrancar

### 1. Verifica que la cuenta es DEMO — dos veces

En el terminal MT5, arriba a la derecha, tiene que decir `Demo`.

Y luego, con el bot:

```
myenv\Scripts\python.exe -m app.main --config config/gts.yaml doctor
```

Busca la línea de la cuenta. Tiene que decir `is_demo: true`. **Si dice
`FUNDED`, PARA.** No es una cuenta demo, y el bot se negará igualmente — pero
no llegues ahí: si el terminal dice demo y el doctor dice funded, hay algo que
no entiendes, y ese es el momento de parar, no de seguir.

### 2. Verifica que la cuenta está VACÍA

En el terminal: **Comercio** (o Ctrl+T). No tiene que haber ninguna posición
abierta, ni órdenes pendientes. Ninguna. Ni tuyas de antes.

Si hay posiciones: ciérralas a mano antes, o usa otra cuenta demo. Una cuenta
con posiciones previas hace imposible saber si la posición que aparece es del
bot o tuya de ayer.

### 3. Apunta el magic number

El bot firma todas sus órdenes con un número, el `magic`. Está en
`config/gts.yaml`, bajo `system:`, en `bot_magic` (hoy: `20260928`).

**Apúntalo.** Es lo que distingue una posición del bot de una posición tuya en
el mismo terminal. Cada posición que abra el bot llevará ese número en sus
detalles.

### 4. Comprueba que 'Algo Trading' está ACTIVADO

En el toolbar del MT5 tiene que estar el botón **Algo Trading** encendido
(verde). Si está apagado, toda orden falla con el código 10027 y un mensaje que
no dice nada útil. Enciéndelo ahora, no cuando falle.

### 5. Ten a mano la parada de emergencia

**La parada de emergencia es a mano, no del bot.** Es esto:

1. En el terminal MT5, botón **Algo Trading** → apágalo. El bot ya no puede
   enviar nada más.
2. En una terminal aparte:

   ```
   myenv\Scripts\python.exe -m app.main --config config/gts.yaml run
   ```

   ...y si el bot está corriendo, en el panel (dashboard) → botón **PAUSE**.
   El bot deja de decidir pero no cierra nada.
3. Las posiciones que queden abiertas, **las cierras tú a mano** en el
   terminal, botón derecho → Cerrar posición. Solo las que lleven el magic del
   punto 3.

**Escríbete estos tres pasos en un papel y déjalo al lado del teclado.** En el
momento en que hacen falta, no es el momento de buscarlos.

---

## Arrancar

```
myenv\Scripts\python.exe -m app.main --config config/gts.yaml run
```

El bot arranca, carga la historia, y espera a que cierre una vela. **No envía
nada hasta que su regla no diga algo Y los once validadores digan que sí.**
Puede estar media hora sin hacer nada. Eso es correcto.

Cuando decide, en el journal (`data/journal/decisions.jsonl`) aparece una
entrada con `"profile": "demo"` y, si operó, el ticket en `position`.

---

## Qué mirar en los primeros minutos

- **El terminal, pestaña Comercio**: aparece una posición con tu magic. Solo
  una. Si aparecen dos con el mismo comment (`gts-...`), es una anomalía que el
  sistema detecta pero **que revisas tú**: cierra una de las dos a mano y mira
  el journal.
- **El panel**: estado `ready`, y el contador de órdenes en 1.
- **La cuenta**: el balance se mueve como en una demo. Si ves palancas o
  cantidades que no entiendes, apaga Algo Trading y para.

---

## Cómo abortar

**Inmediato, en este orden:**

1. **Algo Trading → OFF** en el terminal. Corta el envío. Es el paso más
   rápido y el que siempre funciona.
2. Panel → **PAUSE** (o `Ctrl+C` en la consola del bot).
3. Cierra a mano las posiciones con tu magic.

No hay paso 4. Con los tres primeros ya no queda nada vivo.

---

## Si algo no cuadra

- **El bot no abre nada**: mira el journal. Cada barra que no operó dice por qué
  en `fail_reason`. `risk_rejected` con el motivo en `risk.reasons` es la
  respuesta normal las primeras veces.
- **Una orden aparece y desaparece**: puede ser un take-profit o un stop que se
  ejecutó. Está en el journal con su motivo de salida.
- **No sabes qué es**: para. Apaga Algo Trading, abre el journal, y mira la
  última entrada. El journal es la memoria del bot; no hay nada que el bot haya
  hecho que no esté ahí.

---

## Qué NO es este runbook

No es una autorización para LIVE. LIVE no tiene vía: la fábrica de puertos se
niega, y cambiar eso es una decisión que se revisa con este runbook delante, no
un flag que se toca.
