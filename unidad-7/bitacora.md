# Unidad 7

<details>
<summary>Bitácora de proceso de aprendizaje</summary>

  ### Actividad 1 

¿Qué diferencia hay entre un evento musical y un mensaje de control?
 - Un evento musical es algo que ocurre en un instante específico del tiempo.
Por ejemplo, un golpe de sonido generado por Strudel tiene un timestamp y debe ejecutarse exactamente en ese momento para mantener sincronización.
En cambio, un mensaje de control (OSC) no representa un evento puntual, sino un cambio de estado.
 
¿Qué quiere decir que un parámetro del sistema sea persistente?
 - Un parámetro persistente es una variable del sistema que mantiene su valor en el tiempo, no depende de un instante específico, solo cambia cuando llega un nuevo mensaje que lo actualiza.

¿Qué partes del sistema de la unidad 6 permanecen intactas en este nuevo caso?
 - Se conservan: bridgeServer.js, comunicación por WebSocket, concepto de adapters, separación entre entrada de datos; procesamiento; renderizado.

#### Paso 1
Si Open Stage Control fuera “el dispositivo” de esta unidad, ¿Cuál sería su protocolo?
 - Si Open Stage Control fuera el “dispositivo”, su protocolo sería: OSC (Open Sound Control) sobre UDP.
 
¿Qué parte de ese protocolo te interesa conservar y cuál te gustaría normalizar?
  - Se conservan:
  - address → identifica el parámetro (ej: /rgb_1).
  - args → valores enviados (ej: [255,120,30]).

  - Se normaliza:
  - Convertir nombres (/rgb_1) a algo más claro.
  - Validar rangos (0–255).
  - Estandarizar formato de datos.

#### Paso 2
¿Por qué no conviene procesar un mensaje OSC igual que un mensaje de Strudel?
 - Si tratas OSC como evento: el color cambiaría solo un instante y se perdería el estado persistente.
 - Si tratas Strudel como estado: se pierde sincronización temporal y el sistema deja de ser rítmico.
 
¿Qué variables del sistema deberían vivir como estado persistente y no como evento efímero?
 - Persistentes: Colores (rgb), escala de objetos, velocidad de animación, intensidad de efectos y opacidad.

#### Paso 3
¿Qué componentes de la arquitectura necesitas conservar obligatoriamente?
 - Para no romper la arquitectura: Adapter por cada fuente de datos, bridgeServer.js como intermediario (sin lógica de dominio), WebSocket como canal de comunicación.
 - Separación de responsabilidades: input (OSC / Strudel), transformación (adapter), transporte (bridge), estado (app), render (visualización).
 
¿Qué nuevas estructuras de estado necesitas introducir para soportar control paramétrico?
 - Antes: Sistema basado en eventos.
 - Ahora: Sistema híbrido; eventos (Strudel) y estado persistente (OSC).

</details>

<details>
<summary>Bitácora de aplicación</summary>

Cómo configuraste Open Stage Control;
- Abrí Open Stage Control y cargué el archivo OSCUI.json, que es donde están los controles que voy a usar. Luego configuré para que enviara los mensajes al computador mismo (localhost) por el puerto 9000, que es donde mi sistema está escuchando. No usé conexión TCP porque Open Stage Control funciona enviando mensajes por UDP directamente al bridge.

Qué widgets decidiste usar y por qué; 
- Usé tres controles diferentes: uno RGB para cambiar el color, un slider para cambiar el tamaño de las figuras y un toggle para prender o apagar las visuales. Los elegí porque cada uno controla algo distinto y eso ayuda a representar mejor la idea de que los valores se quedan guardados y afectan todo lo que se dibuja.

Qué estructura final de mensaje decidiste usar para los controles;
- Cuando Open Stage Control envía datos, el adapter los convierte en un formato simple con un tipo (osc) y un contenido (payload) que tiene la dirección del control y sus valores. Así, el resto del sistema no tiene que entender cómo funciona OSC por dentro, solo usa esos datos ya organizados.

Cómo conectaste bridgeClient.js, FSMTask, updateLogic y drawRunning; 
- Primero, bridgeClient recibe los mensajes del servidor. Luego esos mensajes pasan a la FSM, que es como una cola que los organiza. Después, la función updateLogic decide qué hacer con cada mensaje (si es de Strudel o de OSC). Finalmente, draw usa ese estado ya procesado para dibujar en pantalla, sin tener que preocuparse por los mensajes de red.

Cómo integraste ambas fuentes de datos en el mismo frontend; 
- Strudel genera eventos con tiempo, como “dispara esto en tal momento”, y esos se guardan en una cola para animarlos después. En cambio, Open Stage Control cambia valores que se quedan activos, como el color o el tamaño. Al final, el dibujo usa ambas cosas: los eventos para saber cuándo animar y los controles para saber cómo se ve.

Qué pruebas hiciste para verificar que el control paramétrico funciona sin romper la sincronización de Strudel;
- Primero probé que Strudel funcionara normal y que las animaciones siguieran sincronizadas. Luego moví el RGB y vi que el color cambiaba en tiempo real. Después moví el slider y cambió el tamaño de las figuras. También probé el toggle para apagar y prender todo. Verifiqué que todo esto no dañara la sincronización de Strudel.

Qué problemas encontraste y cómo los solucionaste.
- Al principio tenía todo mezclado en el socket, lo que hacía difícil controlar el sistema, así que separé la lógica usando updateLogic. También estaba conectando mal el WebSocket, y lo arreglé usando bridgeClient. Otro problema fue que Open Stage Control no enviaba bien los datos, así que ajusté las direcciones (/rgb_1, /size, /toggle) y el puerto. Finalmente, separé bien la diferencia entre eventos (Strudel) y estado persistente (OSC), usando una cola para uno y un objeto de controles para el otro.


</details>

<details>
<summary>Bitácora de reflexión</summary>
</details>
