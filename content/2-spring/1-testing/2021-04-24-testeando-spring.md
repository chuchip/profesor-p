---
title: Testeando Spring
pre: "<b>o </b>"
author: profesor-p
weight: 30
type: post
date: 2021-04-24
url: /spring/testeando-tu-aplicacion-web/
categories:
  - java
  - springboot
  - mockito
  - junit
  - mvc
tags:
  - java
  - springboot
  - mockito
  - junit
  - mvc
  - test
---
En este artículo muestro diferentes maneras de testear una aplicación  basada con **SpringBoot**.
 <!--more--> 

La URL del proyecto de ejemplo es esta: https://github.com/chuchip/demoSpringTest

Este programa tiene un simple controlador y una conexión con **MongoDB**. En el ejemplo uso una conexión  a Atlas. Si no conoces esta magnifica herramienta, te animo a que veas está pagina y levantes tu propia base de datos en la nube: https://cloud.mongodb.com

Este programa realiza los siguientes tipos de test:

- **DemoSpringTestApplicationTests**  que no usa **Mockito** y es un test completo

  Levanta una instancia de **MongoDB** embebida para realizar las pruebas y realiza las llamadas a nuestro controlador usando **TestRestTemplate**. En este caso las llamadas son reales, es decir se conecta por HTTP y el test es el más completo posible pues pasa por todas las partes de nuestro programa.

- **MockMvcTestApplication** 

  Las peticiones web son realizadas a través de **MockMVC** y por lo tanto vamos a la URL del controlador. Se levanta una instancia embebida de **MongoDB**. También es un test completo.

-  **MockitoTestApplication**:

  Es el test mas simple y también el más rápido. Usando **Mockito** no se levanta la aplicación de **SpringBoot** ni usa **MongoDB**.  Las peticiones al controlador se hacen llamando a las funciones, por lo cual no se comprueban las URL.

- **MockitoWebTestApplication**:

  Lanza la aplicación entera, es decir se levanta el servidor web pero usamos **Mockito**  para crear el controlador y el repositorio de **MongoDB**. La base de datos en este caso se levanta pero no la usamos ya que mockeamos el repositorio.

  

No os voy a dar mucho la chapa de como funcionan las diferentes clases,  he preferido comentar el código que creo que se explica el mismo y dejar, al que este interesado, que mire y juegue.

