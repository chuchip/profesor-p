---
title: Backend I
weight: 10
pre: "<b>o </b>"
author: airec69
type: post
date: 2018-08-31T05:53:48+00:00
url: /2018/08/31/aplicacion-en-spring-y-angular/
categories:
  - angular
  - java
  - jndi
  - jpa
  - spring
tags:
  - angular
  - java
  - spring

---
En <a href="http://anjelica.sf.net" target="_blank" rel="noopener">Anjelica</a>, el ERP desarrollado por mi hace unos años, hay un programa que saca un comparativo de ventas, entre dos años, mostrando mes a mes, los kilos, importes y ganancias.

<img class="alignnone size-full wp-image-150" src="http://www.profesor-p.com/wp-content/uploads/2018/08/cohive.png" alt="" width="996" height="578" />

La idea es realizar esta misma consulta pero usando Spring con REST en el backend y como frontend usar Angular.

Para ello se realizaran los siguientes pasos:

  1.  Crear tablas y cargar datos de prueba en la base de datos. Usare H2 embebido, en vez de postgresql, que es la base de datos que usa **Anjelica.**
  2. Crear aplicación servidor, bajo TOMCAT 9, usando Spring 5.  No usare Spring Boot para este ejemplo, aunque podría hacerlo perfectamente y de hecho haría la aplicación más sencilla.
  3. Crear aplicación cliente, para lo cual usare Angular 6.

La aplicación, como se ve en el anterior pantallazo, nos mostrara las ventas por meses, haciendo un comparativo del ejercicio introducido contra el anterior. Pulsando en cada mes, veremos las ventas por semanas.

Para empezar vamos a definir las tablas que usaremos en esta consulta:

<pre>--
-- Tabla calendarios
--
create table calendario
(
cal_ano int, -- Año
cal_mes int, -- Mes
cal_fecini date, -- Fecha Inicial
cal_fecfin date, -- Fecha Final.
constraint ix_calendario primary key (cal_ano,cal_mes)
);
create table histventas
(
hve_fecini date not null, -- Fecha Inicial
hve_fecfin date not null, -- Fecha Final
hve_kilven float, -- Kilos Venta (Total)
hve_impven float, -- Importe Ventas (Total)
hve_impgan float, -- Importe ganancia
div_codi int not null default 1, -- Divisa
constraint ix_histventas primary key (hve_fecini ,hve_fecfin,div_codi  )
);</pre>

Como se ve, tenemos una tabla **calendario** donde definimos, para cada mes, cual es la fecha inicial y fecha final. Esto es porque nosotros definimos cuando empieza un mes y cuando termina, la razón de hacerlo así es porque, en muchas empresas se vende por semanas enteras y como queremos hacer comparativos nos puede interesar especificar que semanas consideramos que son de un mes.

Así, en los datos que cargaremos, el mes de enero del 2018, empieza el día 1/1 (lunes) y termina el día 27/1 sábado, y el mes de febrero empieza el 28/1 y termina el 24/2. Es decir en nuestra empresa ficticia la semana comienza el domingo y termina el sábado siguiente (ambos inclusive). Excepto al principio del año, que empieza siempre el 1/1 y termina el 31/12.

En la tabla **histventas** tenemos una serie de acumulados para los periodos marcados por **hve_fecini** (fecha Inicio) y **hve_fecfin**  (Fecha Final) y para una divisa dada.Esta tabla no tiene un indice único, pero buscaremos por los datos por esos 3 campos

Esta tabla la llena un proceso automático del ERP, a través de las tablas de ventas, pero en este ejemplo no explicare como se llenan esos datos, sino que precargare unos datos en la propia aplicación, cuando se instancie la base de datos.

En la siguiente entrada, empezare con el programa de la parte del servidor, pero podéis echarle un vistazo al código en <a href="https://github.com/chuchip/yagesserver.git" target="_blank" rel="noopener">https://github.com/chuchip/yagesserver.git</a>

&nbsp;

¡¡ Nos vemos pronto!!

&nbsp;

&nbsp;

&nbsp;