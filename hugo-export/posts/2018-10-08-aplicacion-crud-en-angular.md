---
title: Aplicación CRUD en Angular
author: El Profe
type: post
date: 2018-10-08T20:23:38+00:00
url: /2018/10/08/aplicacion-crud-en-angular/
categories:
  - angular
  - json
  - rest
tags:
  - angular
  - frontend
  - json
  - rest

---
Buenas chavales. En esta ocasión os traigo un programa realizado en Angular, el cual a través de peticiones REST, da de alta, modifica, borra  y consulta los diferentes países disponibles en una base de datos. Lo que se viene diciendo una aplicacion CRUD.

Por supuesto el protocolo para las comunicaciones es JSON.

El código fuente lo tenéis en: <a href="https://github.com/chuchip/crud-rest-angular" target="_blank" rel="noopener">https://github.com/chuchip/crud-rest-angular</a>

Este programa es el frontend del realizado en JavaEE, y que explico como funciona en las entradas: <a href="http://www.profesor-p.com/2018/10/06/aplicacion-rest-en-javaee/" target="_blank" rel="noopener">http://www.profesor-p.com/2018/10/06/aplicacion-rest-en-javaee/</a> y  <a href="http://www.profesor-p.com/2018/10/07/aplicacion-rest-en-javaee-2a-parte/" target="_blank" rel="noopener">http://www.profesor-p.com/2018/10/07/aplicacion-rest-en-javaee-2a-parte/</a>

Es decir, este programa realizara sera el cliente  que correrá en el navegador de los usuarios y el anterior programa estará corriendo en un servidor y tendrá acceso a la base de datos correspondiente, para actualizarla según las peticiones REST realizadas por el cliente.

Esto es un pantallazo de como queda en modo consulta.

<img class="imagen_con_borde aligncenter wp-image-357 size-full" src="http://www.profesor-p.com/wp-content/uploads/2018/10/Captura-4.png" alt="" width="700" height="809" srcset="http://www.profesor-p.com/wp-content/uploads/2018/10/Captura-4.png 700w, http://www.profesor-p.com/wp-content/uploads/2018/10/Captura-4-260x300.png 260w" sizes="(max-width: 700px) 100vw, 700px" />Y este otro, un pantallazo del alta:

<img class="imagen_con_borde aligncenter wp-image-363 size-full" src="http://www.profesor-p.com/wp-content/uploads/2018/10/Captura1.png" alt="" width="540" height="431" srcset="http://www.profesor-p.com/wp-content/uploads/2018/10/Captura1.png 540w, http://www.profesor-p.com/wp-content/uploads/2018/10/Captura1-300x239.png 300w" sizes="(max-width: 540px) 100vw, 540px" />

Os invito a que echéis un vistazo al programa, sobre todo al fichero **datosserver.service.ts** que es donde se se hacen las diferentes peticiones GET, POST, PUT y DELETE para interactuar con el servidor .

Si veo interés ya explicare con detalle como funciona.

Espero vuestros comentarios y sugerencias.

¡¡ Hasta pronto!!