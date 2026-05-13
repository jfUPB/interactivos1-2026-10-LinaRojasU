# Unidad 8

<details>
<summary>Microbit Editor</summary>
  
```
  from microbit import *

while True:
    x = accelerometer.get_x()
    y = accelerometer.get_y()
    a = button_a.is_pressed()
    b = button_b.is_pressed()

    print("{},{},{},{}".format(x, y, str(a).lower(), str(b).lower()))
    sleep(50)
```
</details>

<details>
<summary>Strudel</summary>

```
setcps(0.5)

const pat = s("bd*4, [- cp]*2, [- hh]*4").bank("tr909")

$: stack(
  pat.gain('1'),
  pat.osc(),
)
```
</details>

<details>
<summary>Bitácora de aplicación</summary>

### Actividad 1
### Un diagrama inicial de la arquitectura;
<img width="595" height="672" alt="image" src="https://github.com/user-attachments/assets/87b9df26-d454-4074-afa7-f44009333fef" />

### Los adapters que vas a usar;
Para integrar las diferentes fuentes de entrada del sistema, decidí trabajar con adapters independientes para cada tecnología, manteniendo la arquitectura.

microbit: MicrobitASCIIAdapter en MicrobitASCIIAdapter.js
Strudel: StrudelAdapter en StrudelAdapter.js
Open Stage Control: OpenStageControlAdapter en OpenStageControlAdapter.js

Todos los adapters se conectan al sistema principal mediante un bridge multi-adapter implementado en bridgeServer.js.

### El contrato de mensajes de cada fuente;
Cada fuente envía información usando un formato normalizado para mantener una comunicación consistente entre backend y frontend.

- micro:bit → Bridge → Frontend
> El micro:bit envía información relacionada con inclinación y botones.
> El parsing serial se realiza en MicrobitASCIIAdapter.js y luego se normaliza en bridgeServer.js.

- Strudel → Bridge → Frontend
> Strudel envía eventos musicales y temporales.
> La normalización de estos mensajes se realiza en StrudelAdapter.js.

- Open Stage Control → Bridge → Frontend
> Open Stage Control envía información relacionada con sliders y controles OSC
> La normalización se realiza en OpenStageControlAdapter.js.

### Pruebas técnicas básicas de integración;
Para verificar que toda la arquitectura funcionara correctamente, realicé pruebas individuales y combinadas de las tres fuentes de entrada.
El backend multi-adapter se ejecutó usando: ```npm run act1``` y el frontend se probó desde indexAct1.html conectando BridgeClient.

#### Verificaciones realizadas:
- microbit: los movimientos y botones modificaban el estado visual correctamente.
- Strudel: los eventos musicales activaban pulsos, ribbons y diferentes efectos visuales sincronizados con el ritmo.
- Open Stage Control: los controles /rgb_1, /size y /toggle modificaban correctamente el color, la escala y algunos comportamientos visuales del render.
La mayor parte de la lógica visual se desarrolló en sketchAct1.js.

### Los errores encontrados y cómo los solucionaste.
Durante el proceso aparecieron varios errores técnicos que tuve que corregir.
- OSC no generaba cambios visuales
> El problema era que el layout utilizaba direcciones OSC como /rgb_1, /size y /toggle, pero el frontend no estaba interpretando correctamente esas direcciones.

- Solución:
> Se corrigió el parseo por address y se agregó el manejo de customColor en sketchAct1.js.

- Explosiones visuales o comportamientos inestables
> Algunos valores OSC llegaban inválidos o demasiado altos, causando deformaciones exageradas en el render.

- Solución:
> Se fortaleció el parseo utilizando toNum() y constrain() para limitar los valores permitidos.

- Pantalla en blanco por error de sintaxis
> El proyecto dejó de renderizar debido a una llave faltante y duplicación de código en ParticleSystem.

- Solución:
> Se reorganizó y corrigió la estructura de sketchAct1.js.

### Actividad 2
### El concepto de la obra;
La propuesta consiste en una experiencia de live coding audiovisual inmersiva donde el ritmo, la melodía y el gesto físico transforman constantemente un paisaje visual ambiental compuesto por ondas, grano, pulsos y trazos dinámicos.
La idea principal es generar una sensación de flujo orgánico donde sonido e interacción corporal afectan directamente el comportamiento visual del sistema.


### El rol de micro:bit, Strudel y Open Stage Control;
micro:bit
> El micro:bit funciona como una herramienta gestual que modifica la dirección y velocidad del flujo visual.
> Su propósito es aportar una sensación física y expresiva dentro de la performance.

Strudel
> Strudel actúa como la estructura temporal de la obra.
> Los drums (bd, cp, hh) activan distintos efectos visuales, mientras que la melodía mantiene una capa ambiental continua.

Open Stage Control
> Open Stage Control permite modificar parámetros persistentes en tiempo real, especialmente color, escala y activación de algunos efectos visuales.

### Las decisiones visuales, musicales y performáticas;
Visuales
> Decidí trabajar una estética inmersiva y ambiental utilizando: ondas, grano visual, ribbons y pulsos dinámicos
> Todo el sistema busca sentirse fluido y orgánico, evitando gravedad o movimientos demasiado rígidos.

Musicales
> Los drums se utilizan para marcar estructura rítmica y generar eventos visibles claros, mientras que las melodías aportan continuidad y atmósfera.

Performáticas
> El micro:bit funciona como una especie de “batuta corporal” que deforma el flujo visual mediante movimiento físico, mientras que Open Stage Control permite intervenir parámetros en vivo durante la ejecución.

### Los cambios realizados entre la iteración ingenieril y la iteración estética;
En la iteración ingenieril el objetivo principal fue lograr una integración funcional mínima entre las tres fuentes y validar el recorrido completo de mensajes dentro de la arquitectura.
Posteriormente, en la iteración estética, el sistema evolucionó hacia una propuesta más inmersiva y performática. Se añadieron efectos visuales diferenciados según el sonido, smoothing para los movimientos del micro:bit y un mapeo más claro de los controles OSC.

### Evidencias de ensayo.
<img width="958" height="453" alt="image" src="https://github.com/user-attachments/assets/a735fca1-c985-4a48-886e-7fee62265d41" />

### Actividad 3

### Un diagrama completo del flujo de datos del sistema.
<img width="595" height="672" alt="image" src="https://github.com/user-attachments/assets/54ce02b0-93ee-4fbb-bbd4-f600b58a6a66" />

### Una tabla con el rol de cada fuente:
<img width="1073" height="431" alt="image" src="https://github.com/user-attachments/assets/9b4d01f1-279a-47b3-a7ad-c273f62b12df" />

### La explicación del recorrido: Adapter -> bridgeServer -> bridgeClient -> FSMTask -> updateLogic -> drawRunning
El sistema sigue la arquitectura propuesta durante el curso para mantener separadas las responsabilidades de entrada, procesamiento y visualización.
Primero, cada fuente de entrada pasa por su propio Adapter:

MicrobitASCIIAdapter.js
StrudelAdapter.js
OpenStageControlAdapter.js

Cada Adapter recibe datos crudos de su tecnología correspondiente y los transforma en mensajes normalizados usando una estructura común basada en type y payload.
Luego, todos los mensajes llegan a bridgeServer.js, cuya función es únicamente distribuir los datos hacia el frontend sin tomar decisiones de lógica ni modificar el comportamiento del sistema.
En el frontend, bridgeClient.js recibe los mensajes y revisa el msg.type para convertirlos en eventos entendibles para la FSM.
Después, FSMTask organiza el sistema y coordina los estados de ejecución.
La lógica principal ocurre en updateLogic, donde se actualizan variables persistentes, eventos temporales y estados visuales según la información recibida desde micro:bit, Strudel y Open Stage Control.
Finalmente, drawRunning utiliza únicamente el estado ya calculado para dibujar la visualización en pantalla, evitando realizar parsing o lógica de red directamente en el render.

### La justificación de la propuesta estética y performática.
La propuesta busca crear una experiencia audiovisual inmersiva basada en la relación entre ritmo, gesto físico y control visual en tiempo real.
Visualmente, el sistema utiliza ondas, grano, ribbons y pulsos para construir un paisaje dinámico y orgánico que reacciona constantemente al sonido y al movimiento.
Musicalmente, Strudel funciona como la estructura temporal principal de la obra. Los drums generan eventos visuales claros y sincronizados, mientras que las melodías mantienen una capa ambiental continua.
Performáticamente, el micro:bit se utiliza como una herramienta gestual que modifica el flujo visual mediante movimiento físico, funcionando casi como una “batuta corporal”. Por otro lado, Open Stage Control permite intervenir parámetros visuales en vivo, especialmente color, escala y activación de efectos.
La intención general es que sonido, gesto y visualidad no funcionen como elementos separados, sino como partes de un mismo sistema interactivo.

### Evidencias de pruebas y ensayos.
<img width="1905" height="854" alt="image" src="https://github.com/user-attachments/assets/41ff08b3-a8fa-478d-8129-be1b0fbda137" />

</details>
