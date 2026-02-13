# Unidad 2

## Bitácora de proceso de aprendizaje

### Activida 1
 #### ¿Cuáles son los estados en el programa?
  > R// Inicialización: cuando el programa arranca, crea el objeto Pixel, pinta el pixel inicial y empiza el temporizador. 
Bucle: el programa está en su loop principal (cada ~20 ms), esperando y ejecutando la lógica de actualización. Aquí no pasa nada, solo se consulta y actualiza.
Actualizando: cuando en una iteración del bucle se ejecuta update() para cambiar el valor del pixel o la pantalla.
Timeout ocurrido: cuando el Timer llega a su tiempo, genera un evento “Timeout” que se queda esperando a ser procesado.
Procesando eventos: el programa saca los eventos de la cola (como el “Timeout”) y ejecuta las acciones correspondientes (cambiar intensidad, reiniciar timer, etc.).

 #### ¿Cuáles son los eventos en el programa?
  > R// Inicio / creación del pixel: es un evento de arranque que prepara el objeto.
Start del Timer: el temporizador se arranca y programará futuros eventos.
Tick periódico del loop: aunque no es un “evento único”, cada ciclo es un trigger que llama a update()
Timeout del Timer: el temporizador expira y publica/genera un evento post_event("Timeout").
Evento “Timeout” en la cola: el sistema detecta ese evento y lo hace llegar al pixel para que lo procese.

 #### ¿Cuáles son las acciones en el programa?
  > R// set_pixel(0,0,9): se fija el valor (intensidad) del pixel en coordenadas dadas.
start() del Timer: se comienza a contar para que, después de X ms, ocurra un timeout.
update() del Pixel: función que se ejecuta en el bucle para actualizar estado interno.
post_event("Timeout"): el timer publica un evento en la cola cuando expira.
procesar eventos: el programa extrae el evento Timeout y ejecuta la lógica asociada.
dibujar en display: después de actualizar estado se pinta la pantalla con el nuevo valor del pixel.


### Actividad 2

 #### Main.py
```
from microbit import *
import utime
from fsm import Timer,FSMTask,ENTRY

class Semaforo(FSMTask):
    def __init__(self,_x,_y,_timeInRed,_timeInGreen,_timeInYellow):
        super().__init__()
        self.x = _x
        self.y = _y
        self.timeInRed = _timeInRed
        self.timeInGreen = _timeInGreen
        self.timeInYellow = _timeInYellow
        self.myTimer = self.add_timer("Timeout",self.timeInRed)
        self.transition_to(self.estado_waitInRed)

    def clear(self):
            display.set_pixel(self.x,self.y,0)
            display.set_pixel(self.x,self.y+1,0)
            display.set_pixel(self.x,self.y+2,0)

    def estado_waitInRed(self, ev):
        if ev == ENTRY:
            self.clear()
            display.set_pixel(self.x,self.y,9)
            self.myTimer.start(self.timeInRed)
        if ev == "Timeout":
            display.set_pixel(self.x,self.y,0)
            self.transition_to(self.estado_waitInGreen)

    def estado_waitInGreen(self, ev):
        if ev == "ENTRY":
            self.clear()
            display.set_pixel(self.x,self.y+2,9)
            self.myTimer.start(self.timeInGreen)
        if ev == "Timeout":
            display.set_pixel(self.x,self.y+2,0)
            self.transition_to(self.estado_waitInYellow)
        if ev == "A":
            display.set_pixel(self.x,self.y+2,0)
            self.transition_to(self.estado_waitInYellow)

    def estado_waitInYellow(self, ev):
        if ev == ENTRY:
            self.clear()
            display.set_pixel(self.x,self.y+1,9)
            self.myTimer.start(self.timeInYellow)
        if ev == "Timeout":
            display.set_pixel(self.x,self.y+1,0)
            self.transition_to(self.estado_waitInRed)

    

semaforo1= Semaforo(0,0,2000,1000,500)

while True:
    # Imput processing
    if button_a.was_pressed(): semaforo1.post_event("A")
        
    semaforo1.update()
    utime.sleep_ms(20)
```
 #### Fsm.py
```
import utime

ENTRY = "ENTRY"
EXIT  = "EXIT"

class Timer:
    def __init__(self, owner, event_to_post, duration):
        self.owner = owner
        self.event = event_to_post
        self.duration = duration
        self.start_time = 0
        self.active = False

    def start(self, new_duration=None):
        if new_duration is not None:
            self.duration = new_duration
        self.start_time = utime.ticks_ms()
        self.active = True

    def stop(self):
        self.active = False

    def update(self):
        if self.active and utime.ticks_diff(utime.ticks_ms(), self.start_time) >= self.duration:
            self.active = False
            self.owner.post_event(self.event)


class FSMTask:
    def __init__(self):
        self._q = []
        self._timers = []
        self._state = None

    def post_event(self, ev):
        self._q.append(ev)

    def add_timer(self, event, duration):
        t = Timer(self, event, duration)
        self._timers.append(t)
        return t

    def transition_to(self, new_state):
        if self._state:
            self._state(EXIT)
        self._state = new_state
        self._state(ENTRY)

    def update(self):
        for t in self._timers:
            t.update()
        while self._q:
            ev = self._q.pop(0)
            if self._state:
                self._state(ev)
```

### Actividad 4

 #### Maquina de estados 
 ```
@startuml
title Temporizador - UML State Machine

[*] --> CONFIGURACION : Task() (constructor)
CONFIGURACION : entry /\n  display.show(FILL[count])\n  myTimer.stop()

CONFIGURACION  : A [count < max_count] /\n  count += 1\n  display.show(FILL[count])  \n  \n B [count > min_count] /\n  count -= 1\n  display.show(FILL[count])

ARMADO : entry /\n display.show(FILL[self.count])\n self.count -= 1 \n self.myTimer.start()\n Timeout / \nself.transicion_a(self.estado_terminado) \n exit /\n self.myTimer.stop() 
ARMADO --> TERMINADO : Timeout [count > 0] /\n  display.show(FILL[0])
TERMINADO : entry /\n  display.show(Image.SKULL)\n  music.play(music.WAWAWAWAA, wait=False)

TERMINADO ---> CONFIGURACION : A / \n  count = 20\n 
CONFIGURACION ---> ARMADO

@enduml
 ```

<img width="365" height="701" alt="image" src="https://github.com/user-attachments/assets/abf71ab4-3481-49c7-8d1e-e0537f5c5e9c" />


 #### MicroBit

 ```
from microbit import *
import utime
import music

# --- make_fill_images ---
def make_fill_images(on='9', off='0'):
   imgs = []
   for n in range(26):
       rows = []
       k = 0
       for y in range(5):
           row = []
           for x in range(5):
               row.append(on if k < n else off)
               k += 1
           rows.append(''.join(row))
       imgs.append(Image(':'.join(rows)))
   return imgs

FILL = make_fill_images()   # FILL[0] .. FILL[25]

# --- Timer class (tal como se pide) ---
class Timer:
   def __init__(self, owner, event_to_post, duration):
       self.owner = owner
       self.event = event_to_post
       self.duration = duration
       self.start_time = 0
       self.active = False

   def start(self, new_duration=None):
       if new_duration is not None:
           self.duration = new_duration
       self.start_time = utime.ticks_ms()
       self.active = True

   def stop(self):
       self.active = False

   def update(self):
       if self.active:
           if utime.ticks_diff(utime.ticks_ms(), self.start_time) >= self.duration:
               self.active = False
               self.owner.post_event(self.event)

# --- Task con máquina de estados ---
class Task:
   def __init__(self):
       self.event_queue = []
       self.timers = []
       # Timer que genera "Timeout" cada 1000 ms cuando está activo
       self.myTimer = self.createTimer("Timeout", 1000)

       # parámetros del temporizador (pixeles)
       self.count = 20          # valor inicial (20 pixels)
       self.min_count = 15      
       self.max_count = 25      

       # estado actual (maquina de estados)
       self.estado_actual = None
       self.transicion_a(self.estado_configuracion)

   # crea y registra timer
   def createTimer(self, event, duration):
       t = Timer(self, event, duration)
       self.timers.append(t)
       return t

   # agregar evento a la cola
   def post_event(self, ev):
       self.event_queue.append(ev)

   # actualizar timers y procesar cola
   def update(self):
       # 1) actualizar timers (no usar sleep dentro de estados)
       for t in self.timers:
           t.update()

       # 2) procesar eventos en cola
       while len(self.event_queue) > 0:
           ev = self.event_queue.pop(0)
           if self.estado_actual:
               self.estado_actual(ev)

   # transicion de estado con EXIT/ENTRY
   def transicion_a(self, nuevo_estado):
       if self.estado_actual:
           self.estado_actual("EXIT")
       self.estado_actual = nuevo_estado
       self.estado_actual("ENTRY")

   # -------------------------
   # Estado: configuración
   # -------------------------
   def estado_configuracion(self, ev):
       if ev == "ENTRY":
           # mostrar valor actual (cantidad de pixeles encendidos)
           display.show(FILL[self.count])
           # temporizador detenido/en desarmado
           self.myTimer.stop()

       elif ev == "A":
           # aumentar (hasta max_count)
           if self.count < self.max_count:
               self.count += 1
               display.show(FILL[self.count])

       elif ev == "B":
           # disminuir (hasta min_count)
           if self.count > self.min_count:
               self.count -= 1
               display.show(FILL[self.count])

       elif ev == "S":
           # armar: pasar a modo armado (inicia la cuenta regresiva)
           # si queremos armar solo si count entre min y max (ya garantizado)
           self.transicion_a(self.estado_armado)

   # -------------------------
   # Estado: armado / cuenta regresiva
   # -------------------------
   def estado_armado(self, ev):
       if ev == "ENTRY":
           # empezar la cuenta regresiva: arrancar el timer de 1s
           # dejamos la pantalla con el número actual (count)
           display.show(FILL[self.count])
           # arrancamos el Timer para que al cabo de 1s postee "Timeout"
           self.myTimer.start()   # usa duración por defecto 1000 ms

       elif ev == "Timeout":
           # cada vez que llega Timeout, descontamos 1 pixel
           if self.count > 0:
               self.count -= 1
               display.show(FILL[self.count])

           # si quedan pixeles, volver a arrancar timer para el siguiente segundo
           if self.count > 0:
               self.myTimer.start()
           else:
               # llegó a 0 → fin
               self.transicion_a(self.estado_terminado)

       elif ev == "EXIT":
           # al salir del armado detenemos timers
           self.myTimer.stop()

   # -------------------------
   # Estado: terminado (alarma)
   # -------------------------
   def estado_terminado(self, ev):
       if ev == "ENTRY":
           # mostrar calavera y sonar el speaker (no bloqueante)
           try:
               display.show(Image.SKULL)   # muestra calavera
           except:
               # si no existe Image.SKULL en la version, usar Image('...')
               display.show(Image('00900:09090:00900:00000:00900'))  # fallback simbólico

           # sonar (no bloqueante)
           try:
               music.play(music.WAWAWAWAA, wait=False)
           except:
               # fallback: reproducir una nota corta sin bloquear
               try:
                   music.play(['C4:1'], wait=False)
               except:
                   pass

       elif ev == "A":
           # al presionar A volver a configuración y resetear a 20
           self.count = 20
           self.transicion_a(self.estado_configuracion)

# --- Instanciar la tarea ---
task = Task()

# --- Bucle principal (genera eventos de botones y shake) ---
while True:
   # Generar eventos físicos (usar was_pressed() como indica la consigna final)
   if button_a.was_pressed():
       task.post_event("A")
   if button_b.was_pressed():
       task.post_event("B")
   if accelerometer.was_gesture("shake"):
       task.post_event("S")

   # actualizar la máquina (lo actualiza timers + procesa la cola)
   task.update()

   # ciclo principal con pequeña espera (no usamos sleep() dentro de estados)
   utime.sleep_ms(20)
 ```


## Bitácora de aplicación 

 ### Actividad 5

 #### ¿Como lo resolvi?
 > Lo que hice fue escribir el mismo codigo que en la activiad 4, solo que esta recibiendo los mismos eventos si vienen de p5.js. Como por ejemplo inicializar UART para recibir datos de p5.js y por ultimo si llega una de las letras validas hace lo mismo que con los botones fisicos, asi que hace lo mismo sin que haya venido del microbit fisico o de los botones de p5.js.

 #### MicroBit
 ```
from microbit import *
import utime
import music
import sys

# habilitar UART para comunicacion con p5.js (usb serial)
from microbit import uart
uart.init(115200)

# --- make_fill_images ---
def make_fill_images(on='9', off='0'):
    imgs = []
    for n in range(26):
        rows = []
        k = 0
        for y in range(5):
            row = []
            for x in range(5):
                row.append(on if k < n else off)
                k += 1
            rows.append(''.join(row))
        imgs.append(Image(':'.join(rows)))
    return imgs

FILL = make_fill_images()   # FILL[0] .. FILL[25]

# --- Timer class ---
class Timer:
    def __init__(self, owner, event_to_post, duration):
        self.owner = owner
        self.event = event_to_post
        self.duration = duration
        self.start_time = 0
        self.active = False

    def start(self, new_duration=None):
        if new_duration is not None:
            self.duration = new_duration
        self.start_time = utime.ticks_ms()
        self.active = True

    def stop(self):
        self.active = False

    def update(self):
        if self.active:
            if utime.ticks_diff(utime.ticks_ms(), self.start_time) >= self.duration:
                self.active = False
                self.owner.post_event(self.event)

# --- Task con máquina de estados ---
class Task:
    def __init__(self):
        self.event_queue = []
        self.timers = []
        # Timer que genera "Timeout" cada 1000 ms cuando está activo
        self.myTimer = self.createTimer("Timeout", 1000)

        # parámetros del temporizador (pixeles)
        self.count = 20          # valor inicial (20 pixels)
        self.min_count = 15
        self.max_count = 25

        # estado actual (maquina de estados)
        self.estado_actual = None
        self.transicion_a(self.estado_configuracion)

    # crea y registra timer
    def createTimer(self, event, duration):
        t = Timer(self, event, duration)
        self.timers.append(t)
        return t

    # agregar evento a la cola
    def post_event(self, ev):
        self.event_queue.append(ev)

    # actualizar timers y procesar cola
    def update(self):
        # 1) actualizar timers (no usar sleep dentro de estados)
        for t in self.timers:
            t.update()

        # 2) procesar eventos en cola
        while len(self.event_queue) > 0:
            ev = self.event_queue.pop(0)
            if self.estado_actual:
                self.estado_actual(ev)

    # transicion de estado con EXIT/ENTRY
    def transicion_a(self, nuevo_estado):
        if self.estado_actual:
            self.estado_actual("EXIT")
        self.estado_actual = nuevo_estado
        self.estado_actual("ENTRY")

    # -------------------------
    # Estado: configuración
    # -------------------------
    def estado_configuracion(self, ev):
        if ev == "ENTRY":
            # mostrar valor actual (cantidad de pixeles encendidos)
            display.show(FILL[self.count])
            # temporizador detenido/en desarmado
            self.myTimer.stop()
            # opcional: informar por serial el estado
            try:
                uart.write("STATE:CONFIG\n")
                uart.write("COUNT:%d\n" % self.count)
            except:
                pass

        elif ev == "A":
            # aumentar (hasta max_count)
            if self.count < self.max_count:
                self.count += 1
                display.show(FILL[self.count])
                # opcional: enviar feedback por serial
                try:
                    uart.write("COUNT:%d\n" % self.count)
                except:
                    pass

        elif ev == "B":
            # disminuir (hasta min_count)
            if self.count > self.min_count:
                self.count -= 1
                display.show(FILL[self.count])
                try:
                    uart.write("COUNT:%d\n" % self.count)
                except:
                    pass

        elif ev == "S":
            # armar: pasar a modo armado (inicia la cuenta regresiva)
            self.transicion_a(self.estado_armado)

        elif ev == "EXIT":
            pass

        elif ev == "Timeout":
            # no usamos Timeouts en configuración
            pass

    # -------------------------
    # Estado: armado / cuenta regresiva
    # -------------------------
    def estado_armado(self, ev):
        if ev == "ENTRY":
            # empezar la cuenta regresiva: arrancar el timer de 1s
            display.show(FILL[self.count])
            self.myTimer.start()   # usa duración por defecto 1000 ms
            try:
                uart.write("STATE:ARMED\n")
                uart.write("COUNT:%d\n" % self.count)
            except:
                pass

        elif ev == "Timeout":
            # cada vez que llega Timeout, descontamos 1 pixel
            if self.count > 0:
                self.count -= 1
                display.show(FILL[self.count])
                try:
                    uart.write("COUNT:%d\n" % self.count)
                except:
                    pass

            # si quedan pixeles, volver a arrancar timer para el siguiente segundo
            if self.count > 0:
                self.myTimer.start()
            else:
                # llegó a 0 → fin
                self.transicion_a(self.estado_terminado)

        elif ev == "A" or ev == "B":
            # durante armado no permitimos cambiar la cuenta (ignorar)
            pass

        elif ev == "S":
            # ignorar
            pass

        elif ev == "EXIT":
            # al salir del armado detenemos timers
            self.myTimer.stop()

    # -------------------------
    # Estado: terminado (alarma)
    # -------------------------
    def estado_terminado(self, ev):
        if ev == "ENTRY":
            # mostrar calavera y sonar el speaker (no bloqueante)
            try:
                display.show(Image.SKULL)
            except:
                display.show(Image('00900:09090:00900:00000:00900'))

            try:
                music.play(music.WAWAWAWAA, wait=False)
            except:
                try:
                    music.play(['C4:1'], wait=False)
                except:
                    pass

            try:
                uart.write("STATE:FINISHED\n")
            except:
                pass

        elif ev == "A":
            # al presionar A volver a configuración y resetear a 20
            self.count = 20
            self.transicion_a(self.estado_configuracion)

        elif ev == "B" or ev == "S":
            # ignorar otros botones en modo terminado
            pass

        elif ev == "EXIT":
            # limpiar pantalla si salimos
            display.clear()

        elif ev == "Timeout":
            # no aplicable
            pass

# --- Instanciar la tarea ---
task = Task()

# --- Bucle principal (genera eventos de botones y shake y lee serial) ---
while True:
    # Generar eventos físicos
    if button_a.was_pressed():
        task.post_event("A")
    if button_b.was_pressed():
        task.post_event("B")
    if accelerometer.was_gesture("shake"):
        task.post_event("S")

    # --- NUEVO: leer serial (datos desde p5.js) ---
    # Si llega una letra A/B/S desde p5.js la encolamos como evento.
    try:
        if uart.any():
            data = uart.read(1)
            if data:
                # data puede ser bytes; normalizamos a str de 1 caract.
                try:
                    ch = data.decode('utf-8')
                except:
                    ch = chr(data[0]) if isinstance(data, (bytes, bytearray)) else str(data)
                ch = ch.strip()
                if ch in ("A", "B", "S"):
                    task.post_event(ch)
    except Exception as e:
        # Si hay problemas con UART no interrumpimos la tarea
        pass

    # actualizar la máquina (timers + procesar cola)
    task.update()

    # espera corta en el loop principal (permitido)
    utime.sleep_ms(20)
 ```

 #### p5.js
 ```
let port;
let connectBtn;
let upBtn, downBtn, armBtn;
let feedbackP;

function setup() {
  createCanvas(400, 200);
  background(240);

  port = createSerial();

  connectBtn = createButton('Connect to micro:bit');
  connectBtn.position(20, 120);
  connectBtn.mousePressed(connectBtnClick);

  upBtn = createButton('UP (A)');
  upBtn.position(150, 20);
  upBtn.mousePressed(() => sendChar('A'));

  downBtn = createButton('DOWN (B)');
  downBtn.position(230, 20);
  downBtn.mousePressed(() => sendChar('B'));

  armBtn = createButton('ARM (S)');
  armBtn.position(150, 60);
  armBtn.mousePressed(() => sendChar('S'));

  feedbackP = createP('Feedback: -');
  feedbackP.position(20, 150);

  textAlign(LEFT, TOP);
}

function draw() {
  background(240);

  fill(0);
  text("Controles remotos para el temporizador (A,B,S)", 20, 5);

  // lectura de feedback del micro:bit (opcional)
  if (port.availableBytes && port.availableBytes() > 0) {
    let data = port.read(1); // lee 1 byte
    if (data) {
      // data puede ser 'C', o parte de "COUNT:..." si micro:bit mandó texto
      feedbackP.html('Feedback: ' + String(data));
    }
  }

  // estado del botón de conexión
  if (!port.opened()) {
    connectBtn.html('Connect to micro:bit');
  } else {
    connectBtn.html('Disconnect');
  }
}

function connectBtnClick() {
  if (!port.opened()) {
    port.open('MicroPython', 115200);
  } else {
    port.close();
  }
}

function sendChar(ch) {
  if (port.opened()) {
    port.write(ch);
  } else {
    console.log('Port not open');
  }
}
 ```


## Bitácora de reflexión




