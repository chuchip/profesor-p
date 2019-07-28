---
title: Securizando servicios REST con Oauth2 en SpringBoot
pre: "<b>o </b>"
author: El Profe
type: post
date: 2018-10-18T13:20:43+00:00
url: /2018/10/18/securizando-servicios-rest-con-oauth2-en-springboot/
wpglobus_language:
  - es
categories:
  - java
  - oauth2
  - rest
  - seguridad
  - spring boot
tags:
  - java
  - oauth2
  - rest
  - seguridad
  - spring boot

---
En esta entrada explicare como podemos dotar de seguridad a servicios REST en Spring Boot. La aplicación de ejemplo es la misma que la [entrada de seguridad WEB anterior][1], así que el código fuente lo tenéis en: <a href="https://github.com/chuchip/OAuthServer" target="_blank" rel="noopener">https://github.com/chuchip/OAuthServer</a>.

### Explicando la tecnologia Oauth2

Como he dicho, utilizaremos el protocolo OAuth2, así que lo primero sera explicar como funciona este protocolo.

OAuth2 tiene algunas variantes pero yo os voy a explicar la que utilizare en el programa y,  para ello, voy a poneros un ejemplo para que entendáis lo que pretendemos hacer.

Voy a poner una escena cotidiana: El pago con una tarjeta de crédito en un comercio.En este caso hay tres interlocutores: La tienda, el banco y nosotros. En el protocolo Oauth2 pasa algo parecido. Estos son los pasos:

  1. El cliente , o sea, el comprador, solicita al banco una tarjeta de crédito, para que el banco nos la de,  comprobara quienes somos, y nos otorga un crédito dependiendo de la pasta que tengamos en la cuenta o bien nos dice que no le hagamos perder el tiempo ;-). En el protocolo OAuth2 al que otorga las tarjetas se le llama Servidor de Autentificación.
  2. Si el banco nos ha dado la tarjeta, podremos ir a la tienda, es decir al servidor web, y le presentamos la tarjeta de crédito. La tienda no nos conoce de nada, pero puede preguntar al banco, a través del lector de tarjetas si puede confiar en nosotros y hasta que punto (el saldo de crédito). La tienda seria el Servidor de Recursos.
  3. La tienda  dependiendo del dinero que le diga el banco que tenemos nos permitirá comprar unos productos u otros. En la analogía OAuth2, el servidor web nos permitirá acceder a unas paginas u a otras dependiendo de si  somos muy ricos, ricos, medios  o pobres.

Como comentario, decir, por si no os habéis dado cuenta, que se utilizan servidores de autentificación habitualmente. Cuando vais a una pagina web y os pide registraros, pero como opción os deja hacerlo a través de Facebook o Google, estáis utilizando esta tecnología. Google o Facebook se convierte en el &#8216;banco&#8217; que emite esa &#8216;tarjeta&#8217;, la pagina web que os pide registraros, la usara para comprobar que tenéis &#8216;credito&#8217; y dejaros entrar. Espero que se entienda el ejemplo ;-).

![](/img/2018/10/Captura-17.png") 



Aquí podéis ver la pagina web de el periódico &#8220;El Pais&#8217;, creando una cuenta. Si utilizamos Google o Facebook, el periódico (la tienda) confiara en lo que les digan esos proveedores de autentificaciones. En este caso lo único que necesita la pagina web es que tengáis una tarjeta de crédito, sin importar el saldo 😉


### Creando un Servidor de Autorizaciones (AuthServer)

¿ Entendido ?. OK, pues vamos a ver como crear un banco, la tienda y toda la parafernalia 😉

Lo primero, en nuestro proyecto,  necesitamos tener las dependencias adecuadas, necesitaremos los inicializadores (starters en ingles) : **Cloud OAuth2, Security y Web**

![](/img/2018/10/Captura-13.png)

Bien, empecemos por definir el banco, esto lo hacemos en la clase: **AuthorizacionServerConfiguration**

```
@Configuration
@EnableAuthorizationServer
public class AuthorizacionServerConfiguration  extends AuthorizationServerConfigurerAdapter  {

	  @Autowired
	  @Qualifier("authenticationManagerBean")
	  private AuthenticationManager authenticationManager;
	  
	  @Autowired
	  private TokenStore tokenStore;
	
	@Override
	public void configure(ClientDetailsServiceConfigurer clients) throws Exception {
		clients.inMemory()
		    .withClient("cliente")
            .authorizedGrantTypes("password", "authorization_code", "refresh_token", "implicit")
            .authorities("ROLE_CLIENT", "ROLE_TRUSTED_CLIENT","USER")
            .scopes("read","write")
            .autoApprove(true)	        
            .secret(passwordEncoder().encode("password"));          
	}
	
	 @Bean
	    public PasswordEncoder passwordEncoder() {
	        return new BCryptPasswordEncoder();
	    }
	 @Override
	 public void configure(AuthorizationServerEndpointsConfigurer endpoints) throws Exception {
	     endpoints
	             .authenticationManager(authenticationManager)	        
	             .tokenStore(tokenStore);
	 }
	 
	  @Bean
	  public TokenStore tokenStore() {
	      return new InMemoryTokenStore();
	  }	
}
```

Empezamos la clase anotandola como de configuración con la etiqueta @**Configuration** y después usamos la etiqueta **@EnableAuthorizationServer** para decirle a Spring que active el servidor de autorizaciones. Para definir las propiedades del servidor especificamos que nuestra clase extiende de **AuthorizationServerConfigurerAdapter** , la cual implementa el interface **AuthorizationServerConfigurerAdapter**, por lo cual, Spring usara esta clase para parametrizar el servidor.

Definimos un bean tipo **AuthenticationManager ** que Spring provee automagicamente y que nosotros recogeremos con la etiqueta **@Autowired**.  También definimos un objeto  **TokenStore,**   pero para poder injectarlo debemos definirlo, lo cual hacemos en la función **public TokenStore tokenStore().**

El **AuthenticationManager **  o Controladores de Autentificaciones, como he dicho lo provee Spring pero tenderemos que configurarlo nosotros. Más tarde explicare como se hace. El **TokenStore** o Almacén de Identificadores es  donde se guardaran los identificadores que nuestro servidor de autentificaciones vaya suministrando, para que cuando el servidor de recursos (la tienda) le pregunte el crédito sobre una tarjeta de crédito este pueda responderle. En este caso usamos la clase **InMemoryTokenStore** que guardara los identificadores en memoria. En una aplicación real podríamos usar un **JdbcTokenStore** para guardarlos en una base de datos, para que si se cae la aplicación los clientes no tengan que renovar sus tarjetas de credito 😉

En la función **configure(ClientDetailsServiceConfigurer clients)** especificamos las credenciales del banco, digo del administrador de autentificaciones. ademas de los servicios que ofrece. Sí, hablando en plural, porque para poder acceder al banco debemos tener un usuario y contraseña para cada uno de los servicios que ofrece. Esto es  un concepto muy importante: _El usuario y contraseña es del banco no del cliente,_ para cada servicio que ofrezca el banco habrá una única autentificación, si bien podrá ser la misma para diferentes servicios.

Detallare las lineas:

  * **clients.inMemory()**  Especifica que vamos a guardar los servicios en memoria. En una aplicación &#8216;real&#8217; lo guardaríamos en una base de datos, un servidor LDAP, etc.
  * **withClient(&#8220;cliente&#8221;)** Es el usuario con el que nos identificaremos en el banco. En este caso se llamara &#8216;cliente&#8217;. ¿ Igual habria sido mejor llamarle &#8216;user&#8217; 😉 ?
  * a**uthorizedGrantTypes(&#8220;password&#8221;, &#8220;authorization\_code&#8221;, &#8220;refresh\_token&#8221;, &#8220;implicit&#8221;)**. Especificamos los servicios que estamos configurando para el usuario definido , para &#8216;**cliente**&#8216;. En nuestro ejemplo solo usaremos el servicio **password**.
  * **authorities(&#8220;ROLE\_CLIENT&#8221;, &#8220;ROLE\_TRUSTED_CLIENT&#8221;,&#8221;USER&#8221;).** Especifica roles o grupos que tiene el servicio ofrecido. Tampoco lo usaremos en nuestro ejemplo asi que dejemoslo correr de momento.
  * **scopes(&#8220;read&#8221;,&#8221;write&#8221;).** El ambito del servicio. Tampoco lo usaremos en nuestra aplicación.
  * **autoApprove(true)**. Si debe aprobar automaticamente las peticiones del cliente. Pondremos que si para hacer más sencilla la aplicación.
  * secret(passwordEncoder().encode(&#8220;password&#8221;)). Contraseña del cliente. Observar que se llama la función **encode** que tenemos definida un poco más abajo, para especificar con que tipo de encriptación se guardara la contraseña. La función **encode**, esta anotada con la etiqueta @Bean porque spring, cuando le suministremos la contraseña en una petición HTTP, buscara un objeto **PasswordEncoder** para comprobar la validez de la contraseña entregada.

Y por último tenemos la función **configure(AuthorizationServerEndpointsConfigurer endpoints)** donde definimos que controlador de autentificaciones y que almacén de identificadores deben usar los puntos de salida. Aclarar que los puntos de salida  son las URLs por donde &#8216;hablaremos con nuestro banco&#8217;, para solicitar las tarjetas de cerdito 😉

De acuerdo, ya tenemos nuestro servidor de autentificaciones creado pero aun nos falta la manera de que este sepa quienes somos y nos ponga en diferentes grupos, según las credenciales introducidas. Bien, para hacer esto usaremos la misma clase que en utilizamos para proteger una pagina web. Si habéis leído el articulo anterior: <a href="http://www.profesor-p.com/2018/10/17/seguridad-web-en-spring-boot/" target="_blank" rel="noopener">http://www.profesor-p.com/2018/10/17/seguridad-web-en-spring-boot/ </a>recordareis que creábamos una clase que heredaba de **WebSecurityConfigurerAdapter** , donde sobrescribiamos la función **UserDetailsService userDetailsService().**

```@EnableWebSecurity
public class WebSecurityConfiguration extends WebSecurityConfigurerAdapter {
 ....
    @Bean
    @Override
    public UserDetailsService userDetailsService() {
    	
    	UserDetails user=User.builder().username("user").password(passwordEncoder().encode("secret")).
    			roles("USER").build();
    	UserDetails userAdmin=User.builder().username("admin").password(passwordEncoder().encode("secret")).
    			roles("ADMIN").build();
        return new InMemoryUserDetailsManager(user,userAdmin);
    }
....
}
```

Pues los usuarios con sus roles o grupos se definen de la misma manera. Deberemos tener una clase que extienda **WebSecurityConfigurerAdapter ** y definir nuestros usuarios.

Ahora y podemos comprobar si nuestro servidor de autorizaciones funciona. Vamos  a ver como, utilizando el excelente programa **PostMan.**

Para hablar con el &#8216;banco&#8217; para solicitar nuestras credenciales, y como no hemos definido lo contrario, deberemos ir a la URI &#8220;/oauth/token&#8221;. Este es uno de los puntos finales de los que hablaba anteriormente. Hay más pero en nuestro ejemplo y como solo vamos a usar el servicio &#8216;password&#8217; no usaremos más.

Usaremos una petición HTTP tipo POST, indicando que queremos usar validación básica, pondremos el usuario y contraseña, que serán los del &#8220;banco&#8221;, en nuestro ejemplo: &#8216;cliente&#8217; y &#8216;password&#8217; respectivamente.

![](/img/2018/10/Captura-14.png)

En el cuerpo de la petición, en formato form-url-encoded introduciremos el servicio a solicitar, nuestro usuario y nuestra contraseña.

![](/img/2018/10/Captura-15.png)

y lanzamos la petición, la cual nos deberá sacar una salida como esta:

![](/img/2018/10/Captura-16.png)

Ese &#8216;access_token&#8217; &#8220;**8279b6f2-013d-464a-b7da-33fe37ca9afb**&#8221; es nuestra tarjeta de crédito y es la que deberemos presentar a nuestro servidor de recursos (la tienda) para poder ver paginas (recursos) que no sean públicos.

### Creando un Servidor de Recursos (ResourceServer)

Ahora que ya tenemos nuestra tarjeta de crédito vamos a crear la tienda que acepte esa tarjeta 😉

En nuestro ejemplo vamos a crear el servidor de recursos y de autentificación en el mismo programa, con lo cual Spring Boot, se encargara de hacer que confié una parte en otra, sin tener que configurar nada. Si, como es habitual en la vida real, el servidor de recursos esta en un sitio  y el servidor de autentificaciones en otro, deberíamos indicarle al servidor de recursos, cual es nuestro &#8216;banco&#8217; y como debe hablar con el. Pero eso lo dejaremos para otra entrada.

La única clase del servidor de recursos es **ResourceServerConfiguration**

```
@EnableResourceServer
@RestController
public class ResourceServerConfiguration extends ResourceServerConfigurerAdapter
{
.....
}
```

Observar la anotación  @**EnableResourceServer** que hara que Spring active el servidor de recursos.  La etiqueta **@RestController** es porque en esta misma clase nosotros tendremos los recursos, pero podrían estar perfectamente en otra clase.

Por ultimo fijaros que la clase extiende de **ResourceServerConfigurerAdapter** esto es así porque vamos a sobrescribir metodos de esa clase para configurar nuestro servidor de recursos.

Como he dicho antes al estar el servidor de autentificación y de recursos en el mismo programa no tenemos mas que configurar la seguridad de nuestro servidor de recursos. Esto se hace en la función:

```
@Override
public void configure(HttpSecurity http) throws Exception {
	http
	.authorizeRequests().antMatchers("/oauth/token", "/oauth/authorize**", "/publica").permitAll();  
//	 .anyRequest().authenticated(); 

	http.requestMatchers().antMatchers("/privada")  //  Denegamos el acceso a "/privada"
	.and().authorizeRequests()
	.antMatchers("/privada").access("hasRole('USER')") 
	.and().requestMatchers().antMatchers("/admin") // Denegamos el acceso a "/admin"
	.and().authorizeRequests()
	.antMatchers("/admin").access("hasRole('ADMIN')");
}
```

En la entrada anterior cuando definíamos la seguridad en la web, explicaba una función llamada **configure(HttpSecurity http)**, ¿ a que se parece mucho a esta ?. Pues si básicamente es la misma, y de hecho recibe un objeto HttpSecurity que debemos configurar.

Explico linea a linea las sentencias:

  * **http.authorizeRequests().antMatchers("/oauth/token", "/oauth/authorize", "/publica").permitAll()** Permitimos todas las peticiones a  &#8220;/oauth/token&#8221;, &#8220;/oauth/authorize**&#8221;, &#8220;/publica&#8221; sin ningún tipo de validación.
  * **anyRequest().authenticated()** Esta linea esta comentada, si no lo estuviera todos los recursos serian accesibles solo si se el usuario ha sido validado.
  * **requestMatchers().antMatchers("/privada")** Denegamos el acceso a la url &#8220;/privada&#8221;
  * **authorizeRequests().antMatchers("/privada").access("hasRole('USER')")**  Permitimos el acceso a &#8220;/privada&#8221; si el usuario validado tiene el role &#8216;USER&#8217;
  * **requestMatchers().antMatchers("/admin")**  Denegamos el acceso a la url &#8220;/admin&#8221;
  * **authorizeRequests().antMatchers("/admin").access("hasRole('ADMIN')")**  Permitimos el acceso a &#8220;/admin&#8221; si el usuario validado tiene el role &#8216;ADMIN&#8217;

Una vez que tenemos nuestra servidor de recursos creado solo debemos crear los servicios lo cual se hace con estas lineas:

```
  @RequestMapping("/publica")
  public String publico() {
	   return "Pagina Publica";
  }
  @RequestMapping("/privada")
  public String privada() {
         return "Pagina Privada";
  }
  @RequestMapping("/admin")
  public String admin() {
	    return "Pagina Administrador";
  }
```
Como veis son 3 funciones basicas que solo devuelven sus correspondientes Strings.

Veamos ahora como funciona la validación.

Primero comprobamos que podemos acceder a &#8220;/publica&#8221; sin ningún tipo de validación:

![](/img/2018/10/Captura-18.png)

Correcto. ¡¡ Esto funciona!!

Si intento acceder a la pagina &#8220;/privada&#8221; recibo un error &#8220;401 unauthorized&#8221;, lo cual nos indica que no tenemos permiso para ver esa pagina, así que vamos a usar  el token emitido por nuestro servidor de autorizaciones, para el usuario &#8216;user&#8217;, a ver que pasa 😉

![](/img/2018/10/Captura-19.png)

Anda, si podemos ver nuestra pagina privada. Probemos con la pagina del administrador&#8230;

![](/img/2018/10/Captura-20.png)

Correcto, no podemos verla. Asi que vamos a solicitar un nuevo token al administrador de credenciales, pero identificandonos con el usuario &#8216;admin&#8217;.

El token devuelto es: &#8221; &#8220;ab205ca7-bb54-4d84-a24d-cad4b7aeab57&#8221;. Lo usamos a ver que pasa:

![](/img/2018/10/Captura-21.png)

Bueno, pues ya esta, ya podemos ir de compras con seguridad!!. Ahora ya solo falta montar la tienda y tener los productos 😉

Nos vemos en la próxima, estudiantes 🙂

 [1]: http://www.profesor-p.com/2018/10/17/seguridad-web-en-spring-boot/