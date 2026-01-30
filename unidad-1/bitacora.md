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

 ### Actividad 4

  > R// El programa no funciona ya que en el mirco.Bit editor estaba escrito "button_a.was_pressed()", esa instruccion no nos funciona ya que a la hora de enviar el mensaje solo lo hace una vez y el programa de p5.js no alcanza a leerlo, en cambio al usar "button_a.is_pressed()" el mensaje se envia constantemente cada que el boton esta siedno presionado, lo que deja que el programa p5.js pueda alcanzar a leerlo.

```

```

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

| El sistema fisico interactivo que crea concectando la parte digital y un objeto fisico que es el MicroBit, en el codigo digital de p5.js se crea un circulo donde cada que se presione un boton en el MicroBit, el circulo de p5.js se va moviendo en el eje x cada que se presione el boton A para la derecha y el B para la izquierda.

## Bitácora de reflexión

 ### Actividad 6

  El programa conecta el MicroBit Editor junto con p5.js, donde el microbit va enviado letras cuando se presiona un boton y el programa de p5.js lee las letras continuamente, y segun la letra que lee cambia de color el cuadrado de el programa p5.js. cuando no se esta presionando ningun boton el cuadrado es de color verde, y cuando se presiona el boton A el cuadrado cambia a color rojo el tiempo que este presionado el boton. Ademas hay un boton en la interfaz que nos permite conectar y desconectar el microbit.











