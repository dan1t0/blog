---
layout: post
title: "[THE BEGINNINGS] pySIM-Reader: Accediendo a una tarjeta SIM"
date: 2011-07-20 10:58:48
categories: [forensics, mobile]
tags: [pysim, sim card, forensics, old]
---


![](/img/captura_-2011-07-18-a-las-19-20-31.png)

La herramienta de la que vamos a hablar hoy, hace unos años tenía más sentido que ahora, ya que con los nuevos móviles y smartphones la tarjeta SIM ha quedado relegada a un segundo plano, en la actualidad prácticamente solo sirve de interacción con el operador. Ya podréis haceros a la idea de que hablamos de una herramienta que sirve para explorar la tarjeta SIM de un móvil en busca de datos. Para ello utilizaremos el software escrito en Python llamado [pySIM](http://www.ladyada.net/make/simreader/index.html).

Para ello necesitamos primero el hardware, en este caso hay dos posibilidades, una de ellas es la que comenta la web y otra encontrada por mí de pura suerte. El software del que hablamos se conecta a la tarjeta a través de un dispositivo por un puerto serial, pues bien, en la ya conocida web de DealExtreme encontramos un [lector de tarjetas SIM](http://www.dealextreme.com/p/usb-sim-card-reader-6641) por menos de 3€ que para funcionar necesita un driver USB-2-Serial [windows](http://www.prolific.com.tw/eng/downloads.asp?ID=31)
[Mac OS X](https://github.com/failberg/osx-pl2303) y que funciona correctamente con pySIM-Reader. Y [la otra opción](http://www.ladyada.net/make/simreader/solder.html) más divertida y la que en un principio hice yo es comprarlo y montarlo yo mismo, es más caro, pero la satisfacción de hacerlo y que funcione para algunos es más gratificante.

Por otro lado para hacerlo funcionar depende del sistema operativo, en Windows podemos descargar el ejecutable de [aquí](href="http://www.ladyada.net/media/simreader/pySimReader-Serial-Win32-v2.zip), para Linux y Mac OS X necesitamos tener instalado Python, [pySerial](http://pyserial.sourceforge.net/) (librería para interactuar con un puerto serial [COM] y [wxPhyton](http://www.wxpython.org/) (librerías de Python para generar entornos gráfico), además del [código del programa](http://www.ladyada.net/media/simreader/pySimReader-Serial-src-v2.zip). Para que funcione correctamente en Mac OS X os recomiendo MacPort con Python 2.6 y instalar wxPhyton y pySerial desde MacPort también, a mi de otra forma me dio muchos problemas.


Interfaz pySIM

![](/img/01_interfaz.png)

Para hacerlo funcionar puedes conectarlo a través de un conversor [USB-Serial](http://www.adafruit.com/products/18) y al abrir el pySIM-Reader te pedirá que le digas el dispositivo, en Mac OS X y Linux os puede pasar que el dispositivo creado no sea el que aparece en la ventana del programa, mi solución fué hacer un [ln](http://dns.bdat.net/documentos/cursos/ar01s15.html) entre el dispositivo real creado en `/dev/` y el que te reconoce el programa, si os contáis directamente por cable serie puede que no tengáis este problema.


Creando el link al dispositivo requerido:

![](/img/00_linkear_dev.png)

El programa tiene un pero muy grande y de cierto modo es lógico, y es que para extraer los datos de la SIM hay que facilitar el código SIM, por lo que solo podemos extraer información de nuestra SIM o en el caso de que tenga deshabitado el PIN, pero evidentemente al ser el software OpenSource podemos acceder al código y entender a un bajo nivel como accede al chip.


Informacion incial del programa (izquierda) y de la SIM (derecha):

![](/img/002.png)


El software permite entre otras cosas:

* Agenda:
    * Mostrar Agenda
    * Modificar Agenda
    * Mostrar última llamada
    * Hacer/Restaurar backup de Agenda
 
- SMS:
    - Mostrar SMS
    - Hacer/Restaurar backup de SMS

- SIM:
    - Mostrar información de la SIM (no necesita PIN)
    - Cambiar el PIN
    - Habilitar/Deshabilitar el PIN

Petición de código PIN:

![](/img/03_peticion_pin.png)


El programa accede a la SIM de una forma más o menos sencilla de entender, la SIM funciona como una memoria de tamaño fijo (64K por ejemplo) la cual está dividida en sectores o partes y cada parte tiene habilitada varios espacios de memoria fijos en los que se almacena la información. De esta forma el programa, de acuerdo con la RFC de la arquitectura de una tarjeta SIM, sabe que si accede a la posición X,N accede por ejemplo a un registro que guarda un número de teléfono y si accede a X,N+1 accedería a el número de teléfono siguiente almacenado. Podemos decir que está estructurada en una especie de matriz.


pySIM recorriendo la estructura interna de la SIM:

![](/img/04_recorriendo_sim.png)

![](/img/04_recorriendo_sim2.png)

Para poder acceder a esta matriz hay que conocer el PIN, ya que entre los datos y la petición del teléfono o el lector existe un microprocesador que es el que se encarga de comunicar ambos lados.
Este programa con sus limitaciones puede ser interesante si por ejemplo lo usamos para hacer un forense a un móvil de empresa que conocemos el código PIN.

Os enseño una foto del resultado una vez soldado:
![](/img/image.jpeg)
