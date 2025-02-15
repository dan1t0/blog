---
layout: post
title: "[THE BEGINNINGS] Wireshark: Instalación en Mac OS y manual de uso"
date: 2010-09-18
categories: [networking, tutorial]
tags: [wireshark, macos, tutorial, old]
---


![](/img/200px-wireshark_icon-svg.png)

Wireshark es el analizador de trafico de red más famoso, se usa en muchos ámbitos, educativo, profesional, empresa, etc. Es un programa multiplataforma, muy fácil de instalar en linux ya que esta disponible en todos los repositorios, muy sencillo de instalar en Windows simplemente ejecutando el .exe, pero no tan trivial en Mac OS. En este post explicaremos como se instala wireshark en Mac OS, estamos acostumbrados a arrastrar y soltar en la carpeta de aplicaciones para hacer que un programa se instale pero en el caso del sniffer por excelencia, la instalación no es tan simple, os explicare de forma sencilla cómo instalarlo y hacerlo funcionar. Además os enlazo a un manual de la  Universidad central de Venezuela.

Lo primero es ir a la [web de descarga](http://www.wireshark.org/download.html) y elegir el archivo que se adapte a vuestro sistema (fijaos que hay versión de 64bits para Leopard). Una vez descargado:

* Arrastramos el archivo Wireshark.app a la carpeta de aplicaciones

* Tenemos  que dar permisos de ejecución a una serie de dipositivos dentro de nuestro mac, para ello abrimos el terminal y ejecutamos el siguiente comando:
`sudo chmod 755 /dev/bpf*`

* (nos pedira la contraseña de administrador, la ponemos y pulsamos enter)

* Por último tenemos que copiar todos los archivos de la carpeta Utilities/Command Line a un directorio del Mac para que el programa pueda funcionar correctamente, para ello en el terminal ejecutamos el siguiente comando:
`sudo cp /Volumes/Wireshark/Utilities/Command\ Line/* /usr/local/bin/`

* (nos pedirá la contraseña de administrador, la ponemos y pulsamos enter)

Con estos dos últimos sencillos casos el programa funcionará correctamente, y aquí os dejo un sencillo manual de uso (viene la instalación en windows también pero los comandos y el modo es idéntico en las tres plataformas)

Nota: Si abris la aplicación y no encontrais interfaces de red en las que ejecutar Wreshark, es porque no tiene permisos de ejecucion sobre las mismas, volved a ejecutar:

`sudo chmod 755 /dev/bpf*`

Nota2: Otra opción seria ejecutar como root wireshark, para según que uso y donde estemos puede ser más util, para ello:

`sudo ./Applications/Wireshark.app/Contents/Resources/bin/wireshark`
