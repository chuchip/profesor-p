---
title: Angular 6 – Añadir Bootstrap 4 con JQuery
author: El Profe
type: post
date: 2018-09-20T09:16:12+00:00
url: /2018/09/20/angular-6-anadir-bootstrap-4-con-jquery/
categories:
  - bootstrap
  - jquery
  - pooper
tags:
  - angular
  - bootstrap
  - jquery
  - pooper

---
Recordar lo primero que para utilizar toda la potencia de <a href="http://getbootstrap.com/" target="_blank" rel="noopener noreferrer">BootStrap</a>, necesitamos tener instaladas las librerias de <a href="https://jquery.com/" target="_blank" rel="noopener noreferrer">JQuery</a> y <a href="https://popper.js.org/index.html" target="_blank" rel="noopener noreferrer">Popper</a> <img class="alignright wp-image-326 size-medium" src="http://www.profesor-p.com/wp-content/uploads/2018/09/bootstrap-stack-300x252.png" alt="" width="300" height="252" srcset="http://www.profesor-p.com/wp-content/uploads/2018/09/bootstrap-stack-300x252.png 300w, http://www.profesor-p.com/wp-content/uploads/2018/09/bootstrap-stack-768x645.png 768w, http://www.profesor-p.com/wp-content/uploads/2018/09/bootstrap-stack.png 1024w" sizes="(max-width: 300px) 100vw, 300px" />

Hay varias maneras de  instalar estas librerías.

  1. <span style="font-size: 18pt;">Localmente usando npm</span>

Una vez estemos situados el directorio principal del proyecto, desde tu terminal preferido ejecutar las instrucciones siguientes:

<pre>npm install  bootstrap@4 jquery popper.js --save</pre>

Esto nos instalara los archivos necesarios bajo el directorio **node-modules. **

Ahora debemos incluir el fichero de estilos (css) de BootStrap y las librerías JavaScript de BootStrap, JQuery y Pooper.

Una de las opciones es editando el fichero **angular.json,** e incluirlas en los campos styles y scripts.

El fichero quedaría algo así como esto:

<pre><strong>...... CODIGO ANTERIOR ....</strong>
"styles": [
"src/styles.css",
"./node_modules/bootstrap/dist/css/bootstrap.min.css"
],

"scripts": [
"./node_modules/jquery/dist/jquery.min.js",
"./node_modules/bootstrap/dist/js/bootstrap.min.js",
"./node_modules/pooper.js/dist/js/pooper.min.js",
]
<strong>...... CODIGO POSTERIOR  ....</strong></pre>

Recordar que hay que parar el servidor de **node.js** y volverlo a ejecutar con **ng serve **si modificamos el fichero **angular.json**

Otra manera de incluir estos ficheros seria añadirlos en **index.html**, dentro de **HEAD**. Esto si fuera un proyecto habitual en HTML.

<div>
  <pre>....
&lt;head&gt;
 &lt;link rel="stylesheet" href="../node_modules/bootstrap/dist/css/bootstrap.min.css"&gt;
 &lt;script src="../node_modules/jquery/dist/jquery.min.js"&gt;&lt;/script&gt;
 &lt;script src="../node_modules/bootstrap/dist/js/bootstrap.min.js"&gt;&lt;/script&gt;
 &lt;script src="../node_modules/pooper.js/dist/js/pooper.min.js"&gt;&lt;/script&gt;
&lt;/head&gt;
....</pre>
</div>

  1. <span style="font-size: 18pt;">Usando CDN</span>

En este caso, no bajamos nada locamente y lo que hacemos es incluir los enlaces hacia los correspondientes ficheros en interner.

Así, como explican en la pagina de  <a href="http://getbootstrap.com/docs/4.1/getting-started/download/#bootstrapcdn" target="_blank" rel="noopener noreferrer">BootStrap CDN</a>, añadiríamos esta linea para incluir bootstrap versión 4.1.3

<pre><code class="language-html" data-lang="html">&lt;span class="nt">&lt;link&lt;/span> &lt;span class="na">rel=&lt;/span>&lt;span class="s">"stylesheet"&lt;/span> &lt;span class="na">href=&lt;/span>&lt;span class="s">"https://stackpath.bootstrapcdn.com/bootstrap/4.1.3/css/bootstrap.min.css"&lt;/span> &lt;span class="na">integrity=&lt;/span>&lt;span class="s">"sha384-MCw98/SFnGE8fJT3GXwEOngsV7Zt27NXFoaoApmYm81iuXoPkFOJwJ8ERdknLPMO"&lt;/span> &lt;span class="na">crossorigin=&lt;/span>&lt;span class="s">"anonymous"&lt;/span>&lt;span class="nt">&gt;&lt;/span>
&lt;span class="nt">&lt;script &lt;/span>&lt;span class="na">src=&lt;/span>&lt;span class="s">"https://stackpath.bootstrapcdn.com/bootstrap/4.1.3/js/bootstrap.min.js"&lt;/span> &lt;span class="na">integrity=&lt;/span>&lt;span class="s">"sha384-ChfqqxuZUCnJSK3+MXmPNIyE6ZbWh2IMqE241rYiqJxyMiZ6OW/JmZQ5stwEULTy"&lt;/span> &lt;span class="na">crossorigin=&lt;/span>&lt;span class="s">"anonymous"&lt;/span>&lt;span class="nt">&gt;&lt;/script&gt;&lt;/span></code></pre>

Y esta linea para incluir JQuery y Pooper

<pre><code class="language-html" data-lang="html">&lt;span class="nt">&lt;script &lt;/span>&lt;span class="na">src=&lt;/span>&lt;span class="s">"https://code.jquery.com/jquery-3.3.1.slim.min.js"&lt;/span> &lt;span class="na">integrity=&lt;/span>&lt;span class="s">"sha384-q8i/X+965DzO0rT7abK41JStQIAqVgRVzpbzo5smXKp4YfRvH+8abtTE1Pi6jizo"&lt;/span> &lt;span class="na">crossorigin=&lt;/span>&lt;span class="s">"anonymous"&lt;/span>&lt;span class="nt">&gt;&lt;/script&gt;&lt;/span>
&lt;span class="nt">&lt;script &lt;/span>&lt;span class="na">src=&lt;/span>&lt;span class="s">"https://cdnjs.cloudflare.com/ajax/libs/popper.js/1.14.3/umd/popper.min.js"&lt;/span> &lt;span class="na">integrity=&lt;/span>&lt;span class="s">"sha384-ZMP7rVo3mIykV+2+9J3UJ46jBk0WLaUAdn689aCwoqbBJiSnjAK/l8WvCWPIPm49"&lt;/span> &lt;span class="na">crossorigin=&lt;/span>&lt;span class="s">"anonymous"&lt;/span>&lt;span class="nt">&gt;&lt;/script&gt;
&lt;/span></code></pre>

<div>
  <p>
    ¡¡Y a trabajar con BootStrap y crear aplicaciones profesionales!!
  </p>
</div>