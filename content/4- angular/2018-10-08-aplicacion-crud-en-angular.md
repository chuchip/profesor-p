---
title: Aplicación CRUD en Angular
pre: "<b>o </b>"
weight: 3
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
En esta ocasión os traigo un programa realizado en Angular, el cual a través de peticiones REST, da de alta, modifica, borra  y consulta los diferentes países disponibles en una base de datos. Lo que se viene diciendo una aplicacion CRUD.

Por supuesto el protocolo para las comunicaciones es JSON.

El código fuente lo tenéis en: <a href="https://github.com/chuchip/crud-rest-angular" target="_blank" rel="noopener">https://github.com/chuchip/crud-rest-angular</a>

Este programa es el frontend del realizado en JavaEE, y que explico como funciona en las entradas: <a href="http://www.profesor-p.com/2018/10/06/aplicacion-rest-en-javaee/" target="_blank" rel="noopener">http://www.profesor-p.com/2018/10/06/aplicacion-rest-en-javaee/</a> y  <a href="http://www.profesor-p.com/2018/10/07/aplicacion-rest-en-javaee-2a-parte/" target="_blank" rel="noopener">http://www.profesor-p.com/2018/10/07/aplicacion-rest-en-javaee-2a-parte/</a>

Es decir, este programa realizara sera el cliente  que correrá en el navegador de los usuarios y el anterior programa estará corriendo en un servidor y tendrá acceso a la base de datos correspondiente, para actualizarla según las peticiones REST realizadas por el cliente.

Esto es un pantallazo de como queda en modo consulta.

![](/img/2018/10/Captura-4.png)

Y este otro, un pantallazo del alta:

![](/img/2018/10/Captura1.png)

Os invito a que echéis un vistazo al programa, sobre todo al fichero **datosserver.service.ts** que es donde se se hacen las diferentes peticiones GET, POST, PUT y DELETE para interactuar con el servidor .

Si veo interés ya explicare con detalle como funciona.

Espero vuestros comentarios y sugerencias.

¡¡ Hasta pronto!!