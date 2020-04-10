---
title: Spring WebFlow con JSP – Configuración
pre: "<b>o </b>"
weight: 10
author: El Profe
type: post
date: 2018-10-29T08:49:12+00:00
url: /2018/10/29/spring-webflow-con-jsp-configuracion/
categories:
  - bootstrap
  - java
  - jdbc
  - jquery
  - jsp
  - jstl
  - mvc
  - security
  - seguridad
  - spring
  - tomcat
  - webflow
tags:
  - bootstrap
  - java
  - jdbc
  - jquery
  - jstl
  - l
  - mvc
  - security
  - seguridad
  - spring
  - tomcat
  - webflow

---
Voy a intentar explicar como funciona <a href="https://projects.spring.io/spring-webflow/" target="_blank" rel="noopener">Spring WebFlow</a> y para ello, como siempre, lo haré desarrollando un programa que podéis descargar de  <a href="https://github.com/chuchip/webflowExample" target="_blank" rel="noopener">https://github.com/chuchip/webflowExample</a>

El programa simulara que entras a la página de un banco donde puedes realizar una transferencia de tus cuentas personales a otra. Para ello, primero deberás identificarte y según el usuario con el que te identifiques tendrás acceso a unas cuentas que a su vez disponen de un saldo establecido. Para realizar todo esto utilizo H2 como base de datos y la autentificación se realiza con el <a href="https://spring.io/projects/spring-security" target="_blank" rel="noopener">paquete de seguridad de Spring</a>, utilizando JDBC . Por hacer la página mas funcional utilizo Bootstrap y JQuery.

El programa esta realizado para que funcione bajo Tomcat, en el contexto: webflow. Por lo que deberemos ir a la dirección http://localhost:8080/webflow  para probar nuestra aplicación:

![](/img/2018/10/Captura-23-1024x723.png")

En esta imagen se puede ver la definición del flujo de trabajo.

![Flujo de trabajo](/img/2018/10/diagrama_flujo.gif)



<a href="https://projects.spring.io/spring-webflow/" target="_blank" rel="noopener">Spring WebFlow</a> es un paquete con el cual podemos definir el flujo de nuestra aplicación. Es decir, definimos las acciones a realizar cuando se pulse un enlace, se cumpla cierta condición, etc. Estos flujos son definidos en ficheros XML, de tal manera que ahí es donde definimos que de la PAGINA\_X, al pulsar el BOTON1, vaya a la PAGINA\_XY, siempre y cuando  la CONDICION_Z se cumpla. Esto nos permite separar la lógica del programa de las vistas (los ficheros JSP), ademas de ser más fácil el reutilizar código.

<span style="text-decoration: underline; font-size: 14pt; font-family: georgia, palatino, serif;"><strong>&#8211; Estructura</strong></span>

La estructura del programa es la que se ve en la  imagen siguiente:

![](/img/2018/10/Captura-22.png)

Dentro del directorio **WEB-INF** esta la carpeta **flows,** que a su vez tiene la carpeta **traspaso** y **time** donde definimos los dos flujos de trabajo que se usan en la aplicación. Observar que los ficheros **xml** con la definición del flujo de trabajo también se aloja en las mismas carpetas donde estan los **JSP** de su flujo. Es decir el flujo de trabajo &#8220;**traspaso**&#8221; se define en el fichero &#8220;traspaso.xml&#8221; y utiliza los ficheros: cuentaOrigen.jsp, importe.jsp, confirmar.jsp, \_include.jsp y \_navegador.jsp.

Dentro del directorio **WEB-INF** también se alojan las carpetas **css** y **img** donde se guardan respectivamente las plantillas css y las imágenes de la aplicación.** **

  * **<span style="text-decoration: underline;"><span style="font-size: 14pt;">Configurando la Base de Datos y Persistencia</span></span>**

Empezare mostrando las tablas usadas en la aplicación:

<pre>create TABLE clientes
(
    id int NOT NULL,
    login  varchar(45) not null,
    nombre varchar(100) not null,
    dni varchar(15) not null,
    unique key(login)
);

create table cuentas(
    cuenta varchar(50) not null,
    importe double not null,
    unique key(cuenta)
);

create table cuentas_clientes(
    id int not null,
    cuenta varchar(50) not null,
    unique key(id,cuenta)
);
alter table cuentas_clientes add CONSTRAINT fk_cuentas FOREIGN KEY (cuenta) REFERENCES cuentas (cuenta);


create table users(
    username   varchar(45) not null,
    password varchar(100) not null,
    enabled smallint not null default 1,
    unique key (username)
);

CREATE TABLE user_roles (
  user_role_id int(11) NOT NULL AUTO_INCREMENT,
  username varchar(45) NOT NULL,
  role varchar(45) NOT NULL,
  PRIMARY KEY (user_role_id),
  UNIQUE KEY uni_username_role (role,username)
  );
alter table user_roles add CONSTRAINT fk_username FOREIGN KEY (username) REFERENCES users (username)</pre>

La tabla **clientes, cuentas y cuentas_clientes** son usadas por la lógica del programa, mientras que **users** y **user_roles** son usadas por el modulo de seguridad de spring.

La persistencia esta definida en la clase **JpaConfig.** Podéis ver una explicación de esta clase en la parte de la configuración de JPA de esta entrada: [http://www.profesor-p.com/2018/09/03/aplicacion-en-spring-rest-y-angular-2-parte/#jpa][2]

  * **Dependencias**

Necesitaremos tener el paquete de JPA, JDBC,  Hibernate, soporte de transaciones y por supuesto de H2.

<pre>&lt;dependency&gt;
            &lt;groupId&gt;org.springframework.data&lt;/groupId&gt;
            &lt;artifactId&gt;spring-data-jpa&lt;/artifactId&gt;
            &lt;version&gt;2.0.9.RELEASE&lt;/version&gt;
        &lt;/dependency&gt;
        &lt;dependency&gt;
            &lt;groupId&gt;org.springframework&lt;/groupId&gt;
            &lt;artifactId&gt;spring-jdbc&lt;/artifactId&gt;
            &lt;version&gt;5.0.10.RELEASE&lt;/version&gt;
        &lt;/dependency&gt;
        &lt;dependency&gt;
            &lt;groupId&gt;com.h2database&lt;/groupId&gt;
            &lt;artifactId&gt;h2&lt;/artifactId&gt;
            &lt;version&gt;1.4.197&lt;/version&gt;        
            &lt;scope&gt;runtime&lt;/scope&gt;
        &lt;/dependency&gt;
        &lt;dependency&gt; 
            &lt;groupId&gt;org.hibernate&lt;/groupId&gt; 
            &lt;artifactId&gt;hibernate-agroal&lt;/artifactId&gt; 
            &lt;version&gt;5.3.5.Final&lt;/version&gt; 
            &lt;type&gt;pom&lt;/type&gt; 
        &lt;/dependency&gt;
       &lt;dependency&gt; &lt;!-- Paquete para dar soporte a transaciones --&gt;
            &lt;groupId&gt;org.springframework&lt;/groupId&gt;
            &lt;artifactId&gt;spring-tx&lt;/artifactId&gt;
            &lt;version&gt;5.1.1.RELEASE&lt;/version&gt;
        &lt;/dependency&gt;</pre>

  * <span style="text-decoration: underline; font-size: 14pt;"><strong>Configurando MVC y  WEB-FLOW</strong></span>

<ul style="list-style-type: square;">
  <li>
    <em><span style="text-decoration: underline;">Clase WebConfig</span> </em><span style="background-color: #ccffff;"><strong><br /> </strong></span>
  </li>
</ul>

En ella se configura  la parte **MVC**  de la aplicación. Así en la función **addResourceHandlers** especifico que los recursos de las peticiones  a  &#8220;webjars&#8221; deberán ser buscadas en el directorio **/webjars** y que los recursos de las peticiones   a **resources** deberán ser buscadas en el directorio /**WEB-INF/resources/.** Es decir que cuando vayamos a http://localhost:8080/webflow/resources/XX deberá traer el fichero  /WEB-INF/resources/XX

<pre>@Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry
          .addResourceHandler("/webjars/**")
          .addResourceLocations("/webjars/");

        registry
          .addResourceHandler("/resources/**")
          .addResourceLocations("/WEB-INF/resources/");

    }</pre>

La función **viewResolver** establece que se usara la JSTL (que no es sino una extensión de JSP), especificando en que directorio y que extensiones debe resolver.

<pre>@Bean
    public ViewResolver viewResolver() {
        InternalResourceViewResolver viewResolver = new InternalResourceViewResolver();
        viewResolver.setViewClass(JstlView.class);
        viewResolver.setPrefix("/WEB-INF/jsp/");
        viewResolver.setSuffix(".jsp");
 
        return viewResolver;
    }</pre>

Las funciones **FlowHandlerMapping** y **FlowHandlerAdapter** especificaran los identificadores o registros de webflow y sus correspondiente ejecutores. Ahora explicare que es eso 😉

<ul style="list-style-type: square;">
  <li>
    <em>Clase WebFlowConfig</em>
  </li>
</ul>

En esta clase configuramos webflow como tal, para ello lo primero es extender de la clase **AbstractFlowConfiguration** y definir las funciones que detallo a continuación:

<pre>@Bean
    public FlowDefinitionRegistry flowRegistry() {
        return getFlowDefinitionRegistryBuilder()
                .addFlowLocation("/WEB-INF/flows/traspaso/traspaso.xml","traspaso")
                .addFlowLocation("/WEB-INF/flows/traspaso/time/traspaso_time.xml","traspaso_time")
                .build();
    }</pre>

En esta función se declaran los diferentes identificadores de los flujos a usar.Así tenemos que el identificador &#8220;traspaso&#8221; estará definido en el directorio _&#8220;`/WEB-INF/flows/traspaso/traspaso.xml`&#8220;._ Esto lo que significa es que cuando vayamos a la dirección: **http://localhost:8080/webflow/traspaso** entraremos en el flujo que tenemos definido en el fichero indicado. A su vez, cuando vayamos a **http://localhost:8080/webflow/traspaso_time** entraremos al flujo declarado en &#8220;`/WEB-INF/flows/traspaso/time/traspaso_time.xml`&#8220;.

En este caso yo defino individualmente los dos flujos, pero se pueden utilizar comodines para que Spring busque dentro de un directorio todos los ficheros que cumplan ciertos criterios. Así, la sentencia: `.addFlowLocationPattern(<span class="hl-string">"/WEB-INF/flows/**/*-flow.xml"</span>)`especificaría que añadiera todos los flujos que encontrara en el directorio /WEB-INF/flows  cuyo nombre terminara en &#8220;flow.xml&#8221;. Los identificadores en este caso serian los nombres de los ficheros sin la extensión xml. Es decir si tenemos el fichero: `<span class="hl-string">/WEB-INF/flows/consulta-flow.xml</span>` el identificador seria &#8220;consulta-flow&#8221;

La siguiente función simplemente llama a la anterior para especificar los ejecutores de los registros  anteriormente definidos.

<pre>@Bean
    public FlowExecutor flowExecutor() {
        return getFlowExecutorBuilder(flowRegistry()).build();
    }</pre>

Aclarar que el trabajo del F**<span style="text-decoration: underline;">lowExecutor</span>** es crear y ejecutar los flujos de trabajo que anteriormente hayamos definido en los registros.

Las siguientes funciones unen las vistas (ficheros JSP) con el paquete webflow. Como se puede observar crea dos objetos en los cuales se definen las uniones entre el  paquete MVC de Spring y el paquete WebFlow. Esto es así porque el paquete WebFlow también funciona con otros MVC como  <a href="https://es.wikipedia.org/wiki/JavaServer_Faces" target="_blank" rel="noopener">JavaServer Faces de JavaEE</a>

<pre> @Bean
    public FlowBuilderServices flowBuilderServices() {
        return getFlowBuilderServicesBuilder()
          .setViewFactoryCreator(mvcViewFactoryCreator())
          .setDevelopmentMode(true).build();
    }
 
    @Bean
    public MvcViewFactoryCreator mvcViewFactoryCreator() {
        MvcViewFactoryCreator factoryCreator = new MvcViewFactoryCreator();
        factoryCreator.setViewResolvers(
          Collections.singletonList(this.webMvcConfig.viewResolver()));
        factoryCreator.setUseSpringBeanBinding(true);
        return factoryCreator;
    }</pre>

  * **Dependencias**

Ademas del paquete **JSTL**, también usamos el paquete **spring-security-taglibs** que es una extensión de JSTL que nos permite trabajar  con el paquete de seguridad de Spring dentro de nuestros ficheros JSP. Por supuesto también debemos incluir el paquete **spring-webflow**

<pre>&lt;dependency&gt;
            &lt;groupId&gt;jstl&lt;/groupId&gt;
            &lt;artifactId&gt;jstl&lt;/artifactId&gt;
            &lt;version&gt;1.2&lt;/version&gt;
        &lt;/dependency&gt;        
        &lt;dependency&gt;
            &lt;groupId&gt;org.springframework.security&lt;/groupId&gt;
            &lt;artifactId&gt;spring-security-taglibs&lt;/artifactId&gt;
            &lt;version&gt;5.1.1.RELEASE&lt;/version&gt;
        &lt;/dependency&gt;
        &lt;dependency&gt;
            &lt;groupId&gt;org.springframework.webflow&lt;/groupId&gt;
             &lt;artifactId&gt;spring-webflow&lt;/artifactId&gt;
             &lt;version&gt;2.5.1.RELEASE&lt;/version&gt;
        &lt;/dependency&gt;</pre>

&#8211; <span style="text-decoration: underline;"><span style="font-size: 14pt;"><strong>Configurando la seguridad</strong></span></span>

No es mi intención el volver a explicar como funciona la seguridad en Spring, pues aunque en anteriores ejemplos <a href="http://www.profesor-p.com/2018/10/17/seguridad-web-en-spring-boot/" target="_blank" rel="noopener">(http://www.profesor-p.com/2018/10/17/seguridad-web-en-spring-boot/)</a> los usuarios los guardaba en memoria en vez en una base de datos como en esta ocasión, realmente lo único que cambia es que se incluye la  función: `<strong>configure(AuthenticationManagerBuilder auth)</strong>`y se quita la función **UserDetailsService** de la clase **SecurityWebConfig**.

<pre>@Override
protected void configure(AuthenticationManagerBuilder auth) throws Exception
{
     auth
            .jdbcAuthentication()
            .dataSource(dataSource)
             .usersByUsernameQuery(
                 "select username,password, enabled from users where username=?")
            .authoritiesByUsernameQuery(
                "select username, role from user_roles where username=?")
                     .passwordEncoder(passwordEncoder());
}</pre>

En esta función se configura la autentificación por jdbc de la clase **AuthenticationManagerBuilder** especificándole el **DataSource** a usar, así como las sentencias SQL  a ejecutar para validar los usuarios y los roles de estos.

La clase **configure(HttpSecurity http)** configura  la seguridad no teniendo nada en particular, solo resaltar que no deshabilito el modulo de seguridad CRSF por lo cual todas las peticiones POST deben incluir nuestro identificador CRSF. Lo veremos más tarde al explicar los ficheros JSP.

<pre>@Override
    protected void configure(HttpSecurity http) throws Exception {
        http
            .authorizeRequests()
            .antMatchers("/", "/index", "/webjars/**", "/resources/**").permitAll()
            .antMatchers("/**").authenticated()
            .anyRequest().denyAll().and()
            .formLogin()
            .loginPage("/login")
            .defaultSuccessUrl("/user") // Para que registremos el usuario
            .permitAll()
            .and()
            .logout().logoutSuccessUrl("/index")
            .permitAll();
    }</pre>

La clase **SecurityWebApplicationInitializer** que extiende de **AbstractSecurityWebApplicationInitializer** debe estar definida pues esta aplicación al no ser Spring Boot, sino Spring a secas la necesita para implementar la seguridad web. La configuración, simplemente es inexistente. Con que exista la clase es suficiente 😉

  * **Dependencias**

Definimos los paquetes de seguridad de Spring, incluyendo el de seguridad web.

<pre>&lt;dependency&gt;
        	&lt;groupId&gt;org.springframework.security&lt;/groupId&gt;
    		&lt;artifactId&gt;spring-security-core&lt;/artifactId&gt;
                &lt;version&gt;5.1.1.RELEASE&lt;/version&gt;
	&lt;/dependency&gt;
        &lt;dependency&gt;
        	&lt;groupId&gt;org.springframework.security&lt;/groupId&gt;
    		&lt;artifactId&gt;spring-security-web&lt;/artifactId&gt;
                &lt;version&gt;5.1.1.RELEASE&lt;/version&gt;
	&lt;/dependency&gt;
        &lt;dependency&gt;
        	&lt;groupId&gt;org.springframework.security&lt;/groupId&gt;
    		&lt;artifactId&gt;spring-security-config&lt;/artifactId&gt;
                &lt;version&gt;5.1.1.RELEASE&lt;/version&gt;
	&lt;/dependency&gt;</pre>

En la próxima entrada explicare el programa paso a paso 😉

¡¡ Nos vemos!!

 [1]: http://www.profesor-p.com/img/2018/10/diagrama_flujo.gif
 [2]: http://www.profesor-p.com/2018/09/03/aplicacion-en-spring-rest-y-angular-2-parte/