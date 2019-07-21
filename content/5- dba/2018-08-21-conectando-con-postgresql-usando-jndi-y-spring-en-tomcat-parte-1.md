---
title: Conectando a una base de datos usando JNDI y Spring en Tomcat
author: airec69
type: post
date: 2018-08-21T13:34:56+00:00
url: /2018/08/21/conectando-con-postgresql-usando-jndi-y-spring-en-tomcat-parte-1/
featured_image: /wp-content/uploads/2018/08/springsource.jpg
cwp_meta_box_check:
  - No
categories:
  - java
  - jndi
  - spring
  - tomcat
tags:
  - hibernate
  - java
  - jndi
  - jpa
  - spring
  - tomcat

---
# 1. Introducción

En este ejemplo veremos como conecta a Postgresql usando JPA + Hibernate y Spring JDBC. Para ello uso como servidor de aplicaciones Tomcat. La configuración esta realizada con anotaciones y XML y utilizo el pool de conexiones de Tomcat recogido a través de JNDI, para que la aplicación no deba saber realmente ni donde se conecta.

El código fuente lo teneis en: <a href="https://github.com/chuchip/jdbc_jpa_tomcat" target="_blank" rel="noopener">https://github.com/chuchip/jdbc_jpa_tomcat</a>

En <a href="http://www.profesor-p.com/wp-content/uploads/2018/08/ejemplo-jpa-y-jdbc-con-spring.pdf" target="_blank" rel="noopener">este enlace</a> teneis este mismo documento (todas las partes) pero en PDF.

El ejemplo usa Maven y explicare como deberá estar configurado Tomcat para que la aplicación funcione correctamente.

# 2. Configuración de Postgresql {.western}

Esta sera la única tabla a la que accederemos a través de Postgresql:

<pre>create table usuario
(
	login varchar(15) not null,
	nombre varchar(100) not null,
	constraint ix_usuario primary  key (login)
);
insert into  usuario values('cpuente','El profe');
insert into  usuario values('chuchi','El nombre del profe');
</pre>

# 3. Configuración de Tomcat

La configuración de Tomcat deberá tener las siguientes características:

En server.xml (estara en $TOMCAT_HOME/conf) deberemos añadir dentro de **<GlobalNamingResources>** las siguientes lineas:

<pre>&lt;GlobalNamingResources&gt;
…..
&lt;Resource name="jdbc/anjelica" auth="Container"
type="javax.sql.DataSource" driverClassName="org.postgresql.Driver"
url="jdbc:postgresql://localhost:5432/MIBASEDATOS"
username="USUARIO" password="CONTRASEÑA" maxTotal="20" maxIdle="10" maxWaitMillis="-1"/&gt;
…..
&lt;/GlobalNamingResources&gt;</pre>

Esto se utilizara para configurar nuestra fuente JNDI que permitirá conectarnos con la base de datos. Para probar esta aplicación cambiar los valores de _url_ para que apunten a vuestra base de datos, asi como el _username_ y la _password._

También sera necesario añadir las siguientes lineas al fichero _context.xml dentro de_ _**<context>**_ __de tomcat, que también estará en $TOMCAT_HOME/conf

<pre>&lt;Context&gt;
….
&lt;ResourceLink name="jdbc/anjelica"
global="jdbc/anjelica"
type="javax.sql.DataSource"/&gt; 
….
&lt;/Context&gt;</pre>

Con estos dos ficheros ya tendremos nuestro tomcat configurado para que use su propio pool de conexiones al que hemos llamado “jdbc/anjelica”.

Ahora debemos añadir a Tomcat la libreríapara conectarnos a postgresql, en este caso usamos la versión 42.2.2

&#8211; postgresql-42.2.2.jar

# 4. Configuración de la aplicación. {.western}

&nbsp;

En esta aplicación usaremos tanto ficheros xml como configuración en Java.

Lo primero es configurar nuestro ficheros xml para que Tomcat use Spring, para ello en el directorio **src\main\webapp\WEB-INF** de nuestra aplicación tenderemos estos ficheros.

  * applicationContext.xml
  * dispatcher-servlet.xml
  * web.xml

## 4.1 web.xml {.western}

El único fichero que usa Tomcat es web.xml, y este, a su vez, usa los dos anteriores, de tal manera que básicamente en web.xml , lo primero que hacemos es especificar que use el servlet de Spring y le decimos donde tendrá la configuración para ese servlet. Esto se hace con estas lineas:

<pre>&lt;servlet&gt;
        &lt;servlet-name&gt;dispatcher&lt;/servlet-name&gt;
        &lt;servlet-class&gt;
            org.springframework.web.servlet.DispatcherServlet
        &lt;/servlet-class&gt;
        &lt;init-param&gt;
            &lt;param-name&gt;contextConfigLocation&lt;/param-name&gt;
            &lt;param-value&gt;/WEB-INF/dispatcher-servlet.xml&lt;/param-value&gt;
        &lt;/init-param&gt;
        &lt;load-on-startup&gt;1&lt;/load-on-startup&gt;
&lt;/servlet&gt;</pre>

A continuación configuramos el contexto con las siguientes lineas:

<pre>&lt;context-param&gt;
        &lt;param-name&gt;contextConfigLocation&lt;/param-name&gt;
        &lt;param-value&gt;/WEB-INF/applicationContext.xml&lt;/param-value&gt;
&lt;/context-param&gt;
&lt;listener&gt;
	&lt;listener-class&gt;org.springframework.web.context.ContextLoaderListener&lt;/listener-class&gt;
&lt;/listener&gt;</pre>

## 4.2dispatcher-servlet.xml {.western}

Para configurar la parte Web de nuestra aplicación (el servlet realmente) pondremos las siguientes lineas en el fichero dispatcher-servlet.xml

<pre>&lt;annotation-driven /&gt;
  &lt;context:component-scan base-package="chu.jdbc" /&gt;
  
  &lt;beans:bean
    class="org.springframework.web.servlet.view.InternalResourceViewResolver"&gt;
    &lt;beans:property name="prefix" value="/WEB-INF/views/" /&gt;
    &lt;beans:property name="suffix" value=".jsp" /&gt;
  &lt;/beans:bean&gt;</pre>

Con **<annotation-driven />** permitiremos que en nuestra aplicación haya anotaciones @Controller y _@RequestMapping_ 

_Con_ _**<context:component-scan base-package=&#8221;chu.jdbc&#8221; />**_ _especificaremos que paquete deberá escanear Spring para buscar anotaciones en los ficheros java. En este caso especificamos que busque en el paquete “chu.jdbc” y sus hijos._

_Las ultimas lineas indican que usaremos JSP y especifica donde tendremos nuestros ficheros jsp._

Tenéis un excelente documento explicando como hacer esta misma configuración usando anotaciones java en la siguiente página: <https://www.baeldung.com/bootstraping-a-web-application-with-spring-and-java-based-configuration>

## 4.3applicationContext.xml {.western}

En este fichero crearemos nuestro DataSource que utilizaremos para conectarnos con el pool de conexiones anteriormente configurado en Tomcat.

Esto se hace añadiendo la siguiente linea:

<pre>&lt;jee:jndi-lookup id="dataSource" jndi-name="jdbc/anjelica" resource-ref="true"/&gt;</pre>

Tenéis más documentación de lo que hace esta linea en <http://www.jtech.ua.es/j2ee/publico/spring-2012-13/sesion01-apuntes.html>

De todos modos básicamente lo que hace es injectar en nuestra aplicación una clase DataSource que luego podremos usar con sentencias como esta:

<pre>@Autowired
 <span style="font-size: small;">DataSource ds;</span></pre>

# 5. La aplicación {.western}

## 5.1 Configuracion JPA y JDBC {.western}

En la clase JpaConfig es donde se hace toda la configuración que necesitamos para conectarnos a la base de datos.

Para ello, lo primero especificamos las siguientes configuraciones java

<pre>@Configuration
@EnableLoadTimeWeaving
@EnableJpaRepositories("chu.jdbc")</pre>

Explico las directivas, una a una.

  * @Configuration Especifica que es una clase de configuración, con ello haremos que Spring la cargue y ejecute.

  * @EnableLoadTimeWeaving Con esto permitimos a tomcat usar AOP. Si no la ponemos simplemente Tomcat no podrá hacer uso de Programación Orienta a a Aspectos (AOP) y la aplicación no funcionara. Mas documentación en: <https://docs.spring.io/spring/docs/5.0.8.RELEASE/spring-framework-reference/core.html#aop-aj-ltw-environment-tomcat>

  * @EnableJpaRepositories(&#8220;chu.jdbc&#8221;) especificamos que busque repositorios (clases marcadas con la directiva @Repository) en el paquete _chu.jdbc_

Ahora inyectamos el Datasource que ya tendremos disponible por la configuración aplicada anteriormente en nuestro fichero _applicationContext.xml_

<pre>@Autowired
DataSource ds;</pre>

Lo siguiente es crear nuestro fabrica de controladores de entidades (bueno, en ingles, nuestro EntityManagerFactoryBean), para ello y como usamos una versión de Hibernate superior a la 4 (con versiones anteriores utilizaríamos otra clase), ponemos el siguiente código:

<pre>@Bean
   public LocalContainerEntityManagerFactoryBean entityManagerFactory() {
      LocalContainerEntityManagerFactoryBean em  = new LocalContainerEntityManagerFactoryBean();
      em.setDataSource(ds);
      em.setPackagesToScan(new String[] { "chu.jdbc" });
 
      JpaVendorAdapter vendorAdapter = new HibernateJpaVendorAdapter();
      em.setJpaVendorAdapter(vendorAdapter);   
 
      return em;
   }</pre>

Como se puede ver simplemente creamos nuestro _LocalContainerEntityManagerFactoryBean_ , le añadimos nuestro DataSource y el adaptador de Hibernate. También le decimos los paquetes que deberá escanear para buscar “Entity Classes” (clases que sean entidades, vamos).

Por ultimo creamos nuestro controlador de transaciones con el siguiente código:

<pre>@Bean
   public PlatformTransactionManager transactionManager( EntityManagerFactory emf){
       JpaTransactionManager transactionManager = new JpaTransactionManager();
       transactionManager.setEntityManagerFactory(emf); 
       return transactionManager;
   }</pre>

Ahora ya solo falta crear nuestro ficherito hibernate.cfg.xml en nuestro directorio de _resources,_ pero en el solamente tenemos lo siguiente:

<pre>&lt;?xml version="1.0" encoding="UTF-8"?&gt;
&lt;!DOCTYPE hibernate-configuration PUBLIC
		"-//Hibernate/Hibernate Configuration DTD 3.0//EN"
		"http://hibernate.org/dtd/hibernate-configuration-3.0.dtd"&gt;
&lt;hibernate-configuration&gt;
    &lt;session-factory&gt;
        &lt;property name="hibernate.dialect"&gt;org.hibernate.dialect.PostgreSQLDialect&lt;/property&gt;      
    &lt;/session-factory&gt;
&lt;/hibernate-configuration&gt;</pre>

Es decir, simplemente definimos que vamos a usar una base de datos Postgresql. Este fichero **debe** existir. Cosas de Hibernate que hay que complacer.

Y _voila_ nuestro entorno para JPA + Hibernate ya esta configurado y listo para funcionar.

En la siguiente parte explicare como crear un respositorio y la magia que hay detras.

¡¡ Hasta la proxima, chabales!!