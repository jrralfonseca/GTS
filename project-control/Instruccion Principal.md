Utiliza `project-control/GTS_MASTER_SPECIFICATION.md` como especificación principal del proyecto GTS.

Antes de cada tarea, determina qué objetivo de esa especificación estás implementando y cómo se conecta con el sistema existente.

No asumas que una capacidad está implementada porque aparezca documentada. Comprueba el código actual, los puntos de entrada y los consumidores reales.

No confundas una demostración aislada con la ruta operativa del bot. No confundas una señal con una orden, una orden aceptada con una ejecución ni una prueba técnica satisfactoria con rentabilidad.

Prioriza completar el ciclo de vida operativo, mantener la arquitectura coherente y garantizar que el riesgo y los permisos se respeten en todos los caminos de ejecución.

No agregues complejidad que no contribuya a esos objetivos. No conserves artefactos históricos si no son necesarios para la ejecución actual. No habilites capacidades de envío que no estén implementadas y verificadas.

Al terminar cada tarea, informa qué objetivo se completó, qué evidencia lo demuestra y qué queda pendiente. Si detectas que una solicitud contradice esta especificación, señala la contradicción antes de ejecutar cambios destructivos o habilitar un camino de riesgo.

El objetivo no es producir más código, más tests o más documentación. El objetivo es construir y verificar GTS como un sistema operativo de trading completo.