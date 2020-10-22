---
title: Logging en Spring Boot
pre: "<b>o </b>"
author: airec69
type: post
weight: 300
date: 2020-10-18T07:57:07+00:00
url: /springboot/logging-en-spring-boot/
categories:
  - logging
  - java
  - mvn
  - spring boot
tags:
  - logging
  - java
  - mvn
  - spring boot

---
Una vez que tenemos nuestra fantástica aplicación realizada con Spring lo normal es que queramos ver como se va comportando. Para ello, la manera más fácil es escribir mensajes dentro de la aplicación, explicando por que funciones entra, como se toman las decisiones y, en general,  como se va comportando.

<!--more-->

### Redirigiendo la salida.

Spring Boot, como cualquier aplicación, por defecto sacara todos los mensajes por la salida estándar y/o errores, pero a menudo deseamos guardar esos mensajes en un fichero. Para ello hay varias opciones. La primera y mas obvia es redirigir salida estándar y errores a un fichero. Así, en Linux y/o Windows, podríamos ejecutar nuestra aplicación de esta manera:

>  java -jar APP.JAR > PATH/FICHERO.LOG 2>&1

Esto, obviamente funciona, pero tendremos que mirar en el fichero generado para ver como va todo, sin posibilidad de ver inmediatamente en nuestra consola como ha ido el proceso recién lanzado.

Si lo que deseamos es que deje la salida estándar en un fichero, pero, que además, nos muestre en la consola  esos misma salida podremos lanzar el proceso de esta manera:

> java -jar -Dlogging.file.name=PATH/FICHERO.LOG APP.JAR

Por supuesto también podemos poner en nuestro fichero de configuración esa misma propiedad y podremos lanzar el proceso sin el parámetro '-D'

**Fichero**:`application.properties`

```java
logging.file.name=PATH/FICHERO.LOG APP.JAR
```

Si queremos ejecutar el proceso desde maven pondremos este comando:

> mvn spring-boot:run -Dspring-boot.run.arguments=--logging.file.name=PATH/FICHERO.LOG



### Configurando nivel de log

Por supuesto los mensajes que deseamos mostrar los podemos escribir con un simple `system.out.print` pero desde luego no sería muy elegante ni practico.  Siempre es recomendable usar un logger.

Spring usa  [Commons Logging](https://commons.apache.org/logging),  usando por defecto [Logback](https://logback.qos.ch/),  si bien incluye también configuraciones por defecto para r [Java Util Logging](https://docs.oracle.com/javase/8/docs/api//java/util/logging/package-summary.html) y [Log4J2](https://logging.apache.org/log4j/2.x/).

Por defecto Spring Boot, muestra los mensajes de nivel `ERROR`-level, `WARN`-level, and `INFO`-level pero esto se puede cambiar de diferentes maneras.

La mas simple es añadiendo el parámetro `--trace` o  `--debug `cuando ejecutemos nuestro programa.

> java -jar  APP.JAR --debug



Otra manera será especificar en la propiedad 'logging.level.root'. Esto lo haremos  añadiremos la línea correspondiente en el fichero `application.properties` o a la hora de ejecutarlo, añadiendo el parametro adecuado:

>  java -jar  -Dlogging.level.root=DEBUG APP.JAR 



Una vez más si usamos maven para ejecutar el proceso podremos poner el siguiente comando:

> mvn spring-boot:run 
> 	  -Dspring-boot.run.arguments=-logging.level.root=TRACE



Por supuesto, se permite especificar niveles de *logging* por paquetes, así, si queremos que nuestra aplicación que esta en el paquete 'com.profesorp.miaplicacion' muestre los mensajes de nivel *debug*, pero el resto de aplicaciones (que os recuerdo también tendrán sus salidas a través del log) solo muestren los mensajes de error, escribiremos esta sentencia:

> java -jar -Dlogging.level.root=ERROR -D-Dlogging.level.com.propesorp.miaplicacion=DEBUG APP.JAR 

### Formateando la salida

Spring por defecto muestra los mensajes de error con este formato:

```
2019-03-05 10:57:51.112  INFO 45469 --- [           main] org.apache.catalina.core.StandardEngine  : Starting Servlet Engine: Apache Tomcat/7.0.52
2019-03-05 10:57:51.253  INFO 45469 --- [ost-startStop-1] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
```

Es decir: 

- Día y hora, incluyendo milisegundos.
- Nivel de Log:  `ERROR`, `WARN`, `INFO`, `DEBUG`, or `TRACE`.
- Identificador o número del proceso
- Un `---` como separador
- El nombre del Thread entre corchetes.
- El nombre de la clase donde se esta escribiendo el log.
- El mensaje de log.



También podemos especificar que use colores para cuando  muestre en la consola los logs, con la siguiente variable: `spring.output.ansi.enabled=true`. Así, los mensajes de error los mostrara en ROJO, los de aviso (WARN) en amarillo y los demás, en verde.

Este formato puede ser cambiado con la variable `logging.pattern.console` para la salida de consola y con `logging.pattern.file` para la salida a fichero. Un ejemplo configuración seria este:

```
# Salida a consola.
logging.pattern.console= %d{yyyy-MM-dd HH:mm:ss} - %logger{36} - %msg%n
# Salida a consola con colores.
logging.pattern.console=%clr(%d{yy-MM-dd E HH:mm:ss.SSS}){blue} %clr(%-5p) %clr(${PID}){faint} %clr(---){faint} %clr([%8.15t]){cyan} %clr(%-40.40logger{0}){blue} %clr(:){red} %clr(%m){faint}%n

# Salida a un fichero
logging.pattern.file= %d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{36} - %msg%
```



El formato de estas variables es un poco complicado y os aconsejo ver [este enlace](https://logback.qos.ch/manual/layouts.html#ClassicPatternLayout) si queréis modificarlo.

### Configuración avanzada 

Si Spring Boot encuentra en el *classpath*  un fichero con alguno de estos nombres, lo usara para configurar los loggers.

- *logback-spring.xml*
- *logback.xml*
- *logback-spring.groovy*
- *logback.groovy*

Spring recomienda que usemos la variante -spring como se describe [aquí](https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-logging.html#boot-features-custom-log-configuration).

### Enlaces a más documentación

Y para finalizar os dejo unos enlaces con más documentación en ingles, sobre logging:

https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-logging

https://www.baeldung.com/spring-boot-logging

https://howtodoinjava.com/spring-boot2/logging/spring-boot-logging-configurations/





