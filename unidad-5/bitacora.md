# Unidad 5
## Bitácora de proceso de aprendizaje

### Actividad 1
#### Paso 1
En ACSII se traduce cada caracter
1.67903
ACSII --> "1" "." "6" ... "3": 7 Caracteres
           31  2E  ...     33: 7 Bytes
Se traducen segun lo qque signifique cada caracter en ACSII

En codigo binario, se suele ver mas pequeño. por lo cual consume menos memoria, pero la traduccion del ACSII requiere mas espacio en memoria.

Se tiene que especificar si se envia en big endian o little endian, big siendo con el dato de mas peso primero y el little siendo el dato con menos peso primero.

524 --> 02 0c -> Big endian
524 --> 0c 02 -> Little endian

#### ¿Qué ventajas y desventajas ves en usar un formato binario en lugar de texto ASCII?
Que el binario es mas compacta su traducción que en ACSII, ya que en este ultimo se suele traducir caracter por caracter, su traducción toma más tiempo y al ser tan extenso consume mucho más memoria.

#### Si xValue=500, yValue=524, aState=True, bState=False, ¿cómo se vería el paquete en hexadecimal? (Pista: convierte cada valor según su tipo y anota los bytes en orden.) Respuesta esperada: ```01 F4 02 0C 01 00```

#### Paso 2
#### ¿Por qué el protocolo ASCII de la unidad anterior no tenía este problema de sincronización? (Pista: piensa en qué rol cumplía el carácter \n.)
No se tenia este problema en ASCII ya que el caracter \n nos ayudaba con esto, saltando de linea cuando el segmento de los datos o la trama se repetian.

#### ¿Por qué en binario no podemos usar \n como delimitador?
No se puede usar \n en binario ya que este carácter pertenece a ACSII y no a binario, por lo que el sistema binario lo puede interpretar como un carácter más y no un sincronizador.

#### Paso 3
checksum = sum(data) % 256: suma todos los bytes de datos y lo ajusta a 1 byte (0–255).
packet = b'\xAA' + data + bytes([checksum]): concatena header + datos + checksum.

## Bitácora de aplicación 


## Bitácora de reflexión
