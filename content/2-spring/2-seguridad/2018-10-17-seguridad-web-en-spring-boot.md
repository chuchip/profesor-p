---
title: Seguridad WEB en Spring Boot
pre: "<b>o </b>"
author: El Profe
type: post
date: 2018-10-17T10:19:34+00:00
url: /2018/10/17/seguridad-web-en-spring-boot/
wpglobus_language:
  - es
categories:
  - java
  - mvc
  - seguridad
  - spring
  - spring boot
  - thymeleaf
tags:
  - java
  - mvc
  - seguridad
  - spring
  - spring boot
  - thymeleaf

---
Hola de nuevo, estudiantes 😉 . En esta entrada voy a explicar  como gestiona Spring la seguridad. No todo, por supuesto, que el tema de la seguridad daría para un libro muy gordote, pero al menos aprender a securizar una pagina web. En una próxima entrada hablare de como securizar un servicio REST.

Como siempre, comienzo diciendo que el código fuente de lo que explico lo tenéis en mi pagina de GITHUB, en <a href="https://github.com/chuchip/OAuthServer" target="_blank" rel="noopener noreferrer">https://github.com/chuchip/OAuthServer</a>. El programa esta realizado en Java, usando Spring Boot.

Bien, empecemos por como securizar una pagina web en Spring.

Realmente usando Spring Boot, es muy sencillo, pues haremos uso de lo que Spring denomina **starters**, que no son sino grupos de paquetes los cuales agrupan ciertas  funcionalidades. Así en este caso, incluiremos el paquete **Web**, **Thymeleaf** y, por supuesto, **Security.**

Aquí tenéis un pantallazo de Eclipse seleccionando los paquetes necesarios.

![captura-10](/img/2018/10/Captura-10.png")

De todos modos,  ya sabéis que en el fichero pom.xml podéis ver las dependencias más detalladamente.

<a href="https://www.thymeleaf.org/" target="_blank" rel="noopener noreferrer">Thymeleaf</a>, por si alguien no lo sabe, es un software que se integra perfectamente con Spring y que permite realizar plantillas de páginas WEB.  Como JSP, pero muy mejorado o si lo preferís un JavaServer Faces si conocéis más el mundo JavaEE. El caso es que permite realizar paginas HTML que se integran perfectamente con nuestras clases desarrolladas con Spring.

Como queremos poder ver en nuestra página web el nombre del usuario con el que nos hemos registrado, debemos usar la librería de seguridad de Thymeleaf para Spring. Para ello incluiremos las siguientes lineas en nuestro fichero pom.xml de Maven.

<pre>&lt;groupId&gt;org.thymeleaf.extras&lt;/groupId&gt;
  	&lt;artifactId&gt;thymeleaf-extras-springsecurity4&lt;/artifactId&gt;
    	&lt;version&gt;3.0.3.RELEASE&lt;/version&gt;
&lt;/dependency&gt;</pre>

Y aquí podéis ver como quedara la estructura de nuestro programa

![](/img/2018/10/Captura-11.png)

Ahora empecemos a declarar nuestra primera clase, a la que he llamado **WebSecurityConfiguration.java**

```
@SpringBootApplication
@EnableWebSecurity
public class WebSecurityConfiguration extends WebSecurityConfigurerAdapter {

	public static void main(String[] args) {
		SpringApplication.run(WebSecurityConfiguration.class, args);
	}
	
	@Bean
	@Override
	public AuthenticationManager authenticationManagerBean() throws Exception {
	       return super.authenticationManagerBean();
	}
	
    @Bean
    @Override
    public <a href="#UserDetailsService">UserDetailsService</a> userDetailsService() {
    	
    	UserDetails user=User.builder().username("user")
                        .password(passwordEncoder().encode("secret"))
    			.roles("USER").build();
    	UserDetails userAdmin = User.builder().username("admin")
                    .password(passwordEncoder().encode("secret"))
    		    .roles("ADMIN").build();
        return new InMemoryUserDetailsManager(user,userAdmin);
    }
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
  
   @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
        	.csrf().disable()
                .authorizeRequests()
            	.antMatchers("/","/index","/webpublico").permitAll()
            	.antMatchers("/webprivado").authenticated()
            	.antMatchers("/webadmin").hasRole("ADMIN").and()
                .formLogin()
                .loginPage("/login")
                .permitAll()
                .and()
                .logout() // Metodo get pues he desabilitado CSRF
                .permitAll();
    }
}
```

Lo primero es poner las etiquetas **@SpringBootApplication** y **@EnableWebSecurity**. La primera etiqueta es obvia, ya que nuestra aplicación queremos que funcione con Spring Boot ;-). Vamos, que o la ponéis o no seria una aplicación Spring Boot y ya nos podemos ir a casa :-D. La segunda es para especificar que queremos que se active la seguridad Web, realmente esta etiqueta no es obligatorio ponerla porque Spring Boot que es muy listo, en cuanto ve que tenemos el paquete **security **(en el pom.xml, recordad) en nuestro proyecto la incluye, pero no es mala cosa ponerla por claridad, aunque sea redundante.

Ahora especificamos que nuestra clase va a heredar de **WebSecurityConfigurerAdapter** pues vamos a sobrescribir algunas de las funciones de esa clase. Para que lo entendáis, básicamente _Spring_ mira a ver si hay alguna clase que implemente el interface **WebSecurityConfigurer,** el cual implementa la clase **WebSecurityConfigurerAdapter** , y si lo hay pues utiliza las funciones que tiene ese interface para configurar la seguridad de la aplicación.

Si no tuviéramos una clase que implementara ese interface, _Spring_ simplemente no dejaría acceder a ninguna pagina de nuestra aplicación, lo que, como comprenderéis no es muy práctico 😉

Bien, ahora sobrescribimos la función **authenticationManagerBean** que devolverá la clase encargada de manejar las autentificaciones (como su propio nombre indica 😉 ). Vale, os estaréis preguntando, ¿ pero si solo llama a la función padre para que la definimos?. Muy simple porque le ponemos la etiqueta **@Bean**, para que _Spring_ sepa de donde sacar (inyectar) un objeto tipo **AuthenticationManager **pues lo necesita para controlar la seguridad.

En la <a id="UserDetailsService"></a>función **userDetailsService** definimos los usuarios que van a tener acceso a nuestra web. En este caso creamos dos usuarios: **user** y **admin** (sí lo se, no es que me haya currado mucho el tema de los nombres 😉 ). Cada uno de ellos con su contraseña y su ROL. Aclarar que el ROL es un literal libre, es decir que ahí podemos poner lo que queramos, por ejemplo USUARIO\_CON\_PECAS. El caso es que luego ese ROL lo utilizaremos y debe coincidir letra a letra con el establecido.

Observar también que la contraseña se la damos encriptada, en este caso con el algoritmo **BCrypt**. Esto lo hacemos llamando a la función **passwordEncoder**, la cual esta anotada con la etiqueta **@Bean** para que _Spring_ la use.

Es decir, _Spring_ necesita saber que sistema de encriptación estamos usando para guardar nuestras contraseñas, y para ello busca un objeto que implemente el interface **PasswordEncoder**. Si no lo encuentra nos fallara la aplicación.

Aclarar que estamos usando la forma más sencilla de declarar los usuarios, guardándolos en memoria con la clase **InMemoryUserDetailsManager. **En un programa de verdad, se usaría **JdbcUserDetailsManager **que nos permitiría guardarlos en una base de datos o cualquier otra clase que implemente el interface **UserDetailsManager **como podría ser  **LdapUserDetailsManager **si quisiéramos usar un servicio LDAP.

<a id="HttpSecurity"></a>Y ya solo nos falta definir que partes de nuestra aplicación vamos a proteger y que roles deben de tener permisos para acceder a cada parte de ella. Sí, he escrito _roles_ y no usuarios porque como hemos dicho antes, al definir un usuario, lo debemos asignar a un rol  (o grupo que es más español, si lo preferís). Y, normalmente, las reglas de filtrado se aplican por el grupo al que pertenece el usuario. Para definir los permisos de cada recurso lo haremos configurando el objeto <a href="https://docs.spring.io/autorepo/docs/spring-security/3.2.3.RELEASE/apidocs/org/springframework/security/config/annotation/web/builders/HttpSecurity.html" target="_blank" rel="noopener noreferrer"><strong>HttpSecurity</strong> </a>recibido en la función _**protected void configure(HttpSecurity http)**_

Voy explicando linea a linea lo que se hace en esta función:

  * csrf().disable()

Deshabilita el control de csrf. CRSF son las siglas de **Cross-site request forgery ** como explica la Wikipedia

<div style="background-color: #ccffff;">
  <em>CRSF del inglés Cross-site request forgery o falsificación de petición en sitios cruzados) es un tipo de exploit malicioso de un sitio web en el que comandos no autorizados son transmitidos por un usuario en el cual el sitio web confía. Esta vulnerabilidad es conocida también por otros nombres como XSRF, enlace hostil, ataque de un click, cabalgamiento de sesión, y ataque automático.</em>
</div>

El deshabilitar el CRSF tiene como efecto secundario que se pueda realizar un logout de una sesión con una petición HTTP tipo **GET**, pues por defecto solo se puede hacer con una petición POST.

  * .authorizeRequests()
  
    .antMatchers(&#8220;/&#8221;,&#8221;/index&#8221;,&#8221;/webpublico&#8221;).permitAll()

Especificamos que las peticiones que en la ruta este cualquiera de las cadenas **&#8220;/&#8221;,&#8221;/index&#8221;,&#8221;/webpublico&#8221; **no tendrán seguridad. Es decir estarán permitidas para todo el mundo.

  * antMatchers(&#8220;/webprivado&#8221;).authenticated()

Especificamos que las peticiones a la ruta **&#8220;/webprivado&#8221; **solo podrán ser procesadas si el usuario esta autentificado, sin especificar a que ROL debe pertenecer.

  * .antMatchers(&#8220;/webadmin&#8221;).hasRole(&#8220;ADMIN&#8221;)

Solo los usuarios que sean del grupo **ADMIN **tendrán acceso a la URL &#8220;/webadmin&#8221;

La función **antMatchers **permite el uso de expresiones regulares, por lo que si, por ejemplo, quisiéramos aplicar una regla a todo lo que dependa de una ruta, podríamos poner esto :

**http.antMatchers(&#8220;/users/**&#8221;).hasRole(&#8220;USER&#8221;) **para especificar que cualquier petición a la URL **/users/Y\_LO\_QUE_SEA** solo tendrán acceso los usuarios  que pertenezcan al grupo **USER.**

  * .formLogin().loginPage(&#8220;/login&#8221;).permitAll()

Especificamos que la pagina de login sera &#8220;/login&#8221; (valga la _repugnancia_ ;-)) y que  que puede acceder  todo el mundo.

  * logout().permitAll()

Especificamos que a la pagina de desconexión (logout) pude acceder a todo el mundo. Por defecto esta pagina responde en la URL &#8220;/logout&#8221;

Perfecto, ya tenemos definida la seguridad de nuestras paginas web, ahora solo queda definir los puntos de entrada a las páginas. Eso se hace en la clase **WebController.java**

```
@Controller
public class WebController {
	
	 @RequestMapping({"/","index"})
	  public String inicio() {
	    return "index";
	  }

	 @RequestMapping("/webprivado")
	  public String privado() {
	    return "privado";
	  }
	 @RequestMapping("/webpublico")
	  public String loginpub() {
	    return "publico";
	  }
	 @RequestMapping("/webadmin")
	  public String admin() {
	    return "admin";
	  }
	 @RequestMapping("/login")
	  public String login() {
	    return "login";
	  }
}
```

La clase como se ve no tiene muchos misterios,simplemente especificamos con la etiqueta** @Controller **que sera una clase donde vamos a definir puntos de entrada para las peticiones web.

En las diferentes funciones tenemos la etiqueta **@RequestMapping**  para especificar la URL que debe procesar cada función. Así la función **inicio** sera llamada cuando haya una petición a la URL &#8220;/&#8221; o a &#8220;/index&#8221;. Observar que no hay que poner la barra inicial en &#8220;index&#8221;.

La cadena devuelta sera la plantilla **Thymeleaf** devuelta, de tal manera que la llamada  función inicio  devolverá la plantilla &#8220;index.html&#8221; que es la siguiente:

<pre>&lt;!DOCTYPE html&gt;
&lt;html xmlns="http://www.w3.org/1999/xhtml" xmlns:th="http://www.thymeleaf.org"
      xmlns:sec="http://www.thymeleaf.org/thymeleaf-extras-springsecurity5"&gt;
    &lt;head&gt;
        &lt;title&gt;Página Inicio&lt;/title&gt;
    &lt;/head&gt;
    &lt;body&gt;
   		&lt;h1&gt;Pagina Inicio&lt;/h1&gt;
   		&lt;p&gt;Pulsa  &lt;a th:href="@{/webpublico}"&gt;aqui&lt;/a&gt; para ver una pagina publica.&lt;/p&gt;
   		&lt;p&gt;Si eres un usuario normal pulsa &lt;a th:href="@{/webprivado}"&gt;aqui&lt;/a&gt;  para ver una pagina privada&lt;/p&gt;
   		&lt;p&gt;Si eres un administrador normal pulsa &lt;a th:href="@{/webadmin}"&gt;aqui&lt;/a&gt;  para ver la pagina del administrador&lt;/p&gt;   		
   		&lt;div sec:authorize="isAuthenticated()"&gt;
			Hello &lt;span sec:authentication="name"&gt;someone&lt;/span&gt;
			&lt;p&gt;&lt;a th:href="@{/logout}"&gt;Desconectar&lt;/a&gt;&lt;/p&gt;
		&lt;/div&gt;
   		
    &lt;/body&gt;
&lt;/html&gt;</pre>

¿A que casi parece HTML puro?. Es una de las ventajas de **Thymeleaf  **que usa  etiquetas HTML estándar. No voy a explicar este lenguaje, pero os explicare un poco las etiquetas usadas:

`<a th:href=”@{/webpublico}”>`

 Crea un enlace a la URL "/webpublico";. Seria como poner la etiqueta &#8220;

`<div sec:authorize=”isAuthenticated()”>`

Solo se renderizara el código en el DIV si el usuario esta autentificado. En otras palabras si el usuario no esta logueado no se mostrara en la página web lo que hay entre las etiquetas  DIV (de hecho no se mostrara ni el DIV).

`<span sec:authentication=”name”>someone</span>`
Si el usuario esta autentificado mostrara el nombre del usuario, en caso contrario mostrara lo que haya entre las etiquetas **span. **En este caso mostraría **someone**.

Y con esto ya tenemos una aplicación securizada. !Sí!, con solo dos clases java y sus correspondientes ficheros HTML.

Para terminar esta entrada, os dejo unos capturas de pantalla de la aplicación:

![](/img/2018/10/Captura-12.png")

![](/img/2018/10/Captura1-1.png)

![](/img/2018/10/Captura5.png)

![](/img/2018/10/Captura2.png)

![](/img/2018/10/Captura4.png)

**¡¡ Hasta otra !!**
