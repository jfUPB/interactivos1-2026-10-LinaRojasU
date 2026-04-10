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


## Bitácora de reflexión
