---
title: Generar paginas Web estaticas dinamicamente
author: El Profe
type: post
date: 2018-11-28T12:32:50+00:00
url: /2018/11/28/generar-paginas-web-estaticas-dinamicamente/
categories:
  - hugo
  - web
tags:
  - hugo
  - web

---
Buenas alumnos/as,

En esta entrada hablare de [Hugo][1]. Este un software  permite crear sitios web estáticos pero trabajando _casi_ como si fueran dinámicos. Para ello podremos usar diferentes plantillas que configuraremos, para luego añadir entradas y contenido dinámico fácilmente.

Hugo usa [markdown][2] para añadir los contenidos pero también podremos usar HTML, CSS y JavaScript si fuera necesario.

Para crear una pagina web lo primero que tenemos que hacer es bajarnos el programa, lo cual podemos hacer desde <a class="url" href="https://github.com/gohugoio/hugo/releases" target="_blank" rel="noopener">https://github.com/gohugoio/hugo/releases</a> Una vez tengamos instalado **hugo** en nuestro ordenador, podremos empezar a crear nuestra propia pagina web.

Como primer paso abriremos un terminal y ejecutaremos la siguiente sentencia:

     > hugo new site mipagina
    

Con esto **hugo** creara un directorio llamado &#8216;mipagina&#8217; debajo del actual. Nos moveremos a el directorio para poder instalar el tema con el que queremos que funcione nuestra página web.

    > cd mipagina
    

En <a class="url" href="https://themes.gohugo.io/" target="_blank" rel="noopener">https://themes.gohugo.io/</a> tenemos todas los temas (o plantillas) sobre las que basar nuestra página web. Estos temas están en su mayoría alojados en GitHub por lo cual lo más fácil para descargarlos es usar el comando **git** . De todos modos, siempre tenemos la opción de descargar un fichero _zip_ que después descomprimiremos en el directorio **themes**

Supongamos que queremos usar el tema **initio** que es el que usamos en la página de [RiojaTech Alliance][3] . Para ello iremos a su URL <a class="url" href="https://themes.gohugo.io/hugo-initio/" target="_blank" rel="noopener">https://themes.gohugo.io/hugo-initio/</a> y pulsaremos sobre el botón &#8216;**Download**&#8216;, lo cual nos llevara a **GitHub**, a la dirección <a class="url" href="https://github.com/miguelsimoni/hugo-initio" target="_blank" rel="noopener">https://github.com/miguelsimoni/hugo-initio</a>. Allí descargaremos el fichero **ZIP**, dándole a l botón **Download ZIP**.

<img class="size-large wp-image-510 aligncenter" src="http://www.profesor-p.com/wp-content/uploads/2018/11/github-1024x254.png" alt="" width="1024" height="254" srcset="http://www.profesor-p.com/wp-content/uploads/2018/11/github-1024x254.png 1024w, http://www.profesor-p.com/wp-content/uploads/2018/11/github-300x74.png 300w, http://www.profesor-p.com/wp-content/uploads/2018/11/github-768x190.png 768w, http://www.profesor-p.com/wp-content/uploads/2018/11/github.png 1057w" sizes="(max-width: 1024px) 100vw, 1024px" />

&nbsp;

Después descomprimiremos el fichero ZIP dentro del directorio **themes** , de tal manera que la estructura de nuestros directorios será como la siguiente:

<img class="size-full wp-image-509 aligncenter" src="http://www.profesor-p.com/wp-content/uploads/2018/11/directorios.png" alt="" width="245" height="387" srcset="http://www.profesor-p.com/wp-content/uploads/2018/11/directorios.png 245w, http://www.profesor-p.com/wp-content/uploads/2018/11/directorios-190x300.png 190w" sizes="(max-width: 245px) 100vw, 245px" />

Suele ser una buena idea copiar el contenido del directorio `example-site` dentro de nuestro directorio principal para tener una plantilla sobre la que empezar a trabajar.

  * **Probando nuestra pagina web**

Ahora, para ver como va quedando nuestra página web, ejecutaremos este comando:

    > hugo -F serve
    

De está manera se lanzara un pequeño servidor web el cual escuchara en el puerto 1313 .

Este servidor monitorizara los cambios en los ficheros de tal manera que si modificamos cualquier fichero, nuestra pagina web será recargada automáticamente.

En la dirección <a class="url" href="http://localhost:1313/" target="_blank" rel="noopener">http://localhost:1313/</a> con nuestro navegador favorito podremos ver el resultado.

  * **Añadiendo contenido**

Si queremos añadir una entrada en la pagina web usaremos este comando:

<pre><code class="language-shell" lang="shell">&gt; hugo new post/post0.md
</code></pre>

Esto nos generara el fichero &#8216;post0.md&#8217; en el directorio **content/post/**

Este fichero tendrá el siguiente contenido.

    ---
    title: "post0" # Nombre del post
    date: 2018-11-01T11:21:58+01:00 ## fecha actual.
    draft: true # Poner a false para que aparezca el evento
    ---
    
    

Debajo deberemos poner el contenido de nuestro articulo. Recordar que se puede usar el lenguaje **markdown** y también **HTML**.

Si ponemos la etiqueta `<!--more-->`, si nuestra plantilla lo soporta, separaremos el sumario de nuestro articulo, del articulo completo en si.

  * **Configurando HUGO**

El fichero principal de Hugo esta en el directorio raíz y se llama `config.toml`

En este fichero configuraremos el tema a usar , el titulo de nuestra pagina web y otra serie de parámetros. Estos parámetros pueden variar de un tema a otro aunque siempre suele haber unos cuantos en común.

Esto es un extracto del fichero `config.toml` de la web <a class="url" href="http://www.lariojatech.org" target="_blank" rel="noopener">www.lariojatech.org</a>

    baseURL = "http://lariojatech.org/"
    languageCode = "es-es"
    title = "La Rioja Tech Alliance"
    theme = "hugo-initio"
    publishDir = "docs"
    
    [params]
        name = "La Rioja Tech Alliance"
        description = "Los grupos de tecnología de La Rioja hacemos piña para promover la tecnología "   
        favicon = "images/gt_favicon.png"
        DateForm = "2, January 2006"
    
        .....
        
    

Si queremos hacer algún cambio a la plantilla descargada deberemos cambiar los ficheros ubicados en `themes/NOMBRE_PLANTILLA/layouts/partials` . Ahí encontraremos una serie de ficheros donde se especifica el comportamiento de la plantilla.

En `themes/NOMBRE_PLANTILLA/_default` es probable que existan los ficheros list.html y single.html. En _single.html_ se definirá como se deben presentar una pagina de tipo lista, es decir cuando se deben mostrar diferentes artículos juntos. En _single.html_ se especifica como se debe presentar una pagina de un sola entrada.

  * **Generando pagina web con HTML estático**

Para generar todos los ficheros que deberán ir en el servidor web, ejecutaremos el comando

    > hugo -F
    

Los ficheros, por defecto serán generados, en el directorio **public** . Ese comportamiento se puede cambiar añadiendo la directiva `publishDir` en el fichero **config.toml**

El flag &#8220;&#8221;-F&#8221; es para obligar a Hugo a incluir los posts cuya fecha sea superior a la actual, pues por defecto no los incluye. Si no la pusiéramos todos los eventos futuros no aparecerían.

Y esto es todo, solo animaros a que utilizeis este fantastico programa con el que podreis crear paginas dinamicamente para alojarlas, por ejemplo, en GitHub. Gratuitamente por supuesto.

 [1]: https://gohugo.io/
 [2]: https://es.wikipedia.org/wiki/Markdown
 [3]: http://lariojatech.org/