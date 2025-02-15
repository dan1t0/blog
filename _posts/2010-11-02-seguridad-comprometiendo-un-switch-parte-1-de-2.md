---
layout: post
title: "[THE BEGINNINGS] Seguridad: comprometiendo un switch (Parte 1 de 2)"
date: 2010-11-02 10:30:48
categories: [network security, switching]
tags: [switch security, network compromise, part1, old]
---


![](/img/principal.png)

Cuando nos encontramos con una red no domestica, con varios host conectados a la red entre los que podemos encontrar alguna impresora, equipos finales como PC's o teléfonos VoIP, algún servidor web o de correo; como podría ser la red de una pequeña o mediana empresa tenemos que tener como mínimo un router y un dispositivos intermediario entre el router y los equipos finales, lo más eficiente es encontrarnos con un switch.

Podemos pensar que para qué poner un switch pudiendo poner un hub que es más económico y a nivel de instalación es Plug&Play, y la respuesta es bien sencilla, un hub a nivel informática de red es como un ladrón a nivel de electricidad, es decir, envía por todos los puertos la misma información vaya para el host que este conectado a ese puerto o no, por lo que es el equipo final el encargado de descartar la información que no es para él (o no). Esto nos puede parecer una razón poco poderosa a la hora de decantarnos entre un aparato o otro, pero tenemos que tener en cuenta lo que significa que un hub envía toda la información por todos lo puertos y es que si un usuario malintencionado quisiera saber todo lo que hacemos a través de la red con colocar un simple sniffer podría monitorizar todas las conexiones de la red, con lo que esto supondría para la seguridad de nuestra red, recordemos que aunque en nuestra red solo se conecten empleados tenemos que tener siempre en cuenta que el 80% de los ataques a una red siempre se producen desde dentro de esta.

Hay otras dos razones de porque nunca debemos de poner un hub, una de ellas es por eficiencia, un hub reparte el ancho de banda de la red entre sus puertos haya equipos conectados a estos o no, por lo que si necesitamos conexión para seis equipos y ponemos un hub de ocho puertos, perderemos un 25% del ancho de banda. La otra razón también es por eficiencia de la red, ya que un hub al enviar toda la información a todos los puertos aumentaría la posibilidad de colisión de datos exponencialmente (red tipo bus), lo que haría que el rendimiento de nuestra red cayera en picado.

Dicho todo lo anterior y ya decantados por la opción de instalar un switch tenemos que tener claro que **un switch mal configurado es un agujero de seguridad tan grande como un hub**, expliquemos porque. Como hemos explicado antes un switch es un dispositivo intermedio de capa 2 es decir trabaja con direcciones MAC, dirección MAC destino y dirección MAC origen y de esta forma establece la conexión entre los dos hosts (puertos del switch) correctos, por defecto el switch es un dispositivo que "aprende", es decir nada más encenderlo enviará toda la información por todos los puertos al igual que un hub, pero irá memorizando la MAC del dispositivo que hay conectado a ese puerto de forma que creara una tabla interna en la RAM en la que asociará una dirección MAC a un puerto (o interfaz).

Hasta aquí todo parece sencillo, ahora imaginemos que tenemos una red tipo árbol en la que tenemos varios switch conectados unos a otros, en este caso en la tabla descrita anteriormente nos encontraríamos con varias MAC's asignadas al mismo puerto. En la imagen podemos ver una tabla ARP de un switch, como podemos observar tenemos direcciones de tipo estáticas y dinámicas (lo explicaremos más adelante):

![](/img/tablaarp.png)


En la imagen podemos ver una tabla muy sencilla y con pocas direcciones. Imaginemos ahora por un momento que pasaría si la memoria que tiene destinada el switch a guardar esta información se llenara con muchas MAC's, se produciría un desbordamiento de buffer (buffer overflow) y el switch al ver que no puede acceder a la información de su tabla procedería a comportarse como un hub, es decir empezaría a enviar toda la información por todos los puertos, esto en un switch solo se daría si se esta produciendo un ataque con lo que si no tenemos monitorizada la red, estaremos ante un ataque de robo de información que no podremos detectar facilmente si no somos muy perspicaces (como usuarios finales).

Para protegernos de este tipo de ataques basta con asignar todas las direcciones MAC a un puerto determinado y configurar la tabla de forma estática, claro que en un sitio donde hay movimiento de equipos y de usuarios, es normal que tengamos que tener algunos puertos o dispositivos totalmente configurados de manera dinámica, esto no quita que podamos poner restricciones a los puertos como `n` MAC's diferentes por puerto, o si se detecta que un puerto en el que sabemos que solo podría ir conectado un host empieza a enviar información de varias MAC's en un periodo de tiempo ridículo  tirar la conexión automáticamente. De esta forma nos estaremos quitando un gran porcentaje de ataques de este tipo.

Para comprobar si nuestra red esta a salvo de este tipo de ataques lo mejor es probar atacándola, si tenemos un windows podemos usar [WinArpAttacker](http://www.securityfocus.com/tools/3930), si tenemos un sistema basado en Linux podemos usar [ARPTools](http://www.burghardt.pl/files/arptools-1.0.2.tar.gz), los dos nos ayudaran a llenar la RAM de nuestro switch (ataque ARP-Flood) y comprobar simplemente con WireShark como empezamos a recibir paquetes destinados a otros host de la red. Si queremos ocultar nuestra MAC durante la auditoría, si tenemos un LINUX bastaría con:

```
 ifconfig eth0 down
 ifconfig eth0 hw ether 00:AA:BB:CC:DD:00
 ifconfig eth0 up[/sourcecode]
```

Si tenemos un windows bastaría con usar alguna utilidad como [MacShift](http://devices.natetrue.com/macshift/)

En otra próxima entrega entraremos más a ver como solucionar esto con diferentes métodos según nuestras necesidades.
