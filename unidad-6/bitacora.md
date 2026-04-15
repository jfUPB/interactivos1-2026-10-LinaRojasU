# Unidad 6

## Bitácora de proceso de aprendizaje

### Actividad 1 
¿Cuál es la diferencia entre recibir un mensaje y ejecutarlo?
 > Recibir un mensaje significa que el sistema obtiene los datos enviados desde otra aplicación (por ejemplo, Strudel), pero no implica que el evento ocurra inmediatamente. Ejecutar un mensaje implica interpretarlo y activar una acción en el momento adecuado.

¿Por qué un sistema audiovisual puede necesitar timestamp además de los datos del evento?
 > Un sistema audiovisual necesita sincronización precisa entre sonido y visual.
 > El timestamp permite: ejecutar eventos en el momento correcto, evitar desfases por red o rendimiento y mantener ritmo y coherencia visual.
 > Sin timestamp: las animaciones llegan “tarde” o “desordenadas”
 > Con timestamp: todo ocurre en sincronía con el ritmo musical

¿Qué aspectos de la arquitectura de las unidades 4 y 5 permanecen intactos aunque ahora la fuente de datos ya no sea hardware?
 > Aunque cambia la fuente de datos (ya no hardware), la arquitectura sigue igual: existe una fuente externa de datos, se requiere un adapter, hay una capa de transporte (bridgeServer.js), existe un frontend que interpreta eventos.
 > Lo que cambia: antes eran datos físicos (serial), ahora son eventos musicales con tiempo.
 > Lo que no cambia: separación de responsabilidades, diseño desacoplado, flujo por capas.

### Paso 1
Si Strudel fuera “el dispositivo” de esta unidad, ¿Cuál sería su protocolo?
 > Un formato basado en eventos musicales con estructura tipo: sound (qué sonido), timestamp (cuándo ocurre), cycle (posición rítmica), delta (duración), params (ganancia, banco, etc.).

¿Qué variables mínimas necesitarías extraer para poder construir una visualización útil?
 > Para construir una visual útil necesitaría mínimo: tipo de sonido (s) → define el tipo de animación, timestamp → sincronización, delta → duración de la animación, cycle → posición en el ritmo.
 > Opcionales útiles: gain (intensidad visual), bank (variación estética).

### Paso 2
¿Qué problema resuelve la cola de eventos?
 > La cola permite: guardar eventos antes de ejecutarlos, ordenarlos en el tiempo, evitar ejecución inmediata incorrecta.
 > Sin cola: eventos desordenados, problemas de sincronización.
 > Con cola: ejecución precisa y controlada.

¿Por qué esta capa no pertenece al bridge sino al lado que interpreta el evento?
 > Porque el bridge solo debe transportar datos (responsabilidad única).
 > El scheduling pertenece al frontend porque es quien interpreta el evento, decide cómo y cuándo visualizarlo y es dependiente del contexto visual.

### Paso 3
¿Qué papel cumple el Adapter en U4 y U5?
 > En unidades anteriores el Adapter traducía datos crudos (ASCII o binario), normalizaba el formato y protegía al sistema de cambios externos.
> Era un “traductor” entre el dispositivo → sistema.

¿Qué Adapter necesitas ahora para que los eventos de Strudel no entren “crudos” al sistema visual?
 > Ahora necesitamos que la función del Adapter extraiga datos relevantes, simplifique estructuray cree un contrato claro para el frontend.

## Bitácora de aplicación 

### Cómo configuraste Strudel para emitir eventos;
> Strudel, en su versión REPL online, no emite eventos por WebSocket de forma nativa, sino que genera eventos musicales internamente para su motor de audio.

### Qué estructura final de mensaje decidiste usar;
> Se definió un formato de mensaje normalizado mediante un Adapter en el servidor:
```
{
  "type": "strudel",
  "timestamp": 1710000000000,
  "payload": {
    "eventType": "note",
    "sound": "bd",
    "delta": 0.5
  }
}
```
Este formato permite desacoplar completamente el frontend del formato interno de Strudel (args, address, etc.) y trabajar con un contrato claro y estable.

### Cómo conectaste bridgeClient.js, FSMTask, updateLogic y drawRunning;
> El sistema se conectó respetando la arquitectura por capas:

- Strudel (simulado) envía eventos al puerto 8080
- bridgeServer.js recibe estos eventos y los pasa por StrudelAdapter
- StrudelAdapter normaliza los datos
- El bridge retransmite los eventos al puerto 8081
- bridgeClient.js recibe los eventos y los envía a la FSM
- FSMTask organiza el flujo sin modificar la lógica
- updateLogic almacena los eventos en una cola temporal
- drawRunning se encarga exclusivamente de renderizar

Esto garantiza una separación clara entre datos, lógica temporal y visualización.

### Cómo separaste recepción, cola temporal y renderizado;
> Se implementó una separación clara en tres niveles:

1. Recepción

Ocurre en el WebSocket (onmessage), donde solo se reciben los datos ya normalizados.

2. Cola temporal

Los eventos se almacenan en eventQueue, donde se organizan por timestamp y esperan su momento de ejecución.

3. Renderizado

En el draw() se compara el tiempo actual (Date.now()) con el timestamp de cada evento.
Solo cuando coincide, el evento se convierte en una animación visual.

Esto permite desacoplar completamente el momento de llegada del evento y su ejecución.

### Qué pruebas hiciste para verificar la sincronización;
> Se realizaron varias pruebas:

Ejecución sin timestamp: las animaciones aparecían desincronizadas
Uso de cola con timestamp: las animaciones se alinearon con el ritmo esperado
Eventos consecutivos: se verificó que la cola mantuviera el orden correcto
Ajuste de latencia: se utilizó LATENCY_CORRECTION para compensar posibles retrasos

Estas pruebas confirmaron que la sincronización depende del manejo del tiempo, no del momento de recepción.

### Qué problemas encontraste y cómo los solucionaste.
> 1. Strudel no envía eventos externamente

Problema: No hay salida WebSocket nativa
Solución: Simular el envío de eventos respetando su estructura interna

2. Dependencia del formato crudo (args)

Problema: El frontend estaba acoplado a Strudel
Solución: Implementar un Adapter en el servidor

3. Desfase en la ejecución

Problema: Los eventos se ejecutaban al llegar
Solución: Implementar una cola basada en timestamp

4. Inconsistencia en nombres de sonidos

Problema: Diferencias entre tr909bd y bd
Solución: Normalizar nombres en el Adapter


## Bitácora de reflexión
