---
title: Microservicios distribuidos con Eureka
author: El Profe
type: post
date: 2019-01-03T08:52:59+00:00
url: /2019/01/03/microservicios-distribuidos-con-eureka/
categories:
  - java
  - rest
  - spring boot
tags:
  - java
  - microservicios
  - rest
  - spring boot

---
En esta _clase_ hablare de como crear **microservicios** distribuidos en **Spring Boot** utilizando las facilidades que nos da el paquete [Spring Cloud NetFlix][1].

Cualquier microservicio debe poder localizar las diferentes instancias de otro servicio del que dependa sin tener sus direcciones definidas en el código.

En el caso de que un microservicio deba acceder a otro lo ideal seria que de alguna manera pudiera saber en que direcciones esta las instancias de ese otro microservicio funcionando, pues lo más común es que se levanten diferentes instancias dependiendo de la carga. 

Para ello en **Spring** se utiliza **Eureka Server** del paquete [Spring Cloud NetFlix][1]. Utilizando este paquete además de **Ribbon** y **Feign** conseguiremos que nuestra aplicación sea capaz de encontrar las diferentes instancias de un microservicio y balancear las peticiones de tal manera que se reparta la carga.

En este articulo voy a explicar como crear un servicio que al que llamaremos para solicitar la capital de un país. Este servicio a su vez llamara a otro servicio para localizar los datos solicitados, pues el solo será un punto de entrada.

Los programas utilizados serán estos:

  * **Proyecto**: capitals-service **Puerto:**: 8100 
  * **Proyecto**: countries-service **Puerto:**: 8000 y 8001
  * **proyecto**: eureka-server **Puerto**: 8761 

El proyecto &#8216;**countries-service**&#8216; será el que tenga la base de datos con los datos de los diferentes países. Se lanzaran dos instancias del mismo servicio para que podamos comprobar como &#8216;**capitals-service**&#8216; hace una llamada a una instancia y luego, balanceando la carga.

El código de ejemplo de este articulo esta en [GitHub][2].

  1. **Creando un servidor Eureka**

Lo primero que necesitamos es tener un lugar donde todos los microservicios se registren cuando se inicialicen. Ese servicio es el que a su vez se consultara cuando queramos localizar las diferentes instancias. En esta ejemplo vamos a utilizar **Eureka Server** el cual es muy fácil de crear.

Para ello crearemos un nuevo proyecto **Spring Boot** con tan solo el _Starter_ **Eureka Server**.

En este proyecto cambiaremos el fichero **application.properties** para que incluya las siguientes líneas:

<pre class="wp-block-preformatted">spring.application.name=eureka-server<br />server.port=8761<br />​eureka.client.register-with-eureka=false<br />eureka.client.fetch-registry=false</pre>

Es decir especificamos el nombre del programa con la línea **spring.application.name** . El puerto en el que estará escuchando el servicio con **server.port**. Y lo más importante, pues los anteriores valores son opcionales, los parámetros del servidor Eureka.

  * **eureka.client.register-with-eureka=false** para que el servidor no se intente registrar a si mismo. 
  * **eureka.client.fetch-registry=false** con este parámetro especificamos a los clientes que no se guarden en su cache local las direcciones de los diferentes instancias. Esto es para que consulte al servidor Eureka cada vez que necesite acceder a un servicio. En producción a menudo se pone a **true** para agilizar las peticiones. Comentar que esa cache se actualiza cada 30 segundos por defecto.

Ahora en nuestra clase principal, por donde entra **Spring Boot** deberemos poner las anotación **EnableEurekaServer**: 

<pre class="wp-block-preformatted">@SpringBootApplication<br />@EnableEurekaServer<br />public class NetflixEurekaNamingServerApplication {<br />​<br />    public static void main(String[] args) {<br />        SpringApplication.run(NetflixEurekaNamingServerApplication.class, args);<br />    }<br />}</pre>

¡Y ya esta listo!. Nuestro servidor Eureka esta creado. Para ver su estado podemos usar nuestro navegador preferido y navegar a: <http://localhost:8761/> para ver las aplicaciones que se han registrado. Como se ve en la captura de pantalla todavía no hay ninguna.<figure class="wp-block-image">

<img src="http://www.profesor-p.com/wp-content/uploads/2019/01/captura2-1-1024x504.png" alt="" class="wp-image-536" srcset="http://www.profesor-p.com/wp-content/uploads/2019/01/captura2-1-1024x504.png 1024w, http://www.profesor-p.com/wp-content/uploads/2019/01/captura2-1-300x148.png 300w, http://www.profesor-p.com/wp-content/uploads/2019/01/captura2-1-768x378.png 768w, http://www.profesor-p.com/wp-content/uploads/2019/01/captura2-1.png 1175w" sizes="(max-width: 1024px) 100vw, 1024px" /></figure> 

En la misma pantalla se muestra el estado del servidor.<figure class="wp-block-image">

<img src="http://www.profesor-p.com/wp-content/uploads/2019/01/captura5-1024x427.png" alt="" class="wp-image-539" srcset="http://www.profesor-p.com/wp-content/uploads/2019/01/captura5-1024x427.png 1024w, http://www.profesor-p.com/wp-content/uploads/2019/01/captura5-300x125.png 300w, http://www.profesor-p.com/wp-content/uploads/2019/01/captura5-768x320.png 768w" sizes="(max-width: 1024px) 100vw, 1024px" /></figure> 

Observar que lo normal es que tengamos varios servidores Eureka levantados. En nuestro ejemplo solo levantaremos uno, aunque eso nos será lo normal en producción.

### **2. Microservicio &#8216;countries-service&#8217;** 

Ahora que tenemos nuestro servidor vamos a crear nuestro primer cliente. Para ello crearemos otro proyecto de **Spring Boot** con los siguientes _starters_ 

  * Eureka Discovery
  * Web
  * Lombok
  * H2
  * JPA

Como he comentado anteriormente, este microservicio es el que va a tener la base de datos y el que será consultado por &#8216;capitales-service&#8217; para buscar las capitales de un país.

Lo destacable de este proyecto esta en el fichero `application.properties` de **Spring Boot**

<pre class="wp-block-preformatted">spring.application.name=paises-service<br />eureka.client.serviceUrl.defaultZone=http://localhost:8761/eureka<br />server.port=8000<br /># Configuacion JPA<br />spring.jpa.show-sql=true<br />spring.h2.console.enabled=true</pre>

Como se puede ver, con el paramero **eureka.client.service-url.default-zone** especificamos donde esta el servidor Eureka. **Spring Boot** automáticamente al ver que tiene el paquete **Eureka Discovery** disponible intentara registrarse en su correspondiente servidor.

Para poder lanzar con **Eclipse** la segunda instancia de la aplicación **paises-service** en el puerto 8001, deberemos ir a la opción `Run Configurations` en el menú `Run`y copiar la que Eclipse habra creado de **countries-service** una vez hayamos ejecutado la aplicación por primera vez. En la pestaña `Arguments` deberemos añadir el parámetro `--server.port=8001`<figure class="wp-block-image">

<img src="http://www.profesor-p.com/wp-content/uploads/2019/01/captura4.png" alt="" class="wp-image-538" srcset="http://www.profesor-p.com/wp-content/uploads/2019/01/captura4.png 801w, http://www.profesor-p.com/wp-content/uploads/2019/01/captura4-300x206.png 300w, http://www.profesor-p.com/wp-content/uploads/2019/01/captura4-768x526.png 768w" sizes="(max-width: 801px) 100vw, 801px" /></figure> 

En la siguiente captura de pantalla se puede ver como si lanzamos dos instancias de este programa, una en el puerto 8000 y otra en el puerto 8001, en **Eureka Server** podemos ver como se han registrado las diferentes instancias. El nombre que se han registrado y por el que el se podrán buscar es el nombre de la aplicación como se ha declarado en la variable `spring.application.name` del fichero `application.properties`<figure class="wp-block-image">

<img src="http://www.profesor-p.com/wp-content/uploads/2019/01/captura5-1024x427.png" alt="" class="wp-image-539" srcset="http://www.profesor-p.com/wp-content/uploads/2019/01/captura5-1024x427.png 1024w, http://www.profesor-p.com/wp-content/uploads/2019/01/captura5-300x125.png 300w, http://www.profesor-p.com/wp-content/uploads/2019/01/captura5-768x320.png 768w" sizes="(max-width: 1024px) 100vw, 1024px" /></figure> 

Así vemos que la aplicación `COUNTRIES-SERVICE`tiene dos instancias, levantadas ambas en el host `port-chuchi`una en el puerto 8000 y otra en el puerto 8001.

_Mi ordenador se llama `port-chuchi`_

Esta sencilla aplicación usara H2 para la persistencia de datos teniendo una simple tabla llamada `countries`con los datos de los países, a la que accederemos por JPA. La estructura de la tabla esta definida en `com.profesorp.countriesservice.entities.Countries.java`

En la clase `CapitalsServiceController`se definen los siguientes puntos de entrada.

  1. Petición GET. **/{country}**

  * **Recibe**: Código de Pais. (&#8216;es&#8217;,&#8217;eu&#8217;,&#8217;en&#8217;&#8230;.)
  * **Devolverá** un objeto `CapitalsBean`<figure class="wp-block-image">

<img src="http://www.profesor-p.com/wp-content/uploads/2019/01/captura3-1.png" alt="" class="wp-image-537" srcset="http://www.profesor-p.com/wp-content/uploads/2019/01/captura3-1.png 616w, http://www.profesor-p.com/wp-content/uploads/2019/01/captura3-1-300x149.png 300w" sizes="(max-width: 616px) 100vw, 616px" /></figure> 

  1. Petición GET. **/time/{time}**

Establece el tiempo que la entrada **/{country}** realizara una pausa antes de devolver el resultado.

### 3. **Microservicio &#8216;capitals-service&#8217;** 

Este servicio es el que llamara al anterior para solicitar todos los datos de un país, pero mostrara solo la capital, el puerto del servicio al que realizo la llamada y el nombre del país.<figure class="wp-block-image">

<img src="http://www.profesor-p.com/wp-content/uploads/2019/01/captura9.png" alt="" class="wp-image-543" srcset="http://www.profesor-p.com/wp-content/uploads/2019/01/captura9.png 420w, http://www.profesor-p.com/wp-content/uploads/2019/01/captura9-300x132.png 300w" sizes="(max-width: 420px) 100vw, 420px" /></figure> 

Necesitaremos tener los siguientes _starters_

  * Eureka Discovery
  * Feign
  * Lombok
  * Web

En primer lugar, como en el anterior servicio, en el fichero `application.properties`tendremos el siguiente contenido:

<pre class="wp-block-preformatted">spring.application.name=capitals-service<br />eureka.client.serviceUrl.defaultZone=http://localhost:8761/eureka<br />server.port=8100</pre>

Es decir, definimos el nombre de la aplicación, después especificamos donde esta el servidor Eureka donde nos debemos registrar y por fin el puerto donde escuchara el programa.

  * **Utilizando RestTemplate.**

Para realizar una petición RESTFUL a `countries-service` la forma más simple seria usar la clase `RestTemplate`del paquete `org.springframework.web.client`. 

<pre class="wp-block-preformatted">@GetMapping("/template/{country}")<br />public CapitalsBean getCountryUsingRestTemplate(@PathVariable String country) { <br />    Map&lt;String, String&gt; uriVariables = new HashMap&lt;&gt;();<br />    uriVariables.put("country", country);               <br />    ResponseEntity&lt;CapitalsBean&gt; responseEntity = new RestTemplate().getForEntity(<br />            "http://localhost:8000/{country}", <br />            CapitalsBean.class, <br />            uriVariables );     <br />    CapitalsBean response = responseEntity.getBody();       <br />    return response;<br />}</pre>

Como se ve, simplemente, metemos en un `hashmap` las variables que vamos a pasar en la petición, que en este caso es solo el parámetro `pais`, para después realizar crear un objeto `ResponseEntity` llamando a la función estática`RestTemplate.getForEntity()` pasando como parámetros, la URL que deseamos llamar, la clase donde debe dejar la respuesta de la petición REST y las variables pasadas en la petición. 

Después, capturamos el objeto `CapitalsBean`que tendremos en el _Body_ del objeto `ResponseEntity`.

Pero usando este método tenemos el problema de que debemos tener definido en nuestro programa las URLs donde están las diferentes instancias del microservicio al que llamamos, además como se ve, tenemos que escribir mucho código para hacer una simple llamada. 

  * **Petición FEIGN simple**

Una manera más elegante de hacer esa llamada seria utilizando [Feign][3]. **Feign** es una herramienta de **Spring** que nos permite realizar llamadas usando funciones declarativas.

Para utilizar **Feign** debemos incluir la etiqueta **@EnableFeignClients** en nuestra clase principal. En nuestro ejemplo la ponemos en la clase `CapitalsServiceApplication`

<pre class="wp-block-preformatted">@SpringBootApplication<br />@EnableFeignClients("com.profesorp.capitalsservice")<br />public class CapitalsServiceApplication {<br />    public static void main(String[] args) {<br />        SpringApplication.run(CapitalsServiceApplication.class, args);<br />    }<br />}</pre>

Si no pasamos ningún parámetro a la etiqueta **@EnableFeignClients** buscara clientes **Feign** en nuestro paquete principal, si le ponemos un valor solo buscara clientes en el paquete mandado. Así en el ejemplo solo buscaría en el paquete `com.profesorp.capitalsservice`

Ahora definimos el cliente _Feing_ con el _interface_ `CapitalsServiceProxy`

<pre class="wp-block-preformatted">@FeignClient(name="simpleFeign",url="http://localhost:8000/")<br />public interface CapitalsServiceProxySimple {   <br />    @GetMapping("/{country}")<br />    public CapitalsBean getCountry(@PathVariable("country") String country);<br />}</pre>

Lo primero es etiquetar la clase con **@FeignClient** especificando la URL donde esta el servidor REST que queremos llamar. Prestar atención al hecho de que ponemos la dirección base, en este caso solo el nombre del host y su puerto `localhost:8000`. El parámetro `name`debe ser puesto pero no es importante su contenido.

Después definiremos las diferentes entradas que queremos tener disponibles. En nuestro caso solo hay una definida, pero podríamos incluir la llamada a **/time/{time}** .

Para usar este cliente simplemente pondríamos este código en nuestro programa

<pre class="wp-block-preformatted">@Autowired<br />private CapitalsServiceProxySimple simpleProxy;<br />@GetMapping("/feign/{country}")<br />public CapitalsBean getCountryUsingFeign(@PathVariable String country) {<br />    CapitalsBean response = simpleProxy.getCountry(country);        <br />    return response;<br />}</pre>

Usamos el inyector de dependencias de **Spring** para crear un objeto **CapitalsServiceProxySimple** y después simplemente llamamos a la función `getCountry()`del interface.

Mucho más limpio, ¿verdad?. Suponiendo que nuestro servidor REST tuviera muchos puntos de entrada nos ahorraríamos muchísimo de teclear, además de tener un código mucho más limpio.

Pero aún tenemos el problema de que la dirección del servidor RESTFUL esta escrita en nuestro código lo cual nos hace imposible poder llegar a las diferentes instancias del mismo servicio y nuestro microservicio no será verdaderamente escalable.

  * **Petición FEIGN usando el servidor Eureka** 

Para resolver el problema en vez de poner la dirección del servidor, pondremos el nombre de la aplicación y **Spring Boot** se encargara de llamar el servidor Eureka, pidiéndole la dirección donde esta ese servicio .

Para ello crearíamos un interface **Feign** de esta manera

<pre class="wp-block-preformatted">@FeignClient(name="countries-service")<br />public interface CapitalsServiceProxy {<br />    @GetMapping("/{country}")<br />    public CapitalsBean getCountry(@PathVariable("country") String country);<br />}</pre>

Como se puede ver aquí no especificamos la dirección del servicio, simplemente ponemos el nombre. En este caso `countries-service` que es como esta registrada la aplicación en el servidor Eureka.

Ahora cada petición que se haga ira balanceándose de una instancia a otra. De tal manera que la primera petición ira a la del puerto 8000 y la siguiente a la del puerto 8001.<figure class="wp-block-image">

<img src="http://www.profesor-p.com/wp-content/uploads/2019/01/captura6.png" alt="" class="wp-image-540" srcset="http://www.profesor-p.com/wp-content/uploads/2019/01/captura6.png 539w, http://www.profesor-p.com/wp-content/uploads/2019/01/captura6-300x126.png 300w" sizes="(max-width: 539px) 100vw, 539px" /></figure> <figure class="wp-block-image"><img src="http://www.profesor-p.com/wp-content/uploads/2019/01/captura7.png" alt="" class="wp-image-541" srcset="http://www.profesor-p.com/wp-content/uploads/2019/01/captura7.png 582w, http://www.profesor-p.com/wp-content/uploads/2019/01/captura7-300x142.png 300w" sizes="(max-width: 582px) 100vw, 582px" /></figure> 

De esta manera nuestra aplicación ya utilizara todas las instancias del servicio automáticamente.

  * **Configurando RIBBON** 

El paquete **Feign** usa el paquete [Ribbon][4] por debajo y realmente es este es el que se encarga de balancear las peticiones. Por defecto **Ribbon** usara la regla _RoundRobinRule_. Con esta regla escogerá secuencialmente cada uno de las instancias que **Eureka** le muestre levantadas, sin tener en cuenta el tiempo que a cada instancia le cuesta responder. 

Si deseamos que use alguna de las otras tres disponibles por defecto o incluso una regla que nosotros definamos deberemos crear una clase de configuración para **Ribbon**, como la siguiente:

<pre class="wp-block-preformatted">import org.springframework.context.annotation.Bean;<br />import com.netflix.loadbalancer.IPing;<br />import com.netflix.loadbalancer.IRule;<br />import com.netflix.loadbalancer.NoOpPing;<br />import com.netflix.loadbalancer.WeightedResponseTimeRule;<br />public class RibbonConfiguration {<br />     @Bean<br />     public IPing ribbonPing() {<br />     &nbsp; &nbsp; &nbsp; &nbsp;return new NoOpPing();<br />     }   <br />     @Bean  <br />     public IRule ribbonRule() {<br />     &nbsp; &nbsp; &nbsp; &nbsp;return new WeightedResponseTimeRule();<br />     }<br />}</pre>

En la función `ribbonRule`()devolveremos el objeto `WeightedResponseTimeRule` si queremos que la lógica de balanceo tenga en cuenta el tiempo de respuesta de cada instancia.

Ahora, para especificar que queremos usar esta clase para configurar **Ribbon** añadiremos la etiqueta 

`@RibbonClient(name="countries-service", configuration = RibbonConfiguration.class)`en nuestra clase `CapitalsServiceApplication`

<pre class="wp-block-preformatted">@SpringBootApplication<br />@EnableFeignClients <br />@RibbonClient(name="countries-service", configuration = RibbonConfiguration.class)<br />public class CapitalsServiceApplication {<br />....<br />}</pre>

Para comprobar como funciona el balanceo por peso, estableceremos una pausa de 10 milisegundos al servidor del puerto 8001 y una de 300 al servidor del puerto 8000, usando la llamada a **/time/{time}** del servicio `countries-service`

<pre class="wp-block-preformatted">&gt; curl localhost:8001/time/10<br />&gt; curl localhost:8000/time/300</pre>

Suponiendo que estamos trabajando en **Linux**, usando **Bash** haremos 100 peticiones.

<pre class="wp-block-preformatted">CONTADOR=0; while [ $CONTADOR -lt 100 ]; do <br />    curl http://localhost:8100/es<br />    let CONTADOR=CONTADOR+1<br />done</pre>

Al cabo de un tiempo podremos ver las peticiones que se han realizado a cada puerto llamando a <http://localhost:8100/puertos><figure class="wp-block-image">

<img src="http://www.profesor-p.com/wp-content/uploads/2019/01/captura8.png" alt="" class="wp-image-542" srcset="http://www.profesor-p.com/wp-content/uploads/2019/01/captura8.png 457w, http://www.profesor-p.com/wp-content/uploads/2019/01/captura8-300x81.png 300w" sizes="(max-width: 457px) 100vw, 457px" /></figure> 

Como se puede ver hay muchas más peticiones al puerto 8001 que al puerto 8000, lo cual es normal teniendo en cuenta que el puerto 8000 tiene un retraso de 300 milisegundos, mientras que el 8001 solo de 10.

Para terminar este articulo comentar que **Ribbon** se puede usar sin tener **Feign**, utilizando directamente _RestTemplate_ pero el estudio de ese caso lo dejare para otra ocasión.

Mencionar, además que para realizar pruebas de balanceo he utilizado **Docker** por lo cual en el código fuente de [GitHub][2], veremos que en el fichero `application.properties`del proyecto `countries-service` están estas líneas:

<pre class="wp-block-preformatted">eureka.client.serviceUrl.defaultZone:http://eurekaserver:8761/eureka<br />server.port=${SERVER_PORT}</pre>

En vez de las mostradas anteriormente. Esto esta puesto así para poder definir dinámicamente cuando se lanza el contenedor **docker** , con la variable de entorno **SERVER_PORT** el puerto donde debe escuchar cada instancia.

Gracias por leer este articulo y hasta la próxima lección 😉

 [1]: http://spring.io/projects/spring-cloud-netflix
 [2]: https://github.com/chuchip/springEureka
 [3]: https://cloud.spring.io/spring-cloud-netflix/multi/multi_spring-cloud-feign.html
 [4]: https://cloud.spring.io/spring-cloud-netflix/multi/multi_spring-cloud-ribbon.html