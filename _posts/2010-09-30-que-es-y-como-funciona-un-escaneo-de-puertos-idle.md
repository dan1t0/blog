---
layout: post
title: "[THE BEGINNINGS] Qué es y cómo funciona un escaneo de puertos Idle"
date: 2010-09-30 11:00:19
categories: [network security, scanning]
tags: [port scanning, idle scan, nmap, old]
---


![](/img/idle_scan.png)


Hoy vamos a hablar de los [escaneos de puertos](http://es.wikipedia.org/wiki/Esc%C3%A1ner_de_puertos), ya todos más o menos conocemos o hemos oído hablar de escaneos tipo TCP, SYN, UDP, ACK, ICMP, etc pero hay un tipo de escaneo muy curioso y muy especial que se llama Idle (del inglés vago) en este post os voy a intentar explicar las principales características para entender como funciona.

Para entender un poco el funcionamiento de este método hay que conocer o tener unas nociones básicas de como funciona la [conexión en el protocolo TCP](http://es.wikipedia.org/wiki/Tcp#Establecimiento_de_la_conexi.C3.B3n_.28negociaci.C3.B3n_en_tres_pasos.29).

Cuando se realiza un escaneo de puertos sobre un host, si este esta minimamente securizado detectara que esta siendo víctima de un escaneo de puertos y la IP del atacante quedará grabada en los logs con lo que no seria difícil localizarle, aqui es donde nace el escaneo idle, en el que introduce una variante más en la ecuación, un tercer host que actuara de [zombie](http://es.wikipedia.org/wiki/Zombie_(inform%C3%A1tica)), es decir hará el trabajo sucio por el supuesto atacante. Veamos un resumen de una conexión TCP normal entre dos host:

![](/img/Tcp-handshake.png)

En una conexión normal el cliente envia un paquete con un valor SYN al servidor, el servidor devuelve un paquete con un valor SYN + un valor ACK y un valor SEQ, si todo ha ido correctamente el cliente devolvera un paquete con un valor ACK+1 y una valor SEQ+1. Esto se llama negociación en tres pasos. Si el puerto al que se lanza la conexión esta cerrado el servidor cancelara la conexión, enviara un paquete con el bit del valor RST activado lo que significa que rechaza la conexión y lo que nos interesa es que **al no completar la conexión el valor ACK del cliente no varia**.

Explicado el método de conexión es donde entra nuestro host zombie, como hemos dicho antes, si se realiza un escaneo de este tipo el host "atacante" seria facilmente localizable, pero ¿qué pasaría si entre el host atacante y la víctima se metiera un zombie? Si nosotros controlamos el valor SEQ de un equipo zombie podríamos mediante una tecnica de suplantación de identidad (spoofing) construir un paquete TCP que al mandarlo al host víctima devolviera la conexión con éxito (SEQ+1) o la rechazada (RST=1) al equipo zombie y el atacante pudiera comprobar de nuevo mediante un paquete TCP el valor de SEQ y saber si ese puerto esta abierto o cerrado sin enviar ni un solo paquete al host víctima (el único que se envia tiene como ip origen la del host zombie). Lo entenderemos mejor con este esquema:

![](/img/Idlescan_Technique_es.png)

La utilidad que en un principio se le puede ver a esta técnica es saber si tiene determinados puertos abiertos, pero si le damos una vuelta de rosca a la técnica nos daremos cuenta de un uso mucho mas práctico.

Imaginemos la red de una empresa algo parecido a esto:
![](/img/dmz.png)


Cuando tenemos una red de este tipo con una zona [DMZ](http://es.wikipedia.org/wiki/Zona_desmilitarizada_(inform%C3%A1tica)) los host de la LAN pueden acceder al los servidores HTTP, J2EE, etc de la zona DMZ, para acceder a ella tienen que pasar por un firewall que a buen seguro tendra un filtrado de IP's y de puertos determinado con lo que un host externo con una IP desconocida por el firewall será "tirado" cuando intente realizar una conexión (escaneo de puertos por ejemplo). Una forma de saber que puertos están abiertos en el firewall para poder descubrir una vulnerabilidad es **realizar un escaneo idle usando como zombie un host con una ip perteneciente a la zona de confianza del firewall** de esta forma el escaneo nos devolvera una lista de puertos abiertos para una ip determinada.

Una forma de hacer que los escaneos de este tipo no sean validos viene por parte de los desarrolladores de los sistemas operativos,  el valor SEQ se incrementa de uno en uno, si este valor se reseteara de alguna forma o diera algún valor aleatorio según el momento o la conexión estos escaneos serian inviables.

La fuente de este articulo ha sido la web del manual de Nmap, podeis conseguir muchísima más información y ejemplos de implementación de este tipo de escaneo con Nmap en su [web](http://nmap.org/idlescan-es.html).
