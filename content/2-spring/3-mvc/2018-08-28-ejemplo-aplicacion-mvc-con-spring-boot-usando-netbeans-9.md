---
title: 'Ejemplo aplicación MVC con Spring Boot  usando NetBeans 9'
author: airec69
type: post
date: 2018-08-28T06:51:33+00:00
url: /2018/08/28/ejemplo-aplicacion-mvc-con-spring-boot-usando-netbeans-9/
categories:
  - java
  - mvc
  - netbeans
  - spring boot
tags:
  - java
  - mvc
  - netbeans
  - spring boot

---
<a href="http://www.profesor-p.com/2018/08/27/instalar-plugin-spring-boot-con-netbeans-9-en-ubuntu-18-04/" target="_blank" rel="noopener">En una entrada anterior</a> explique como instalar NetBeans 9, añadiendole el plugin para usar Spring Boot.

Ahora vamos a crear nuestro primer proyecto en este entorno.

Pulsaremos **New Project,** lo cual nos mostrara una pantalla como la siguiente.

<img class="alignnone size-full wp-image-123" src="http://www.profesor-p.com/wp-content/uploads/2018/08/Captura3.png" alt="" width="742" height="514" srcset="http://www.profesor-p.com/wp-content/uploads/2018/08/Captura3.png 742w, http://www.profesor-p.com/wp-content/uploads/2018/08/Captura3-300x208.png 300w" sizes="(max-width: 742px) 100vw, 742px" />

y en el campo **Filter** pondremos _spring, _ para después eligir **Spring Boot Inititilizr project.** Pulsaremos **Next** y nos pedirá  una serie de datos sobre  nuestro proyecto. Para este ejemplo podemos dejar los campos como aparecen por defecto.

<img class="alignnone size-full wp-image-140" src="http://www.profesor-p.com/wp-content/uploads/2018/08/Captura9.png" alt="" width="725" height="503" srcset="http://www.profesor-p.com/wp-content/uploads/2018/08/Captura9.png 725w, http://www.profesor-p.com/wp-content/uploads/2018/08/Captura9-300x208.png 300w" sizes="(max-width: 725px) 100vw, 725px" />

Pulsamos siguiente (Next) y ahora debemos especificar de que tipo es nuestro proyecto:

&nbsp;

<img class="alignnone size-full wp-image-122" src="http://www.profesor-p.com/wp-content/uploads/2018/08/Captura4.png" alt="" width="790" height="484" srcset="http://www.profesor-p.com/wp-content/uploads/2018/08/Captura4.png 790w, http://www.profesor-p.com/wp-content/uploads/2018/08/Captura4-300x184.png 300w, http://www.profesor-p.com/wp-content/uploads/2018/08/Captura4-768x471.png 768w" sizes="(max-width: 790px) 100vw, 790px" />

Como se ve hay muchas opciones, nosotros solo marcaremos la casilla **WEB**, porque va a ser una aplicación WEB y **Thymeleaf**, porque es el motor de plantillas (Template Engines) que vamos a usar. **Thymeleaf** es un sustituto o evolución de JSP con diferentes mejoras y que se integra perfectamente con Spring. Su pagina web es: <a href="https://www.thymeleaf.org/" target="_blank" rel="noopener">https://www.thymeleaf.org/</a> y tenéis una buena documentación. De todos modos, en este ejemplo solo usaremos código HTML sin poner código en nuestras plantillas.

Volveremos a pulsar **Next**, para, en el ultimo paso poner el nombre de nuestro provecto

&nbsp;

<img class="alignnone size-full wp-image-130" src="http://www.profesor-p.com/wp-content/uploads/2018/08/Captura5.png" alt="" width="820" height="589" srcset="http://www.profesor-p.com/wp-content/uploads/2018/08/Captura5.png 820w, http://www.profesor-p.com/wp-content/uploads/2018/08/Captura5-300x215.png 300w, http://www.profesor-p.com/wp-content/uploads/2018/08/Captura5-768x552.png 768w" sizes="(max-width: 820px) 100vw, 820px" />:

Pulsaremos **Finish ** y esperaremos a que Maven baje los paquetes necesarios y configure el entorno de trabajo.

Ahora vamos a crear un par de paginas web. La primera sera **index.html**, la otra sera **otrapagina.html**. Esas paginas las debemos crear en **OtherResources   src/main/resources    templates**, como se ve en la siguiente imagen:

<img class="alignnone size-large wp-image-128" src="http://www.profesor-p.com/wp-content/uploads/2018/08/Captura7-1024x603.png" alt="" width="1024" height="603" srcset="http://www.profesor-p.com/wp-content/uploads/2018/08/Captura7-1024x603.png 1024w, http://www.profesor-p.com/wp-content/uploads/2018/08/Captura7-300x177.png 300w, http://www.profesor-p.com/wp-content/uploads/2018/08/Captura7-768x452.png 768w, http://www.profesor-p.com/wp-content/uploads/2018/08/Captura7.png 1075w" sizes="(max-width: 1024px) 100vw, 1024px" />Los ficheros creados son tipo HTML.

Los Ficheros quedaran así

**index.html**

<pre>&lt;!DOCTYPE html&gt;
&lt;html xmlns="http://www.w3.org/1999/xhtml" xmlns:th="http://www.thymeleaf.org"
      xmlns:sec="http://www.thymeleaf.org/thymeleaf-extras-springsecurity3"&gt;
    &lt;head&gt;
        &lt;title&gt;Pagina de Inicio&lt;/title&gt;
        &lt;meta charset="UTF-8"/&gt;
        &lt;meta name="viewport" content="width=device-width, initial-scale=1.0"/&gt;
    &lt;/head&gt;
    &lt;body&gt;
        &lt;div&gt;Mi pagina de inicio&lt;/div&gt;
       &lt;div&gt;&lt;a href="otra"&gt;ir a la otra pagina&lt;/a&gt;&lt;/div&gt;
    &lt;/body&gt;
&lt;/html&gt;</pre>

**otrapagina.html**

<pre>&lt;!DOCTYPE html&gt;

&lt;html&gt;
    &lt;head&gt;
        &lt;title&gt;Otra pagina de prueba&lt;/title&gt;
        &lt;meta charset="UTF-8"&gt;
        &lt;meta name="viewport" content="width=device-width, initial-scale=1.0"&gt;
    &lt;/head&gt;
    &lt;body&gt;
        &lt;div&gt;Mi otra pagina de prueba&lt;/div&gt;
    &lt;/body&gt;
&lt;/html&gt;</pre>

Ahora mismo ya podríamos ejecutar nuestra aplicación, y veríamos nuestro index.html, pero vamos a crear el controlador para que se pueda mostrar la pagina **otrapagina.html**. Para ello, en **Source Packages,** dentro del paquete **com.example.demo** crearemos un nuevo fichero java al que llamaremos **Controlador.java**

En este fichero tendremos lo siguiente:

<pre>package com.example.demo;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;

@Controller
public class Controlador {

        @GetMapping("otra")
	 public String login()
	 {		
		 return "otrapagina";
	 }  
}</pre>

Esta clase marcada con @Controller, tendrá una única función que tiene la etiqueta @GetMapping. El parámetro de esa etiqueta sera la URL o página que debe tratar la función que tenemos debajo.

Es decir, que cuando nosotros pongamos en nuestro navegador:

**http://localhost:8080/otra**

Spring ejecutara lo que haya en la función **login()**.

Si quisiéramos tener mas paginas simplemente añadiremos más funciones, con la etiqueta @GetMapping(&#8220;PAGINA QUE NOS DE LA GANA&#8221;)

Terminar, comentando que el String devuelto por nuestra función es la pagina web (el fichero) creado bajo el directorio templates (sin la extensión html). Resumiendo, en este ejemplo, al ir a:

http://localhost:8080/otra

Spring parseara el fichero **otrapagina.html** y devolverá la pagina  HTML  creada por el.

Y con esto, ya hemos creado nuestra primera pagina web con Spring 😉

<img class="alignnone size-full wp-image-141" src="http://www.profesor-p.com/wp-content/uploads/2018/08/Captura10.png" alt="" width="418" height="200" srcset="http://www.profesor-p.com/wp-content/uploads/2018/08/Captura10.png 418w, http://www.profesor-p.com/wp-content/uploads/2018/08/Captura10-300x144.png 300w" sizes="(max-width: 418px) 100vw, 418px" />

<img class="alignnone size-full wp-image-142" src="http://www.profesor-p.com/wp-content/uploads/2018/08/Captura11.png" alt="" width="413" height="146" srcset="http://www.profesor-p.com/wp-content/uploads/2018/08/Captura11.png 413w, http://www.profesor-p.com/wp-content/uploads/2018/08/Captura11-300x106.png 300w" sizes="(max-width: 413px) 100vw, 413px" />

Terminar diciendo que el código fuente de este proyecto lo tenéis en mi <a href="https://github.com/chuchip/mvc_springboot" target="_blank" rel="noopener">repositorio de GITHUB.</a>

&nbsp;