---
title: Usando Zuul para redirigir peticiones REST
pre: "<b>o </b>"
author: El Profe
type: post
date: 2019-03-16T07:19:41+00:00
url: /2019/03/16/zuul-para-redirigir-peticiones-rest-en-spring-boot/
categories:
  - cloud
  - java
  - json
  - rest
  - seguridad
  - spring boot
  - zuul
tags:
  - cloud
  - gateway
  - java
  - spring boot
  - zuul

---
En este articulo explicare como crear una pasarela para peticiones REST (una _gateway_) utilizando **Zuul**.

**[Zuul][1]** es parte del paquete [Spring Cloud NetFlix][2] y permite redirigir peticiones **REST**, realizando diversos tipos de filtros.

En casi cualquier proyecto donde haya microservicios, es deseable que todas las comunicaciones entre esos microservicios pasen por un lugar común, de tal manera que se registren las entradas y salidas, se pueda implementar seguridad o se puedan redirigir las peticiones dependiendo de diversos parámetros.

Con **Zuul** esto es muy fácil de implementar ya que esta perfectamente integrado con _Spring Boot_.

Como siempre [en mi página de GitHub][3] podéis ver los fuentes sobre los que esta basado este articulo.

### Creando el proyecto.

Si tenemos instalado _Eclipse_ con el [plugin de _Spring Boot_][4] (lo cual recomiendo), el crear el proyecto seria tan fácil como añadir un nuevo proyecto del tipo _Spring Boot_ incluyendo el _starter_ **Zuul**. Para poder hacer algunas pruebas también incluiremos el _starter_ **Web**, como se ve en la imagen:

![Eclipse Plugin Spring Boot][5]

Tambien tenemos la opción de crear un proyecto Maven desde la página web <a class="url" href="https://start.spring.io/" target="_blank" rel="noopener noreferrer">https://start.spring.io/</a> que luego importaremos desde nuestro IDE preferido.

![Crear proyecto Maven desde start.spring.io][6]

### Empezando

Partiendo que nuestro programa esta escuchando en <a class="url" href="http://localhost:8080/" target="_blank" rel="noopener noreferrer">http://localhost:8080/</a> , vamos a a suponer que queremos que todo lo que vaya a la _URL_, <a class="url" href="http://localhost:8080/google" target="_blank" rel="noopener noreferrer">http://localhost:8080/google</a> sea redirigida a <a class="url" href="https://www.google.com" target="_blank" rel="noopener noreferrer">https://www.google.com</a>.

Para ello deberemos crear el fichero `application.yml` dentro del directorio _resources_, como se ve en la imagen

![Estructura del proyecto][7]

En este fichero incluiremos las siguientes líneas:

    zuul:  
      routes:
        google:
          path: /google/**
          url: https://www.google.com/
    

Con ellas especificaremos que todo lo que vaya a la ruta **/google/** y algo más (**) sea redirigido a **<a class="url" href="https://www.google.com/" target="_blank" rel="noopener noreferrer">https://www.google.com/</a>** , teniendo en cuenta que si por ejemplo la petición es realizada `http://localhost:8080/google/search?q=profesor_p` esta será redirigida a `https://www.google.com/search?q=profesor_p`. Es decir lo que añadamos después de **/google/** será incluido en la redirección, debido a los dos asteriscos añadidos al final del path.

Para que el programa funcione solo será necesario añadir la anotación `@EnableZuulProxy`en la clase de inicio, en este caso en: **ZuulSpringTestApplication**

```
import org.springframework.cloud.netflix.zuul.EnableZuulProxy;
@SpringBootApplication
@EnableZuulProxy
public class ZuulSpringTestApplication {
	public static void main(String[] args) {
		SpringApplication.run(ZuulSpringTestApplication.class, args);
	}
}
```

Para poder demostrar las diversas funcionalidades de ZUUL, en <a class="url" href="http://localhost:8080/api" target="_blank" rel="noopener noreferrer">http://localhost:8080/api</a> estará escuchando un servicio REST que esta implementada en la clase **TestController** de este proyecto. Esta clase simplemente devuelve en el cuerpo, los datos de la petición recibida.

```
@RestController
public class TestController {
	final  static String SALTOLINEA="\n";
	
	Logger log = LoggerFactory.getLogger(TestController.class); 
	@RequestMapping(path="/api")
	public String test(HttpServletRequest request)
	{
		StringBuffer strLog=new StringBuffer();
		
		strLog.append("................ RECIBIDA PETICION EN /api ......  "+SALTOLINEA);
		strLog.append("Metodo: "+request.getMethod()+SALTOLINEA);
		strLog.append("URL: "+request.getRequestURL()+SALTOLINEA);
		strLog.append("Host Remoto: "+request.getRemoteHost()+SALTOLINEA);
		strLog.append("----- MAP ----"+SALTOLINEA);
		request.getParameterMap().forEach( (key,value) -&gt;
		{
			for (int n=0;n&lt;value.length;n++)
			{
				strLog.append("Clave:"+key+ " Valor: "+value[n]+SALTOLINEA);
			}
		} );
		
		strLog.append(SALTOLINEA+"----- Headers ----"+SALTOLINEA);
		Enumeration&lt;String&gt; nameHeaders=request.getHeaderNames();				
		while (nameHeaders.hasMoreElements())
		{
			String name=nameHeaders.nextElement();
			Enumeration&lt;String&gt; valueHeaders=request.getHeaders(name);
			while (valueHeaders.hasMoreElements())
			{
				String value=valueHeaders.nextElement();
				strLog.append("Clave:"+name+ " Valor: "+value+SALTOLINEA);
			}
		}
		try {
			strLog.append(SALTOLINEA+"----- BODY ----"+SALTOLINEA);
			BufferedReader reader= request.getReader();
			if (reader!=null)
			{
				char[] linea= new char[100];
				int nCaracteres;
				while  ((nCaracteres=reader.read(linea,0,100))&gt;0)
				{				
					strLog.append( linea);
					
					if (nCaracteres!=100)
						break;
				} 
			}
		} catch (Throwable e) {
			e.printStackTrace();
		}
		log.info(strLog.toString());
		
		return SALTOLINEA+"---------- Prueba de ZUUL ------------"+SALTOLINEA+
				strLog.toString();
	}
}
```

### Filtrando: Dejando logs

En esta parte vamos a ver como crear un filtro de tal manera que se deje un registro de las peticiones realizadas.

Para ello crearemos la clase `PreFilter.java` la cual debe extender de **ZuulFilter**

    
    public class PreFilter extends ZuulFilter {
    	Logger log = LoggerFactory.getLogger(PreFilter.class); 
    	@Override
    	public Object run() {		
    		 RequestContext ctx = RequestContext.getCurrentContext();	    
    	     StringBuffer strLog=new StringBuffer();
    	     strLog.append("\n------ NUEVA PETICION ------\n");	    	    	    
    	     strLog.append(String.format("Server: %s Metodo: %s Path: %s \n",ctx.getRequest().getServerName()	    		 
    					,ctx.getRequest().getMethod(),
    					ctx.getRequest().getRequestURI()));
    	     Enumeration<String> enume= ctx.getRequest().getHeaderNames();
    	     String header;
    	     while (enume.hasMoreElements())
    	     {
    	    	 header=enume.nextElement();
    	    	 strLog.append(String.format("Headers: %s = %s \n",header,ctx.getRequest().getHeader(header)));	    			
    	     };	  	    
    	     log.info(strLog.toString());
    	     return null;
    	}
    
    	@Override
    	public boolean shouldFilter() {		
    		return true;
    	}
    
    	@Override
    	public int filterOrder() {
    		return FilterConstants.SEND_RESPONSE_FILTER_ORDER;
    	}
    
    	@Override
    	public String filterType() {
    		return "pre";
    	}
    
    }
    

En esta clase deberemos sobrescribir las funciones que vemos en el fuente. A continuación explico que haremos en cada de ellas

  * **public Object run()**Aquí pondremos lo que queremos que se ejecute por cada petición recibida. En ella podremos ver el contenido de la petición y manipularla si fuera necesario.
  * **public boolean shouldFilter()**Si devuelve **true** se ejecutara la función **run** .
  * **public int filterOrder()**Devuelve cuando que se ejecutara este filtro, pues normalmente hay diferentes filtros, para cada tarea. Hay que tener en cuenta que ciertas redirecciones o cambios en la petición hay que hacerlas en ciertos ordenes, por la misma lógica que tiene **zuul** a la hora de procesar las peticiones.
  * **public String filterType()** Especifica cuando se ejecutara el filtro. Si devuelve &#8220;pre&#8221; se ejecutara antes de que se haya realizado la redirección y por lo tanto antes de que se haya llamado al servidor final (a google en nuestro ejemplo).Si devuelve &#8220;post&#8221; se ejecutara después de que el servidor haya respondido a la petición.En la clase `org.springframework.cloud.netflix.zuul.filters.support.FilterConstants` tenemos definidos los tipos a devolver, PRE\_TYPE , POST\_TYPE,ERROR\_TYPE o ROUTE\_TYPE.

En la clase de ejemplo vemos como antes de realizar la petición al servidor final, se registran algunos datos de la petición, dejando un log con ellos.

Por último, para que Spring Boot utilize este filtro debemos añadir la función siguiente en nuestra clase principal.

    @Bean
    public PreFilter preFilter() {
            return new PreFilter();
     }
    

**Zuul** buscara _beans_ hereden de la clase **ZuulFilter** y los usara.

En este ejemplo, también esta la clase **PostFilter.java** que implementa otro filtro pero que se ejecuta después de realizar la petición al servidor final. Como he comentado esto se consigue devolviendo &#8220;post&#8221; en la función **filterType()**.

Para que **Zuul** use esta clase deberemos crear otro _bean_ con una función como esta:

     @Bean
     public PostFilter postFilter() {
            return new PostFilter();
     }
    

Recordar que también hay un filtro para tratar los errores y otro para tratar justo después de la redirección (&#8220;route&#8221;), pero en este articulo solo hablare de los filtros tipo &#8220;**post**&#8221; y tipo &#8220;**pre**&#8221;

Aclarar que aunque no lo trato en este articulo con **Zuul** no solo podemos redirigir hacia URL estáticas sino también a servicios, suministrados por Eureka Server, del cual hable en un articulo articulo. Además se integra con Hystrix para tener tolerancia a fallos, de tal manera que si no puede alcanzar un servidor se puede especificar que acción tomar.

  * ## Filtrando. Implementando seguridad

Añadamos una nueva redirección al fichero **application.yml**

     sensitiveHeaders: usuario, clave
      privado:
          path: /privado/**
          url: http://www.profesor-p.com  
    

Esta redirección llevara cualquier petición tipo <a class="url" href="http://localhost:8080/privado/LO_QUE_SEA" target="_blank" rel="noopener noreferrer">http://localhost:8080/privado/LO_QUE_SEA</a> a la pagina donde esta este articulo (<a class="url" href="http://www.profesor-p.com" target="_blank" rel="noopener noreferrer">http://www.profesor-p.com</a> )

La linea `sensitiveHeaders` la explicare más adelante.

En la clase `PreRewriteFilter`he implementando otro filtro tipo **pre** que trata solo esta redirección. ¿ como ?. Fácil, poniendo este código en la función `shouldFilter()`

    @Override
    public boolean shouldFilter() {				
    	return RequestContext.getCurrentContext().getRequest().getRequestURI().startsWith("/privado");
    }
    

Ahora en la función **run** incluimos el siguiente código

```
    Logger log = LoggerFactory.getLogger(PreRewriteFilter.class); 
	@Override
	public Object run() {		
		 RequestContext ctx = RequestContext.getCurrentContext();	    
	     StringBuffer strLog=new StringBuffer();
	     strLog.append("\n------ FILTRANDO ACCESO A PRIVADO - PREREWRITE FILTER  ------\n");	    
	     
	     try {	    	
    		 String url=UriComponentsBuilder.fromHttpUrl("http://localhost:8080/").path("/api").build().toUriString();
    		 String usuario=ctx.getRequest().getHeader("usuario")==null?"":ctx.getRequest().getHeader("usuario");
    		 String password=ctx.getRequest().getHeader("clave")==null?"":ctx.getRequest().getHeader("clave");
    		 
    	     if (! usuario.equals(""))
    	     {
    	    	if (!usuario.equals("profesorp") || !password.equals("profe"))
    	    	{
	    	    	String msgError="Usuario y/o contraseña invalidos";
	    	    	strLog.append("\n"+msgError+"\n");	  
	    	    	ctx.setResponseBody(msgError);
	    	    	ctx.setResponseStatusCode(HttpStatus.FORBIDDEN.value());
	    	    	ctx.setSendZuulResponse(false); 
	    	    	log.info(strLog.toString());	    	    	
	    	    	return null;
    	    	}
    	    	ctx.setRouteHost(new URL(url));
    	     }	    	     	    	
		} catch ( IOException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
	    
	     log.info(strLog.toString());
	     return null;
	}
```

Esta función busca en las cabeceras de la petición (_headers_) si  existe la cabecera **usuario**, en caso de no encontrarla no hace nada con lo cual redireccionara a `http://www.profesor-p.com` como se indica en el filtro. En el caso de que exista la cabecera **usuario** con el valor `profesorp`y que la variable **clave** tenga el valor `profe`, se redirigirá a `http://localhost:8080/api`. En caso contrario devolverá un código HTTP **FORBIDEN** devolviendo la cadena `"Usuario y/o contraseña invalidos"` en el cuerpo de la respuesta HTTP. Ademas se cancela el flujo de la petición debido a que se llama a **ctx.setSendZuulResponse(false)**

Debido a la linea **sensitiveHeaders** del fichero **application.yml** que he mencionado anteriormente las cabeceras &#8216;**usuario**&#8216; y &#8216;**clave**&#8216; no serán pasadas en el flujo de la petición.

Es muy importante que este filtro se ejecute despues del filtro de PRE_DECORATION, pues en caso contrario la llamada a <code class="language-java" lang="java">ctx.setRouteHost()</code> no tendra efecto. Por ello en la función filterOrder tenemos este código:

```
@Override
public int filterOrder() {
	return FilterConstants.PRE_DECORATION_FILTER_ORDER+1; 
}
```

Así una llamada pasando el usuario y la constraseña correctas, nos redirigira a http://localhost:8080/api

```
> curl -s -H "usuario: profesorp" -H "clave: profe" localhost:8080/privado

---------- Prueba de ZUUL ------------
................ RECIBIDA PETICION EN /api ......
Metodo: GET
URL: http://localhost:8080/api
Host Remoto: 127.0.0.1
----- MAP ----

----- Headers ----
Clave:user-agent Valor: curl/7.63.0
Clave:accept Valor: */*
Clave:x-forwarded-host Valor: localhost:8080
Clave:x-forwarded-proto Valor: http
Clave:x-forwarded-prefix Valor: /privado
Clave:x-forwarded-port Valor: 8080
Clave:x-forwarded-for Valor: 0:0:0:0:0:0:0:1
Clave:accept-encoding Valor: gzip
Clave:host Valor: localhost:8080
Clave:connection Valor: Keep-Alive

----- BODY ----
```

Si se pone mal la contraseña la salida seria esta:

```
> curl -s -H "usuario: profesorp" -H "clave: ERROR" localhost:8080/privado
Usuario y/o contraseña invalidos
```

###Filtrando. Filtrado dinámico

Para terminar incluiremos dos nuevas redirecciones en el fichero `applicaction.yml`

     local:
        path: /local/**
        url: http://localhost:8080/api
     url:
        path: /url/**
        url: http://url.com
    

En la primera cuando vayamos a la URL `http://localhost:8080/local/LO_QUE_SEA` seremos redirigidos a `http://localhost:8080/api/LO_QUE_SEA`. Aclarar que la etiqueta `local:`es arbitraria y podría poner `pepe` no teniendo porque coincidir con el _path_ que deseamos redirigir.

En la segunda cuando vayamos a la URL `http://localhost:8080/url/LO_QUE_SEA` seremos redirigidos a `http://localhost:8080/api/LO_QUE_SEA`

La clase **RouteURLFilter** sera la encargada de realizar tratar el filtro URL. Recordar que para que **Zuul** utilize los filtros debemos crear su correspondiente _bean._
```
@Bean
 public RouteURLFilter routerFilter() {
        return new RouteURLFilter();
 }
```

En la función **shouldFilter** de **RouteURLFilter** tendremos este código para que trate solo las peticiones a **/url.** 

    @Override
    	public boolean shouldFilter() {
    		RequestContext ctx = RequestContext.getCurrentContext();
    		if ( ctx.getRequest().getRequestURI() == null || ! ctx.getRequest().getRequestURI().startsWith("/url"))
    			return false;
    		return ctx.getRouteHost() != null
    				&& ctx.sendZuulResponse();
    	}
    

Este filtro será declarado del tipo **pre** en la función **filterType** por lo cual se ejecutara después de los filtros _pre_ y antes de ejecutar la redirección y llamar al servidor final.

    	@Override
    	public String filterType() {
    		return FilterConstants.PRE_TYPE;
    	}
    

En la función **run** esta el código que realiza la magia. Una vez hayamos capturado la _URL_ de destino y el _path_, como explico más adelante, es utilizada la función **setRouteHost()** del **RequestContext** para redirigirla adecuadamente.

    	@Override
    	public Object run() {
    		try {
    			RequestContext ctx = RequestContext.getCurrentContext();
    			URIRequest uriRequest;
    			try {
    				uriRequest = getURIRedirection(ctx);
    			} catch (ParseException k) {
    				ctx.setResponseBody(k.getMessage());
    				ctx.setResponseStatusCode(HttpStatus.BAD_REQUEST.value());
    				ctx.setSendZuulResponse(false);
    				return null;
    			}
    
    			UriComponentsBuilder uriComponent = UriComponentsBuilder.fromHttpUrl(uriRequest.getUrl());
    			if (uriRequest.getPath() == null)
    				uriRequest.setPath("/");
    			uriComponent.path(uriRequest.getPath());
    
    			String uri = uriComponent.build().toUriString();
    			ctx.setRouteHost(new URL(uri));
    		} catch (IOException k) {
    			k.printStackTrace();
    		}
    		return null;
    	}
    

Si encuentra en el _header_ la variable `hostDestino` será donde mandara la petición recibida. También buscara en la cabecera de la petición la variables `pathDestino` para  añadirla al `hostDestino`.

Por ejemplo, supongamos una petición como esta:

    > curl --header "hostDestino: http://localhost:8080" --header "pathDestino: api" \    localhost:8080/url?nombre=profesorp
    

La llamada será redirigida a <a class="url" href="http://localhost:8080/api?q=profesor-p" target="_blank" rel="noopener noreferrer">http://localhost:8080/api?q=profesor-p</a> y mostrara la siguiente salida:

    ---------- Prueba de ZUUL ------------
    ................ RECIBIDA PETICION EN /api ......
    Metodo: GET
    URL: http://localhost:8080/api
    Host Remoto: 127.0.0.1
    ----- MAP ----
    Clave:nombre Valor: profesorp
    
    ----- Headers ----
    Clave:user-agent Valor: curl/7.60.0
    Clave:accept Valor: */*
    Clave:hostdestino Valor: http://localhost:8080
    Clave:pathdestino Valor: api
    Clave:x-forwarded-host Valor: localhost:8080
    Clave:x-forwarded-proto Valor: http
    Clave:x-forwarded-prefix Valor: /url
    Clave:x-forwarded-port Valor: 8080
    Clave:x-forwarded-for Valor: 0:0:0:0:0:0:0:1
    Clave:accept-encoding Valor: gzip
    Clave:host Valor: localhost:8080
    Clave:connection Valor: Keep-Alive
    
    ----- BODY ----
    
    

También puede recibir la URL a redireccionar en el cuerpo de la petición. El objeto JSON recibido debe tener el formato definido por la clase **GatewayRequest** que a su vez contiene un objeto **URIRequest**

    public class GatewayRequest {
    	URIRequest uri;
    	String body;
    
    }
    

    public class URIRequest {
    	String url;
    	String path;
    	byte[] body=null;
    

Este es un ejemplo de una redirección poniendo la URL destino en el body:

    curl -X POST \
      'http://localhost:8080/url?nombre=profesorp' \
      -H 'Content-Type: application/json' \
      -d '{
        "body": "El body chuli", "uri": { 	"url":"http://localhost:8080", 	"path": "api"    }
    }'
    

URL: &#8220;<a class="url" href="http://localhost:8080/url?nombre=profesorp" target="_blank" rel="noopener noreferrer">http://localhost:8080/url?nombre=profesorp</a>&#8221;

Cuerpo de la petición:

<pre><code class="language-json" lang="json">{
 "body": "El body chuli",
    "uri": {
    	"url":"http://localhost:8080",
    	"path": "api"
    }
}
</code></pre>

La salida recibida será:

    ---------- Prueba de ZUUL ------------
    ................ RECIBIDA PETICION EN /api ......
    Metodo: POST
    URL: http://localhost:8080/api
    Host Remoto: 127.0.0.1
    ----- MAP ----
    Clave:nombre Valor: profesorp
    
    ----- Headers ----
    Clave:user-agent Valor: curl/7.60.0
    Clave:accept Valor: */*
    Clave:content-type Valor: application/json
    Clave:x-forwarded-host Valor: localhost:8080
    Clave:x-forwarded-proto Valor: http
    Clave:x-forwarded-prefix Valor: /url
    Clave:x-forwarded-port Valor: 8080
    Clave:x-forwarded-for Valor: 0:0:0:0:0:0:0:1
    Clave:accept-encoding Valor: gzip
    Clave:content-length Valor: 91
    Clave:host Valor: localhost:8080
    Clave:connection Valor: Keep-Alive
    
    ----- BODY ----
    El body chuli
    
    

Como se ve el cuerpo es tratado y al servidor final solo es mandado lo que se envía en el parámetro `body` de la petición **JSON**

Como se ve, **Zuul** tiene mucha potencia y es una excelente herramienta para realizar redirecciones. En este articulo solo he arañado las principales características de esta fantástica herramienta, pero espero que haya servido para ver las posibilidades que ofrece.

¡¡Nos vemos en la próxima entrada!!

 [1]: https://cloud.spring.io/spring-cloud-netflix/multi/multi__router_and_filter_zuul.html
 [2]: https://spring.io/projects/spring-cloud-netflix
 [3]: https://github.com/chuchip/zuulSpringTest
 [4]: https://marketplace.eclipse.org/content/spring-tools-4-spring-boot-aka-spring-tool-suite-4
 [5]: https://raw.githubusercontent.com/chuchip/zuulSpringTest/master/starters.png
 [6]: https://raw.githubusercontent.com/chuchip/zuulSpringTest/master/springio.png
 [7]: https://raw.githubusercontent.com/chuchip/zuulSpringTest/master/estructura_ficheros.png