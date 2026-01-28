# Unidad 1

## Bitácora de proceso de aprendizaje

 ### Actividad 1

#### ¿Qué es un sistema físico interactivo?
> R// Un sistema fisico interactivo es un conjunto de elementos fisicos, un hadware el cual sera interactuado con un usuario y su entorno. El cual responde a las acciones de tal usuario y crea una experiencia dinamica y participativa.
 
#### ¿Cómo podrías aplicar lo que has visto en tu perfil profesional?
> R// Me gustaria usar estos sistemas en algunos proyecros, tal vez en animaciones para un videojuego o una caricatura como tal, ya sea 2D o 3D y tal vez tambien usarlo para un proyecto de realidad virtual.


 ### Actividad 2

#### ¿Qué es el diseño/arte generativo?
> R// El diseño/arte generativo es una serie de reglas que producen resultados visuales o interactivos. En vez de hacer cada cambio o cada proceso manualmente el diseñador puede elegir lo que quiera diseñar, como el proceso, lo y los limites.

#### ¿Cómo podrías aplicar lo que has visto en tu perfil profesional?
> R// Lo usaria tal vez en algún producto, tal vez una experiencia interactiva, pero tal vez cosas más digitales, tal vez en el diseño de una pagina web o una aplicación.

## Bitácora de aplicación 

### Actividad 5
 #### MicroBit
```
from microbit import *

uart.init(baudrate=115200)
display.show(Image.DIAMOND)

while True:
    if button_a.was_pressed():
        uart.write('A')
        sleep(100)
    if button_b.was_pressed():
        uart.write('B')
        sleep(100)
```

 #### p5.js
```
let port;
let connectBtn;
let x = 350;

function setup() {
    createCanvas(400, 400);
    background(220);
    port = createSerial();
    connectBtn = createButton('Connect to micro:bit');
    connectBtn.position(80, 300);
    connectBtn.mousePressed(connectBtnClick);
    let sendBtn = createButton('Send Love');
    sendBtn.position(220, 300);
    sendBtn.mousePressed(sendBtnClick);
    fill('white');
    ellipse(x / 2, height / 2, 100, 100);
}

function draw() {

    if(port.availableBytes() > 0){
        let dataRx = port.read(1) 
      
        if(dataRx == 'A'){
           x += 10
          
        }
        else if(dataRx == 'B'){
            x -= 10
          
        }
        else{
            
        }
        background(220);
        ellipse(x / 2, height / 2, 100, 100);
        fill('#9C27B0');
        text(dataRx, x / 2, height / 2);
    }


    if (!port.opened()) {
        connectBtn.html('Connect to micro:bit');
    }
    else {
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

function sendBtnClick() {
    port.write('h');
}

```

| El sistema fisico interactivo que cree concecta la parte digital y un objeto fisico que es el MicroBit, en el codigo digital de p5.js se crea un circulo donde cada que se presione un boton en el MicroBit, el circulo de p5.js se va moviendo en el eje x cada que se presione el boton A para la derecha y el B para la izquierda.

## Bitácora de reflexión







