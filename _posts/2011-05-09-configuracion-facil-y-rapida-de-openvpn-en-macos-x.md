---
layout: post
title: "[THE BEGINNINGS] Configuración fácil y rápida de OpenVPN en MacOS X"
date: 2011-05-09 10:32:21
categories: [vpn, macos]
tags: [openvpn, vpn setup, macos, old]
---


![](/img/openvpn.png)

Desde hace un tiempo me he estado pegando con distintos clientes para [OpenVPN](http://es.wikipedia.org/wiki/OpenVPN) en MacOS X y la verdad he probado varios como [Tunnelblick](http://code.google.com/p/tunnelblick/) que es el cliente recomendado por OpenVPN en MacOS X con interfaz gráfica. Pero la verdad es que últimamente me estoy aficionando más a la terminal así que en este post pasaré a explicar de una forma muy sencilla la instalación y conexión contra un servidor OpenVPN desde un MacOS X.

Primero comentar que hay dos formas de instalar el cliente. Por un lado podemos descargar el código fuente de OpenVPN desde [aquí](href="http://tuntaposx.sourceforge.net/) y posteriormente necesitaremos instalar [TUNTAP](href="http://tuntaposx.sourceforge.net/), que es una extensión del kernel que se comporta como un demonio que permite la creación de interfaces virtuales de red, podéis descargar el código fuente desde [aquí](http://tuntaposx.sourceforge.net/download.xhtml). Luego habría que compilar los programas y crear nuestra VPN. _Pero hay una forma más sencilla_ que es la que pasaré a contar. Para ello previamente necesitaremos tener instalado en nuestro sistema [MacPorts](http://dan1t0.wordpress.com/2010/09/01/macports-repositorio-de-aplicaciones-de-linux-en-nuestro-mac-os-x/).

Lo primero es instalar con MacPorts los dos paquetes necesarios, para ello simplemente abrimos un terminal y escribimos:

```
$ sudo port install openvpn2  #instalamos el primer paquete
$ sudo port install tuntaposx  #instalamos el segundo paquete
```

Una vez instalados los dos paquetes primero deberemos arrancar el demonio tuntaposx para que openvpn2 pueda crear las interfaces tun, para ello:

`sudo port load tuntaposx  #cargamos el modulo del kernel`

Llegados a este punto es necesario tener los archivos de configuración de la VPN a la que queramos conectar, para ello debemos contar con un archivo de configuración y una serie de keys que nos debe de generar previamente el administrador de la red, para cargar la VPN, tan fácil como:

`sudo openvpn2 MiConf.conf &   #arrancamos la VPN`

Si el proceso se realizo correctamente ya deberíamos tener levantada nuestra conexión OpenVPN (podemos comprobarlo escribiendo en el terminal `sudo ifconfig` y ver si existe alguna red del tipo `tunX`.

Si tuviéramos que crear alguna ruta para llegar algún equipo de la VPN ahora seria el momento, para ello utilizaríamos el comando `route -n`.

Por último una vez queramos acabar con la conexión OpenVPN simplemente debemos matar el proceso (con un `sudo kill -9 [PID-openvpn2]` devuelto al ejecutar el openvpn2) y bajamos el demonio `tuntap`, para ello simplemente escribimos:

`sudo port unload tuntaposx  #bajamos el modulo`
