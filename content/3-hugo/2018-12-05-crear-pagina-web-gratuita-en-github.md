---
title: Crear pagina web gratuita en GitHub
author: El Profe
pre: "<b>o </b> "
weight: 2
type: post
date: 2018-12-05T10:32:08+00:00
url: /2018/12/05/crear-pagina-web-gratuita-en-github/
categories:
  - html
  - web
tags:
  - html
  - web

---
Buenas. Alumnos,

En esta entrada explicare como crear una página  web gratuita  en [GitHub][1]

Estas  páginas deben ser estaticas  pero no hay limitaciones de ancho de banda, numero de ficheros y por supuesto no meteran publicidad en vuestra pagina web. Ademas, utilizara  https y podreis usar un dominio propio, si lo habeis comprado con anterioridad. Aclarar que un dominio se puede registrar por solo un euro al año 😉
<!-- more -->

Además podeis usar [Hugo][2] como explicaba en [una anterior entrada][3] para hacer vuestro sitio web estático, más dinamico.

Eso sí, para usar GitHub debeis tener al menos ciertas nociones basicas de [Git][4] pues se utilizara esta herramienta para subir los ficheros que formaran la pagina web.

Lo primero es crear nuestro proyecto en [GitHub][1], para lo cual deberemos estar registrados.

Teneis muchos videos y manuales que explican como trabajar con GitHub, uno donde se explica incluso como crear una  página web lo teneis en [devCode.la][5]

Por si os sirve de ayuda he creado un proyecto en <https://github.com/chuchip/web> que genera la pagina web <https://chuchip.github.io/web/>

Aclarar que una vez tengamos creado nuestro proyecto, deberemos hacer al menos un **push**. Es decir subir a **GitHub** al menos un fichero.

Ahora, para especificar que queremos tener una pagina web para nuestro proyecto, iremos a &#8220;**Settings**&#8220;

![Settings en GithHub][6]

y bajaremos hasta que veamos la sección: **GitHub Pages**

![Settings en GithHub][7]

Ahora eligiremos el branch (es decir, la rama) donde estara nuestra pagina web. Para empezar simplemente usar el valor que aparece por defecto: **master branch**. El tema de _&#8216;ramas&#8217;_ es ampliamente utilizado en GIT pero si solo queremos hacer una pagina web no necesitamos crear ninguna, por lo cual deberemos elegir la rama &#8216;**master&#8217;** (que sera la única que exista)

Ahora deberemos elegir que _tema_ vamos a utilizar para nuestra pagina web. Aunque tenemos la opción de no utilizarlo como explico más adelante, es **obligatorio** elegir un _tema_.

Tenemos la opción de tener un dominio propio tipo **<a class="url" href="http://www.midominio.com" target="_blank" rel="noopener">www.midominio.com</a>**, el cual, logicamente, deberemos tener registrado previamente. Si no tenemos un dominio propio nuestra pagina web sera visible bajo el dominio: <a class="url" href="http://USUARIO.github.com/PROYECTO" target="_blank" rel="noopener">http://USUARIO.github.com/PROYECTO</a>

Tenemos dos formas de crear nuestra pagina web

  * Utilizando un _tema_
  * Sin _tema_ para lo cual usaremos un fichero **index.html**.

### * Trabajando con un tema

Si elegimos usar un _tema_ lo que se mostrara sera el fichero **readme.md** que tengamos en nuestro proyecto, maquetado según el _tema_ elegido. En este fichero **readme.md** podremos usar el lenguaje de maquetado [MarkDown][8] con el que podremos poner imagenes, formatear nuestro texto con negritas, cursiva, etc,  poner enlaces, etc.

Para poner enlaces a las imagenes se podra usar el formato: `./IMAGEN_A_ENLAZAR`

### * Pagina web estatica

Si hemos creado un fichero **index.html**, este fichero se utilizara como página inicial de nuestra página web.

Todas las referencias a imagenes queramos poner deberan ser con el formato:

`https://raw.githubusercontent.com/USUARIO/web/master/IMAGEN_A_MOSTRAR`

Es decir si el usuario es &#8216;**chuchip**&#8216; y el proyecto se llama &#8216;**web&#8217;**, todas las referencias a imagenes deberan ser con la URL:

<a class="url" href="https://raw.githubusercontent.com/chuchip/web/master/" target="_blank" rel="noopener">https://raw.githubusercontent.com/chuchip/web/master/</a>

Para las referencias a paginas web, _CSS_ y ficheros javascript podremos usar el formato **&#8220;./PAGINA_XX.html&#8221;**

### * Notas

Aclarar que si tenemos el fichero **index.html** no utilizara el _tema_ ni se vera el fichero **readme.md**

Otra cosa a tener en cuenta que a GitHub le cuesta un poco (menos de un minuto), refrescar la pagina web desde el ultimo _push_

Y con esto ya tendreis vuestra pagina web alojada gratuitamente.

¡¡ Hasta otra!!

 [1]: https://www.github.com
 [2]: https://gohugo.io/
 [3]: http://www.profesor-p.com/2018/11/28/generar-paginas-web-estaticas-dinamicamente/
 [4]: https://git-scm.com/book/es/v2
 [5]: https://devcode.la/tutoriales/publicar-tu-web-usando-github-pages/
 [6]: https://raw.githubusercontent.com/chuchip/web/master/_captura1.png
 [7]: https://raw.githubusercontent.com/chuchip/web/master/_captura2.png
 [8]: https://es.wikipedia.org/wiki/Markdown