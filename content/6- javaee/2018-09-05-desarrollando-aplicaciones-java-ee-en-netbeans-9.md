---
title: Desarrollando aplicaciones Java EE en NetBeans 9
author: El Profe
type: post
date: 2018-09-05T15:05:08+00:00
url: /2018/09/05/desarrollando-aplicaciones-java-ee-en-netbeans-9/
categories:
  - java
  - netbeans
  - tomcat
tags:
  - java
  - Java ee
  - netbeans9
  - tomcat

---
El recién salido <a href="https://netbeans.apache.org/download/nb90/nb90.html" target="_blank" rel="noopener">NetBeans 9</a>, es un excelente IDE, con soporte para Java 10 y 11, y con otra serie de características muy interesantes. Sin embargo, por temas de licencias con Oracle que es el dueño del antiguo NetBeans, solo tiene soporte para  Java Standard Edition, no pudiendo, en teoría, hacer aplicaciones para Java EE.
<!--more-->

Pero es solo en teoria, pues NetBeans 9, mantiene la compatibilidad con la versión 8.2, incluso a nivel de plugins. Así que podemos añadir las fuentes de la versión anterior y podremos crear aplicaciones Java EE.

Eso es tan simple como ir al menú **Tools** y elegir la opción **Plugins**.  En la ventana de plugins, iremos a la pestaña **settings** y pulsaremos el botón **Add.** El cual nos mostrara la siguiente ventana:

<img class="size-full wp-image-212 aligncenter" src="http://www.profesor-p.com/wp-content/uploads/2018/09/netbeans9-2.png" alt="" width="870" height="432" srcset="http://www.profesor-p.com/wp-content/uploads/2018/09/netbeans9-2.png 870w, http://www.profesor-p.com/wp-content/uploads/2018/09/netbeans9-2-300x149.png 300w, http://www.profesor-p.com/wp-content/uploads/2018/09/netbeans9-2-768x381.png 768w" sizes="(max-width: 870px) 100vw, 870px" />

Ahí pondremos rellenaremos los 2 campos, de esta manera:

> Name: NetBeans 8.2
> 
> URL: http://updates.netbeans.org/netbeans/updates/8.2/uc/final/distribution/catalog.xml.gz

Después, volveremos de darle al botón OK, volveremos a añadir una nueva entrada con estos datos:

> Name: Plugin Portal
> 
> URL: http://plugins.netbeans.org/nbpluginportal/updates/9.0/catalog.xml.gz

La pagina debe quedar como la siguiente.

<img class="size-large wp-image-213 aligncenter" src="http://www.profesor-p.com/wp-content/uploads/2018/09/netbeans9-1.png" alt="" width="883" height="424" srcset="http://www.profesor-p.com/wp-content/uploads/2018/09/netbeans9-1.png 883w, http://www.profesor-p.com/wp-content/uploads/2018/09/netbeans9-1-300x144.png 300w, http://www.profesor-p.com/wp-content/uploads/2018/09/netbeans9-1-768x369.png 768w" sizes="(max-width: 883px) 100vw, 883px" />

A partir de ahí, ya veremos que nos aparecen como disponibles muchos más plugins, entre los que veremos Java EE Base. Una vez instalado este plugin ya podremos realizar nuestras aplicaciones Java EE.

¡¡¡ A seguir estudiando!!!