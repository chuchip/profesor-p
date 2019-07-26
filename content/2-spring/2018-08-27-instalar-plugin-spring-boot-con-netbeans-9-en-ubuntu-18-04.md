---
title: Instalar plugin Spring Boot en NetBeans
pre: "<b>o </b>"
author: airec69
type: post
date: 2018-08-27
url: /2018/08/27/instalar-plugin-spring-boot-con-netbeans-9-en-ubuntu-18-04/
categories:
  - java
  - netbeans
  - spring
  - spring boot
tags:
  - java
  - linux
  - netbeans
  - spring boot

---
En esta entrada explicare como crear un aplicación con el framework Spring Boot, usando el IDE NetBeans 9.

Ademas haremos todo esto en un Linux Ubuntu 18.04

Si no sabéis lo que es Spring Boot, ya estáis tardando en aprender al menos los conceptos básicos.

  * Spring Boot es Spring hecho (más) fácil.
  *  Con Spring Boot, entre otras cosas, se puede realizar una aplicación Web, con una mínima configuración. Una aplicación _demo_ se crea en apenas  5 minutos. Y ademas, sin tener que usar un servidor de aplicaciones, pues Spring Boot, ya incluye una versión de Tomcat embebida.
  * Spring Boot usa un sistema de paquetes, que hace mas fácil las dependencias de Maven (o Gradle).
  * Spring Boot, la versión 5, funciona con Java 1.8 o superior, haciendo mucho uso de Lambdas, Streams y todas las nuevas capacidades de este lenguaje.

Este es un video con una charla de introducción, de la gente de Paradigma Digital.

<p><iframe src="https://www.youtube.com/embed/4HGGRLigGTM" allow="autoplay; encrypted-media" allowfullscreen="" width="990" height="557" frameborder="0"></iframe></p>

Ya se que Spring tiene su propio IDE, basado en Eclipse, pero la ultima versión al menos, esta tan pensada para Spring Boot, que intentar utilizar otro servidor de aplicaciones es muy engorroso.  Y bueno, que hay gente que nos sentimos cómodos con NetBeans y no nos apetece cambiar de IDE 🙂

Pero si os apetece probar un IDE, totalmente configurado para  Spring Boot , usar **Spring Tool Suite** bajándolo de <a href="https://spring.io/tools/sts" target="_blank" rel="noopener">https://spring.io/tools/sts</a>. No es mal IDE, pero &#8230;. no es mi IDE 😉

Bueno, lo primero es tener NetBeans 9 instalado, para ello, lo primero seria ir a la siguiente página: <a href="https://netbeans.apache.org/download/nb90/nb90.html" target="_blank" rel="noopener">https://netbeans.apache.org/download/nb90/nb90.html</a> y bajarnos el IDE.

Una vez tengamos el IDE descargado, como es un ZIP, simplemente lo descomprimimos. En este ejemplo voy a suponer que lo instalamos en **/nb** (sí, ya se que no es lo normal, pero es para hacer mas fácil el tutorial).

Para ejecutar NetBeans 9, deberemos ir al directorio bin y ejecutar el fichero netbeans.sh. Es decir en nuestro ejemplo deberemos ejecutar **/nb/bin/netbeans.sh**.

Un momento,  ¿ que versión de Java tenéis instalado en vuestro Ubuntu?.

Me temo que NetBeans solo funciona con el Java de Oracle. OpenJDK no le gusta y no os arrancara, así que no os queda otra  (que yo sepa) que iros a la pagina web de Oracle y bajaros su JDK. En este caso yo me he bajado JDK 10.0.2. Bajaros el <a href="http://www.oracle.com/technetwork/java/javase/downloads/jdk10-downloads-4416644.html" target="_blank" rel="noopener">tar.gz de este enlace</a> y lo descomprimís  en el directorio **/nb** (podríamos dejarlo en cualquier otro lugar pero es por facilitar el tutorial una vez mas). Así, ahora, tendréis el JDK en **/nb/jdk-10.0.2/**

Ahora hay que decirle a NetBeans que use el nuevo JDK que hemos _instalado,_ para ello deberéis editar el fichero situado  en **/nb/etc/netbeans.conf **y cambiar la linea **netbeans_jdkhome ** para que ponga esto:

<pre>netbeans_jdkhome="/nb/jdk-10.0.2/"</pre>

Ok. Ya podemos ejecutar Netbeans, con el comando **/nb/bin/netbeans.sh**

La primera vez que ejecutamos NetBeans 9, nos solicita que instalemos la librería nbjavac, para mejorar el editor de textos y la funcionalidad en general. Le hacemos caso y la instalamos.

![](/img/2018/08/Captura-1024x680.png]

Una vez instalada, nos pedirá que reiniciemos el IDE. Somos buenos chicos y lo reiniciamos.

Ahora toca instalar el plugin para poder funcionar con Spring Boot. Para ello nos vamos al menú **Tools** y elegimos la opción **plugins**. En la ventana que nos sale nos iremos a la pestaña de &#8220;**Available Plugins**&#8221;  y elegiremos **NB SpringBoot.** Después le daremos al botón Install.

&nbsp;

<img class="alignnone size-full wp-image-124" src="http://www.profesor-p.com/wp-content/uploads/2018/08/Captura2.png" alt="" width="934" height="586" srcset="http://www.profesor-p.com/wp-content/uploads/2018/08/Captura2.png 934w, http://www.profesor-p.com/wp-content/uploads/2018/08/Captura2-300x188.png 300w, http://www.profesor-p.com/wp-content/uploads/2018/08/Captura2-768x482.png 768w" sizes="(max-width: 934px) 100vw, 934px" />

Reiniciamos, una vez más nuestro IDE, y ya podremos crear nuestro proyecto Spring Boot, en NetBeans 9. Como crearlo lo contare en la próxima entrada.