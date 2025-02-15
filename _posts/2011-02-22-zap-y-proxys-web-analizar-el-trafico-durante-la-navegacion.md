---
layout: post
title: "[THE BEGINNINGS] ZAP y proxys Web: Analizar el trafico durante la navegación"
date: 2011-02-22 18:35:53
categories: [web security, proxy]
tags: [zap, web proxy, traffic analysis, old]
---


![](/img/proxyweb.jpg)

Bueno debido a mi nuevo oficio, tengo menos tiempo del que me gustaria para dedicarle al blog, pero últimamente lo que más ando haciendo es aprendiendo y dándome cuenta de lo poquito que sé, el post de hoy va a tratar de los proxys webs, este tipo de aplicaciones se usan mucho a la hora de hacer auditorias webs para ver que hay "por debajo", con ellos podemos monitorizar todos los datos que se envían y que recibimos a la hora de realizar una navegación web, hay varios proxys; esta [Webscarab](href="http://www.owasp.org/index.php/Category:OWASP_WebScarab_Project) que es uno de los más complejos en cuanto a opciones y usabilidad y de los más potentes; tenemos también Paros, que es un proyecto ya abandonado pero muy intuitivo y fácil de usar; y el que hasta ahora es mi favorito: [ZAP (Zed Attack Proxy)](http://www.owasp.org/index.php/OWASP_Zed_Attack_Proxy_Project) que es un fork de [Paros](http://www.parosproxy.org/) que sigue actualizándose, este será el que usaré, para este post. ZAP esta corriendo bajo Fedora14 y con un Firefox con el Addons [FoxyProxy](https://addons.mozilla.org/es-ES/firefox/addon/foxyproxy-standard/) para poder gestionar rápidamente los proxys.

Este es el aspecto que presenta este proxy, primero tenemos que configurar la dirección del mismo que será a la que apunte el navegador para conectarse a internet. Para ellos simplemente `Tools -> Options -> Local Proxy` (yo lo tengo puesto por defecto `127.0.0.1:8080`).

![](/img/zap02.png)

Después tendríamos que configurar el navegador (en este caso Firefox y su plugin FoxyProxy), para ello añadimos la dirección del proxy a la lista de nuestro complemento, una vez configurado y guardado podemos empezar a navegar por cualquier web. Recordad que tenemos que seleccionar el perfil del nuevo proxy en el Firefox para poder ver lo que hacemos.

![](/img/zap03.png)

Vamos a comprobar que todo funciona, para ello vamos a realizar una navegación simple por Google por ejemplo y comprobar que nos enseña ZAP:

![](/img/zap04.png)

Como vemos en el ejemplo se ha realizado la búsqueda de la palabra "hola" en Google, esto se ha producido mediante un método GET, y estos son los datos que podemos ver que se han enviado en uno de los 11 envíos/recibos de datos:

![](/img/zap05.png)

En esa petición podemos ver como enviá el navegador los datos solicitados a Google y de que forma este predefine el envió de los mismos, como dijimos más arriba es mediante un método GET, eso significa que si copiamos la petición veríamos la búsqueda y si en el campo `q=` podemos "adiós" y lo enviamos por nuestro navegador podemos ver como Google busca el dato nuevo, es decir mediante un proxy web podemos modificar TODA la información que nuestro ordenador enviá al servidor web y muchas veces saltarnos las verificaciones que realiza el navegador del formato de nuestros datos mediante por ejemplo un JavaScript que se ejecuta en nuestros navegadores de forma local (cosa insegura por esta razón). Como podemos ver en la siguiente captura podemos ver y modificar todos los parámetros que vemos, como el nombre del navegador, el site de donde venimos, etc.

Como hemos comentado este envió se realiza mediante un método GET, otro método más "seguro o invisible" sería enviarlo mediante un método POST, una forma de localizarlos es cuando realizamos por ejemplo una búsqueda en un campo y en el navegador solo vemos algo del tipo `http://www.dominio.com/busqueda.php` podríamos deducir que se trata de un POST.

Aquí podemos ver otra característica de ZAP, mediante el botón `Break` podemos crear un punto de interrupción a partir del cual podemos controlar, modificar y analizar los datos que se envían desde el navegador hasta el servidor destino: 

![](/img/break.png)

Dentro del rectángulo rojo encontramos tres botones, el primero en verde es el que crea el `Break`, el siguiente confirmaría el envío actual y daría paso a la siguiente interrupción y el botón `Play` cancelaría el `Break` y haría que la comunicación vuelva a ser fluida y sin espera ni interrupciones, entre envío y envío de datos.

Esta es la explicación básica de como funciona un proxy web y como se crearía un `Break` o `Trap`, estos programas tienen otras funcionalidades como escaneres de puertos, de vulnerabilidades, `spiders` que analizan el site listando todos los directorios, etc.
