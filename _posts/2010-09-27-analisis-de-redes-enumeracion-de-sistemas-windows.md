---
layout: post
title: "[THE BEGINNINGS] Análisis de redes: Enumeración de sistemas Windows"
date: 2010-09-27
categories: [network security, windows]
tags: [network analysis, windows, enumeration, old]
---


![](/img/securitywin.jpg)

Dentro de una red encontramos diferentes host que dan un determinado servicio a la red, ya sean servidores de correo, servidores de impresión, servidores de dominio, o simplemente equipos que comparten carpetas o impresoras. Podríamos decir que las técnicas de enumeración nos ayudaran a descubrir todos estos recursos y indicarnos qué usuarios pueden acceder a ellos. Debido a que cada sistema operativo tiene unos servicios y una forma de gestionarlos diferente, estas técnicas las tenemos que dividir según el sistema operativo que corran. Antes de pasar a explicar los distintos métodos de enumeración citaremos los tres protocolos en los que nos basaremos para recopilar esta información: [NetBIOS](http://es.wikipedia.org/wiki/NetBIOS), [SNMP](http://es.wikipedia.org/wiki/Simple_Network_Management_Protocol) y [LDAP](http://es.wikipedia.org/wiki/LDAP).

#### Enumeración NetBIOS
* Listar maquinas de nuestra red
`C:\>net view`

* Listar recursos compartidos en nuestra red. Para este caso tenemos diferente opciones,
`C:\>net view <NOMBRE_HOST>`

Nos mostrara los recursos compartidos de esa maquina, este es el comando más completo ya que también podemos usar: 
`C:\>nbtstat -a <NOMBRE_HOST>`
o
`C:\>nbtstat -A <IP_HOST>`

Estos dos comandos solo nos indican que la máquina tiene servicios corriendo, pero no especifica cuales.

* Listar dominios accesibles de una red
`C:\>net view /domain`

* Listar máquinas de un dominio determinado
`C:\>net view /domain:<NOMBRE_DOMINIO>`

* Listar recursos compartidos en maquinas de diferentes dominios
`C:\>net view /domain:<NOMBRE_DOMINIO> \<NOMBRE_HOST>`

Cabe destacar que en todos los comandos podemos sustituir el nombre de la maquina por la IP de la misma y como ya sabemos Internet no es mas que una gran red por lo que podemos sustituir las IP’s de nuestra red por IP’s que pertenezcan a redes y servidores externos de Internet.

También es importante comentar que en un servidor linux que tenga implementado y activo el protocolo samba también se podrán ejecutar estos comandos contra el y averiguar información.

#### Enumeración SNMP
La enumeración de sistemas mediante SNMP no es tan trivial como la enumeración NetBIOS. El protocolo SNMP (Simple Network Managent Protocol). Este protocolo se usa para poder monitorizar y controlar el estado de los diferente dispositivos conetados a una red, normalemente se usa para routers, switchs, servidores de impresión, correo o web; pero se puede usar también sobre cualquier dispositivo que tenga disponible ese protocolo, es un protocolo multiplataforma y se podrá usar siempre que el dispositivo desde el que queramos monitorizar tenga el paquete de software instalado correctamente. En entornos windows existe un programa llamado Getif con un entorno grafico desde el que podemos obtener de una manera sencilla toda la información, mientras que en entornos linux tenemos Scotty y Cheops.


#### Enumeración LDAP
LDAP (Lightweight Directory Access Protocol) es un protocolo diseñado para trabajar con TCP/IP y junto a un directorio jerárquico. Esto puede proporcionar una determinada información sobre los usuarios de esa red como nombres, teléfonos, direcciones de correo, etc. este tipo de objetos se les conoce como AD (Active Directory). Recordemos que tanto los dominios dentro de Windows 2000 Server y Windows 2003 Server trabajan con este tipo de objetos por lo que LDAP nos ayudara a extraer facilmente gran cantidad de información. Para poder sacar información de este tipo podemos usar la aplicación ldp.exe que se encuentra en las herramientas de soporte de Windows XP. Cabe destacar que el manejo de esta herramienta no es trivial y se requiere un alto conocimiento sobre este protocolo.

**Nota**: Para poder realizar estos rastreos utilizaremos comandos NET integrados en sistemas operativos Windows NT.
