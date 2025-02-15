---
layout: post
title: "[THE BEGINNINGS] Seguridad: comprometiendo un switch (Parte 2 de 2)"
date: 2010-12-23 12:17:45
categories: [network security, switching]
tags: [switch security, network compromise, part2, old]
---


![](/img/principalv2.png)

En el articulo anterior pudimos ver cómo comprometer la seguridad de un switch y como me pareció un tema interesante decidí crear esta segunda parte en la que os explicare como securizar un switch de manera que podamos protegerlo contra una serie de ataques, trabajaremos con una sintaxis de IOS que es el sistema operativo de Cisco, que es lo que a fin de cuentas es más normal encontrarse dentro de una red más o menos importante, pero lo importante es entender el concepto y la diferentes medidas.

La mayoría de los switch poseen unos puertos que funcionan con un ancho de banda superior al resto de puertos y suelen ser tipo Ethernet Giga, en estos puertos lo correcto es conectar host que necesiten más ancho de banda que un equipo final normal, estos puertos suelen estar destinados a servidores, por lo que no solo seria necesaria asignar una MAC a ese puerto. Estos puertos también pueden estar destinados a ser puertos troncales, un puerto troncal cumple la finalidad de unir dos switchs, por lo que en estos puertos habría que actuar con cuidado y calculando bien antes de definir un numero máximo de direcciones MAC.

Pasemos al código, una vez dentro del switch pasamos a modo configuración:

```
sw(config)# interface interfaz slot/port # seleccionamos la interfaz y el puerto
sw(config)# switchport mode access # automaticamente cuando conectemos un host funciona
sw(config)# switch port-security # activamos la seguridad
```

Llegados a este punto pasaremos a analizar los comandos y las opciones más comunes que podemos configurar ahora.

`sw(config)# switch port-security aging`

El comando `aging` define cuanto tiempo se aplica la acción definida cuando se produce una violación de las reglas, pasado ese tiempo acaba la restricción. Hablaremos de eso más adelante. Por defecto el comando da un tiempo de 45 segundos que podemos modificar añadiendo los minutos y segundos de la acción después del comando `aging`, por ejemplo si ponemos:

```
sw(config)# switch port-security aging 0 0
# La acción se ejecuta siempre y de forma infinita

# or

sw(config)# switch port-security aging 5 30
# La acción duraria 5 minutos y 30 segundos

sw(config)# switch port-security maximum n
# El comando maximun nos dice el número de direcciones máximas que aprenderá el puerto (n)
```

`sw(config)# switch port-security mac-addres MMMM.AAAA.CCC`

El comando `mac-addres` nos dice las direcciones MAC de los host que tendrán permitido el acceso a ese puerto, este comando tiene sentido si existe un número máximo de direcciones definido previamente, esta sería la opción ideal para un puerto Giga que solo necesitamos conectar un servidor. También nos puede ser útil si vamos a conectar una serie de equipos finitos o dentro de un rango normal que sepamos que si se supera es porque algo raro esta pasando (como un ataque de ARP-Flood), para estos casos nos puede interesar el comando:

```
sw(config)# switch port-security mac-addres sticky
```

Con este comando las direcciones se irán almacenando en la tabla CAM (MAC) del switch de forma automatica y cuando llegue al limite ya tendria definidas las direcciones MAC posibles que se conectarían desde ese puerto.

```
sw(config)# switch port-security violation {shutdown/protect/restrict}
```

Este comando sirve para especificar que acción se llevara acabo en caso de que alguna de nuestras reglas definidas anteriormente sea saltada, nos encontramos con tres opciones:

* `shutdown`: Es la opción por defecto y lo que hace es tirar el puerto que ha sufrido la violación.
* `protect`: Evita la entrada a la red a la MAC no permitida.
* `restrict`: Evita la entrada a la red a la MAC no permitida y además manda una alerta al servidor de eventos por lo que nos da más control sobre lo que pasa en la red.

Cabría añadir que si queremos configurar un switch entero seria más sencillo y rápido usando rangos de configuración de interfaces:

```
sw(config)# interface interfaz slot/port - ultimoPuerto

# Por ejemplo: sw(config)# interface <em>Fa0 0/0 - 15
```

Si aplicamos estas reglas correctamente podremos tener un switch configurado correctamente y nos ahorraríamos la mayor parte de los problemas y ataques mediante ARP.
