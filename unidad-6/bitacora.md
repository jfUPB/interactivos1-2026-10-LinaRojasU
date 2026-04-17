# Unidad 6

<details>
<summary>Bitácora de proceso de aprendizaje</summary>

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
</details>

<details>
<summary>Bitácora de aplicación</summary>

### Cómo configuraste Strudel para emitir eventos;
> Strudel no envía eventos por WebSocket de forma automática. Lo que hace es generar eventos internamente para producir sonido, pero esos datos no salen del navegador. Por eso, lo que hice fue tomar esa idea de evento musical (sonido, duración, etc.) y enviarlo manualmente como un mensaje por WebSocket al servidor.

### Qué estructura final de mensaje decidiste usar;
> Decidí usar un formato simple y claro para que el sistema no dependa de cómo funciona Strudel internamente.
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
Este formato es más limpio y fácil de entender, y permite que el frontend trabaje sin tener que interpretar cosas como args o address.

### Cómo conectaste bridgeClient.js, FSMTask, updateLogic y drawRunning;
> El sistema funciona en cadena:

- Strudel envía eventos al puerto 8080.
- El bridge los recibe.
- El Adapter los organiza en un formato claro.
- El bridge los reenvía al frontend (puerto 8081).
- bridgeClient recibe el mensaje.
- La FSM lo dirige a la lógica del sistema.
- updateLogic guarda los eventos en una cola.
- drawRunning se encarga de dibujar.

Cada parte tiene su responsabilidad y no se mezclan entre sí.

### Cómo separaste recepción, cola temporal y renderizado;
> Dividí el proceso en tres partes:

- Recepción: solo recibo el mensaje por WebSocket.
- Cola temporal: guardo los eventos con su timestamp.
- Renderizado: en cada frame comparo el tiempo actual con el timestamp y ejecuto la animación cuando corresponde.

> Esto evita que todo pase apenas llega el mensaje y permite mantener la sincronización.

### Qué pruebas hiciste para verificar la sincronización;
> Primero probé ejecutar los eventos apenas llegaban, y se veía desordenado y fuera de ritmo, luego implementé la cola con timestamp y las animaciones empezaron a coincidir con el ritmo, también probé con muchos eventos seguidos para asegurar que no se perdieran y que el orden se mantuviera y por último, ajusté un pequeño valor de latencia para mejorar la precisión.

### Qué problemas encontraste y cómo los solucionaste.
> Uno de los principales problemas fue que Strudel no envía eventos directamente, así que tuve que simular ese envío, también el frontend estaba dependiendo del formato original de Strudel (args), lo cual no era buena práctica. Eso lo solucioné usando un Adapter en el servidor, otro problema fue que los eventos se ejecutaban apenas llegaban, lo que causaba desfase. Esto lo arreglé usando una cola basada en timestamp y finalmente, tuve diferencias en los nombres de los sonidos, y lo solucioné normalizándolos en el Adapter.

</details>

## Bitácora de reflexión
