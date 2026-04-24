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
- 

Qué widgets decidiste usar y por qué; 
- 

Qué estructura final de mensaje decidiste usar para los controles;
- 

Cómo conectaste bridgeClient.js, FSMTask, updateLogic y drawRunning; 
- 

Cómo integraste ambas fuentes de datos en el mismo frontend; 
- 

Qué pruebas hiciste para verificar que el control paramétrico funciona sin romper la sincronización de Strudel;
- 

Qué problemas encontraste y cómo los solucionaste.
- 


</details>

<details>
<summary>Bitácora de reflexión</summary>
</details>
