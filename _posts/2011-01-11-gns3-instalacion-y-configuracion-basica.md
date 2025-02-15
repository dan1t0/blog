---
layout: post
title: "[THE BEGINNINGS] GNS3: Instalación y configuración básica"
date: 2011-01-11 11:00:08
categories: [networking, tutorial]
tags: [gns3, network emulation, tutorial, old]
---


![GSN3 Logo](img/gns3.png)

Cuando hablamos de seguridad de la información, una pieza muy importante para saber de lo que hablamos muchas veces es poder verlo, es decir comprobarlo; hay muchas pruebas que no podemos realizar, cuando hablamos por ejemplo de un súper servidor o de un software de firewall muy determinado es complicado poder investigar sobre ello o probar una determinada vulnerabilidad existente. Sin embargo hoy os voy a hablar sobre un software que virtualiza el sistema operativo de los switch y router de Cisco, se trata de GNS3, un software multiplataforma, que es muy útil, ya que se adapta perfectamente con Wireshark, Qemu e incluso si tenemos varias tarjetas de red podemos emular una red desde ellas y ver como se comportaría el modelo determinado de router. En este post intentare explicaos como instalarlo.

#### Instalación en Windows

Descargar GNS3, el paquete incluye Dynamips (un software necesario para correr GNS3), Putty (cliente para conexiones SSH y telnet por excelencia), WinPCAP (conjunto de librerías para trabajar con protocolos de red presente en los analizadores de red) y Quemu/Pemu (librerías para poder conectar GNS3 con Qemu, software libre emulador de sistemas operativos tipo VMware).

* Como la mayoría de las instalaciones en windows hacemos un "next, next…"
* Nos aparecerá una ventana con dos acciones necesarias a realizar:
![Acciones](http://dan1t0.wordpress.com/wp-content/uploads/2011/01/gns01.png)

* El paso 1 nos dice que tenemos que configurar y comprobar el path. Y comprobar si el software esta trabajando en una ruta que nos valga.

* Una vez realizados estos pasos solo nos quedara seleccionar los IOS desde la carpeta seleccionada en el paso anterior, para ello nos vamos a *Editar -> Imagenes IOS y hypervisors*. En la pestaña de Imágenes en Image Files seleccionamos el/los IOS que usaremos y podemos configurar algunos datos más como el modelo donde lo aplicaremos el IOS, la RAM del mismo y otras opciones.

![Opciones](http://dan1t0.wordpress.com/wp-content/uploads/2011/01/gns02.png)

* El último paso es testera el modulo Dynamips, para ello en<em> Editar -&gt; Preferencias -&gt; Dynamips</em> le damos al botón Test y debería salir un mensaje en verde, si es así ya podremos empezar a usar GNS3 en windows.

#### Instalación en MacOS

* Lo primero es descargar el archivo de [aqui](http://downloads.sourceforge.net/project/gns-3/GNS3/0.7.3/GNS3-0.7.3-intel-x86_64.dmg?r=http%3A%2F%2Fwww.gns3.net%2Fdownload&amp;ts=1294020397&amp;use_mirror=ignum) 

* Una vez descargado, como una aplicación normal de MacOS copiamos el .app a la carpeta que queramos, ahora tenemos que descargarnos desde [aqui](http://sourceforge.net/projects/dyna-gen/files/OS%20X%20Intel%2010.5/Dynagen%200.11.0%20and%20Dynamips%200.2.8-RC2%20OS%20X%20Intel%2010.5/Dynagen_0.11.0-Leo.dmg/download) > el Dynagen (Dynamips), para hacer funcionar el GNS3.

* Una vez descargado al igual que en windows *Editar -> Preferencias -> Dynamips* y ahí en ruta del ejecutable tendremos que escribir (a mi no me dejo seleccionarlo directamente) la ruta donde tenemos descargado dynamips, por ejemplo: */Users/danito/Desktop/Dynagen/dynamips* una vez hecho esto y para asegurarse si pulsamos abajo el botón de test ya tendríamos listo el GNS3 en MacOS, ya solo nos quedaría seleccionar los IOS.

* Para seleccionar los IOS es la misma operación que en windows, desde *Editar -> Imagenes IOS y hypervisors*. En la pestaña de Imágenes en Image Files seleccionamos el/los IOS que usaremos y podemos configurar algunos datos más como el modelo donde lo aplicaremos el IOS, la RAM del mismo y otras opciones.

![GNS3 en Windows XP abriendo un consola](http://dan1t0.wordpress.com/wp-content/uploads/2011/01/gns03.png)

#### Consideraciones generales y consejos:

Como ya os comentaba en la introducción, una de las cosas que más me gusta de este software es que puedes conectar maquinas virtuales Qemu, esto nos da mucha versatilidad y le da mucha potencia a la aplicación siendo la capacidad de GNS3 la que nosotros queramos (emulación y prueba de DDos, MiTM, secularizar Apache, BBDD, etc). También podemos 'meter' en medio un Wireshark que nos puede ayudar a entender que esta pasando, desde problemas de red antes de hacer una subida de una configuración a reproducción, hasta estudiar los protocolos de enrolamiento viendo los paquetes que se envían entre equipos, etc.

También es valido este software para facilitar el estudio de certificaciones como CCNA, CCNP, CCNA-Sec, etc, teniendo en cuenta que GNS3 es mucho mas potente que el ya conocido PacketTracer de CISCO.

Es conveniente explicar que GNS3 es un software de emulación y necesita un sistema operativo IOS que emular, el cual se supone que es ilegal distribuirlo, pero ya sabéis como conseguirlo. Cuando consigáis o os compréis una IOS para practicar recordar que siempre es interesante que la IOS acepte crypto (opciones de seguridad) para ello una forma de guiarse es con el nombre que tiene, si lleva el sufijo **k9**, también comentaros que este software no emula equipos de capa 2 como switches, la única forma de trabajar con switchs es con un firmware de un equipo router multicapa, es decir que tiene opciones de configuración de switchs), yo uso el modelo Cisco 3725 y una posible IOS seria: `c3725-adventerprisek9-mz.124-7.bin`.

Espero poder hacer una entrega futura, cuando disponga de más tiempo, explicando como meter un GNS3 en una red puenteado con una interfaz de red y usarlo de servidor DHCP spoofeado para redirigir el tráfico de nuevo al original y poder ver lo que pasa con los equipos que 'pesquemos'. Como ya sabéis el limite lo pone el conocimiento y la imaginación que tengamos, imaginaos añadiendo un proxy en PHP al que habilitemos SSL-strip ;).
