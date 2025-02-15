---
layout: post
title: "[THE BEGINNINGS] Scapy: Construyendo un paquete UDP"
date: 2011-02-07 10:33:26
categories: [network security, python]
tags: [scapy, udp, packet crafting, old]
---


![](/img/udp_ipjkhg.png)

Hace relativamente poco entre en contacto a raíz de un *paper* publicado en [hackxcrack](http://talleres.hackxcrack.es/) con una potentísima aplicación llamada [scapy](http://www.secdev.org/projects/scapy/). Scapy es una herramienta escrita en python que nos permite desde crear paquetes UDP o TCP, hasta realizar un montón de cosas más con la potencia implícita que nos da python, escaneos de puertos, ataques DDoS, como herramienta didactica, etc. Tengo preparado otro post más de scapy y me gustaria poder dedicarle el tiempo que se merece para poder hacer una serie de ellos y poder explicar y entender ciertas cosas. En este primer post vamos a aprender como se genera un paquete TCP y como se vería en Wireshark y si lo recibimos con ncat.

Como sabemos UDP es un protocolo de red en el nivel de transporte que permite el envío de datos sin haber establecido previamente una conexión. Por lo que en este primer post de introducción veremos la facilidad de crear y enviar un paquete.

Las herramientas que yo he usado fueron, un linux con scapy y un mac con ncat y wireshark.

Entramos en materia, lo primero es tener claro cuales son las ip’s origen y destino y el puerto por el que queremos conectarnos.

Ejecutamos Scapy y si hacemos `ls(IP())` vemos lo siguiente:

![](/img/c_ip.jpg)

Son todos los campos de un paquete en la capa IP, vienen definidos de la siguiente forma: `nombreDato : tipoDato = ValorActual`. Para nuestro ejemplo solo nos valdría `dst` (ip destino) y `src` (ip origen). Por lo que tecleamos: (en mi caso)

```
c_IP=IP(dst="47.69.69.22" , src="47.69.69.55")
```

Hemos creado un objeto llamado `c_IP` que contiene los valores por defecto de la capa IP excepto los moficicados (src y dst). Podemos listar la parte modificada de la capa con el comando `c_IP`.

Ahora vamos con la capa UDP, si la listamos `ls(UDP())`  vemos lo siguiente:

![](/img/c_udp.jpg)


Al igual que en el resultado anterior, en este caso podemos ver los campos de un paquete en su capa UDP. Para nuestro caso solo nos valdría `dport` (puerto destino) y `sport` (puerto origen). Por lo que tecleamos: (en mi caso)

```c_UDP=UDP(dport=5005 , sport=1024)```


Hemos creado un objeto llamado c_UDP que contiene los valores por defecto de la capa UDP excepto los modificados (`dport` y `sport`). Podemos listar la parte modificada de la capa con el comando `c_UDP`.

Lo siguiente seria añadir un payload, es decir la información contenida en el paquete, en nuestro caso será un mensaje pero podría ser streaming de video, VoIP o cualquier otro formato que viaje en UDP. Tecleamos:

```
payload="Hola mundo -> UDP
```

Una vez rellenado el payload procedemos a ensamblar el paquete creado de la siguiente forma:

```
pqt=c_IP/c_UDP/payload 		#El paquete recibe el nombre "pqt"
```

Si lanzamos `pqt` podemos ver como quedaría nuestro paquete ensamblado:

![](/img/full_pqt.jpg)

Ahora solo tendríamos que enviarlo, esto se hace con el comando `send(nombrePaquete)` en nuestro caso: `send(pqt)`

Pero antes de enviarlo vamos a abrir nuestro wireshark para ver como se vería. Para poder localizarlo mejor podemos poner el filtro:

`ip.src_host==47.69.69.55 || udp.port ==5005`

Este filtro haría que solo se mostrara los paquetes que salen de la ip `47.69.69.55` (donde ejecutamos scapy) y que vayan destinados al puerto `5005` y mediante UDP.

Aquí tenéis la captura del wireshark y la información del paquete, este paquete es  es rechazado por el host destino porque tiene el puerto cerrado y no espera nada.

![](/img/wireshark.png)

Sin embargo si ejecutamos el ncat de esta forma:

```
ncat -v -u -l -p 1024

-v : modo verbose activo
-u : modo UDP
-l : indicamos que lo ponemos en escucha
-p 1024 : especificamos puerto
```

Estaríamos diciendo con este comando que dejamos el puerto 1024 UDP a la escucha de cualquier conexión entrante y al indicar modo verbo que nos especifique todo, al enviar el paquete con scapy veríamos lo siguiente:

![](/img/ncat.png)

Y aquí os pongo la captura completa de lo hecho con scapy, como veréis, muy sencillo:

![](/img/scapy.png)

Y hasta aquí el post de hoy, espero poder volver a subir la frecuencia de publicación próximamente.
