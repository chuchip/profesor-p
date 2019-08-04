--- 
title: Frontend en Angular
weight: 10
pre: "<b>o </b>"
author: El Profe
type: post
date: 2018-09-13T07:51:25+00:00
url: /2018/09/13/aplicacion-en-angular-instalacion-y-configuracion-basica/
categories:
  - angular
  - rest
tags:
  - angular
  - rest

---
En anteriores entradas se creo [la parte del servidor de nuestra aplicación ][1].  Para la parte cliente o frontend usaremos Angular 6.

El código fuente del programa lo tenéis, como siempre,  <a href="https://github.com/chuchip/yagesclient-angular" target="_blank" rel="noopener">en mi página de GitHub</a>.

Lo primero sera instalar Angular, para ello tenemos un excelente manual de  como hacerlo en <a href="https://angular.io/guide/quickstart" target="_blank" rel="noopener">la página web de Angular</a>. Básicamente es instalar el servidor de aplicaciones <a href="https://nodejs.org/es/" target="_blank" rel="noopener">Node.js</a>. y su aplicación incluida **npm** (es un solo ejecutable) de <a href="https://nodejs.org/en/download/" target="_blank" rel="noopener">https://nodejs.org/en/download/</a> y luego, en una shell  (cmd en windows), ejecutar:

<pre><span class="pln">npm install </span><span class="pun">-</span><span class="pln">g </span><span class="lit">@angular</span><span class="pun">/</span><span class="pln">cli</span></pre>

lo cual nos instalara Angular, bajando de Internet todos los ficheros necesarios (que son bastantes).

Ahora crearemos nuestro proyecto a través del interprete de comandos (shell) de Angular (<a href="https://github.com/angular/angular-cli" target="_blank" rel="noopener">Angular CLI</a>) que acabamos de instalar.

Para ello abriremos un terminal en nuestro sistema operativo (Windows con CMD y Linux con una shell), nos posicionaremos en la carpeta donde vamos a crear nuestro proyecto y ejecutaremos el siguiente comando:

<pre><span class="pln">ng new yagesclient</span></pre>

Este comando creara la estructura básica (o template que dirían los ingleses) para el proyecto. Es normal que le cueste un rato, ya que descarga muchos ficheros  de internet .

Esta es una salida típica del comando **ng new **en una ventana windows:

![](/img/2018/09/angular1.png)

Y podemos ver que nos ha creado el directorio **yagesclient, **debajo del directorio donde estábamos:

![](/img/2018/09/angular2.png)

Ahora mismo ya podríamos ver esa aplicación mínima que hemos creado en nuestro navegador. Para ello, lanzamos nuestro servidor web con el comando:

<pre>ng serve --open

</pre>

Este comando nos mostrara la siguiente salida:

![](/img/2018/09/angular4.png)


Tambien abrirá una ventana de nuestro navegador donde podremos ver nuestra aplicación.


Fácil, ¿ verdad ?. Como se puede observar con muy poco esfuerzo se ha creado una aplicación en Angular, totalmente funcional que el servidor de <a href="https://nodejs.org/es/" target="_blank" rel="noopener">Node </a>que hemos instalado sirve.

 [1]: /aplicacion-usando-java-y-angular/