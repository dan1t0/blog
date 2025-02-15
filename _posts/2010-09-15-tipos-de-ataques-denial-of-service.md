---
layout: post
title: "[THE BEGINNINGS] Tipos de ataques Denial-of-Service"
date: 2010-09-15 12:17:45
categories: [cybersecurity, attack]
tags: [dos, denial-of-service, attack, old]
---


![](/img/dos3.png)

Un ataque de denegación de servicio (D.o.S.) es un tipo practica bastante común en el mundo de Internet, y se basa en hacer que un servicio o recurso sea inaccesible para los usuarios del mismo mientras dura el ataque, este tipo de ataques suele usarse a veces como distracción de los administradores de red para realizar un ataque más potente con un objetivo más concreto o simplemente dejar cortado un servicio en un momento vital para la empresa. Por lo tanto es bueno conocer qué es, que tipos hay y así poder tener un idea clara de como defenderse de ellos. No solo desde el exterior de la red, sino desde el interior que es donde se produce la mayoría de los ataques actualmente (80%).

* **Ataques de desbordamiento de buffer**. Es de los más comunes, se basa en enviar más trafico a una aplicación o dispositivo del que este puede soportar y de esta forma lo colapse. Imaginemos que tenemos un servidor de correo que puede procesar 5 correos por segundo si, desde varias maquinas se envían 20 correos por segundo al servidor de correo este colapsara y quedara inutilizado durante el tiempo que dure el ataque.

* **Ataque SYN**. Para entender este ataque hay que conocer un poco del protocolo TCP, para abrir una conexión entre dos hosts se necesita una sincronización previa entre ambos, para ello el host que quiere conectarse (A) tiene que enviar un paquete de tipo SYN al equipo destino (B), este lo que hace es asignar un espacio en la pila TCP para esa conexión y envía a B  una respuesta, si todo siguiera de forma normal en 3 pasos establecerían una conexion; pero si tu idea es atacar enviarás un mismo paquete SYN desde varios equipos con una IP falsa de origen (spoofing) de tal forma que el servidor atacado envié respuesta a un host desconocido, esto hará que no reciba respuesta y la pila se vaya llenando de conexiones abiertas a la espera, si enviamos masivamente muchos paquetes SYN acabaremos por tirar la red y el host atacado quedará incomunicado.

![](/img/attack_syn.png)

* **Ataque Smurf**. Este ataque también es muy sencillo de realizar, si no se esta protegido. El ataque smurf se basa en enviar un ping a la dirección broadcast de la red, el ping utiliza el protocolo ICMP, cuando enviamos un ping enviamos un paquete a un host, si el host «esta vivo» nos devolverá el mismo paquete, imaginemos ahora que mediante un generador de paquetes creamos un paquete ping que contenga gran cantidad de información y hacemos spoofing cambiando la ip de origen por la de nuestro host víctima, si tenemos en cuenta que cuando enviamos un ping a la dirección broadcast este es devuelto por todos los host de la red, si enviamos muchos ping conseguiremos que todos ellos sean devueltos masivamente al host atacado y este quede fuera de juego, tened en cuenta además que una red de una empresa puede tener cientos de pc’s conectados.

![](/img/smurf-attack.jpg)

* **Virus y gusanos**. Este tipo de ataques muchas veces no es intencionado o mejor dicho no se elije a la víctima, es decir navegando por la red, descargando archivos de procedencia no segura puedes infectarte con un virus o con un gusano, un virus ataca a un equipo y según su morfología dañara de una u otra forma, y se propagara mediante ubs’s, etc. Un gusano se replica por toda la red consumiendo recursos de esta hasta tirarla y dejarla sin servicios. Para realizar este ataque puede conseguir acceder a la red de forma remota y liberar un gusano, puede enviarlo por correo a alguna persona dentro de la red mediante un falso correo útil spoofeando una dirección licita o usar miles de métodos como ingeniería social.
