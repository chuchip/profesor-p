---
title: La clase WebClient
pre: "<b>o </b>"
draft: false
weight: 40
author: El Profe
url: /webclient
type: post
date: 2019-08-20
categories:
  - java
  - webflux
  - reactor
  - rest
  - spring
  - web
tags:  
  - webflux
  - reactor
  - java
  - rest
  - spring

---

En esta ocasión hablare de la clase  [WebClient](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/web/reactive/function/client/WebClient.html) de **SpringBoot**.

El proyecto de ejemplo esta disponible en: [https://github.com/chuchip/webClientExample](https://github.com/chuchip/webClientExample)

Esta clase seria la equivalente a **RestTemplate** pero para realizar peticiones asíncronas.

Para poder usar esta clase debemos poner estas dependencias en nuestro fichero **maven**

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-webflux</artifactId>
</dependency>
```

Esto es así porque debemos usar **WebFlux**  el cual esta disponible con la versión 5.0 de **Spring**. Esta versión de **Spring** requiere que usemos al menos **Java 8.0**.

Con [WebFlux](https://docs.spring.io/spring-framework/docs/5.0.0.BUILD-SNAPSHOT/spring-framework-reference/html/web-reactive.html) el cual esta basado en el proyecto [Reactor](https://projectreactor.io/) podemos crear aplicaciones reactivas. Este tipo de aplicaciones se caracteriza porque las peticiones no son bloqueantes y porque se utiliza ampliamente la [programación funcional](https://es.wikipedia.org/wiki/Programaci%C3%B3n_funcional)

Para entender este articulo hay que tener ciertas nociones sobre como funciona **Reactor** y la clase [Mono](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Mono.html) . Aunque si has utilizado **Streams**  en **Java** puedes pensar que un objeto Mono es como un **Stream** que puede emitir un valor o un error.

Pero no voy a profundizar en estos temas que se escapan al ámbito de este articulo, será suficiente con decir que gracias a la clase **WebClient** podremos realizar varias llamadas en paralelo a uno o varios servidores, de tal manera que si a cada llamada le cuesta 2 segundos responder  y realizamos 5 llamadas, podremos conseguir tener todas las respuestas en poco más de 2 segundos, en vez de los 10 que necesitaríamos habitualmente.

### Llamadas en Paralelo

En el programa de ejemplo he creado una parte servidora y otra cliente. La parte servidora se levantara en el puerto 8081 y la parte cliente en el puerto 8080 que es el que tiene por defecto las aplicaciones de Spring Boot.

```
@SpringBootApplication
public class WebServerApplication {
	public static void main(String[] args) {
		new SpringApplicationBuilder(WebServerApplication.class).
				properties(Collections.singletonMap("server.port", "8081")).run(args);
	}
}
```

Si realizamos una  petición REST a la URL `http://localhost:8080/client/XXX` ejecutaremos la función `testGet` de la clase `ClientController` que detallo a continuación:

```
@RestController()
@RequestMapping("/client")
@Slf4j
public class ClientController {
	final String urlServer="http://localhost:8081";
	
	@GetMapping("/{param}")
	public Mono<ResponseEntity<Mono<String>>> testGet(@PathVariable String param) {
		final long dateStarted = System.currentTimeMillis();

		WebClient webClient = WebClient.create(urlServer+"/server/");
		Mono<ClientResponse> respuesta = webClient.get().uri("?queryParam={name}", param).exchange();
		Mono<ClientResponse> respuesta1 = webClient.get().uri("?queryParam={name}","SPEED".equals(param)?"SPEED":"STOP").exchange();
		
		Mono<ResponseEntity<Mono<String>>> f1 = Mono.zip(respuesta, respuesta1)
		.map(t -> {
			if (!t.getT1().statusCode().is2xxSuccessful()) {
				return ResponseEntity.status(t.getT1().statusCode()).body(t.getT1().bodyToMono(String.class));
			}
			if (!t.getT2().statusCode().is2xxSuccessful()) {
				return ResponseEntity.status(t.getT2().statusCode()).body(t.getT2().bodyToMono(String.class));
			}
			return ResponseEntity.ok().body(Mono.just(
					"All OK. Seconds elapsed: " + (((double) (System.currentTimeMillis() - dateStarted) / 1000))));
		});
		return f1;
	}
```

Como se ve es un simple controlador  donde realizamos dos llamadas tipo **get** a la URL **http://localhost:8081**. En la primera llamada se pasa como parámetro lo recibido en la variable **param**. En la segunda se pasa la cadena "**STOP**" si **param** es diferente de **SPEED**. 

El servidor que esta escuchando en el puerto 8081 al recibir como parámetro el texto **STOP** realiza un *sleep* durante 5 segundos.

Como he dicho, las llamadas al servidor se realizan usando la clase **WebClient** . Para ello, al crear la clase especificamos la URL base donde queremos llamar.

```
WebClient webClient = WebClient.create(urlServer+"/server/");
```

Después ejecutamos la llamada del tipo **GET** al servidor, pasándole el parámetro **queryParam**. Por fin, con la llamada a **exchange** recibiremos un objeto **Mono** que contiene una clase **ClientResponse** la cual sería equivalente a la clase **ResponseEntity** de  **RestTemplate**. Es decir, contendrá el código HTTP devuelto por el servidor, el cuerpo y las cabeceras.

```
Mono<ClientResponse> respuesta = webClient.get().uri("?queryParam={name}", param).exchange();
```

Un momento, ¿he dicho que ejecutaremos?. Pues he mentido. Realmente solo se ha  declarado lo que queremos hacer. Precisamente la gracia de la programación reactiva es que hasta que alguien no se subscribe a una petición nada se ejecuta, por lo cual la petición al servidor no se ha realizado todavía.

En la siguiente línea declaramos la segunda llamada al servidor:

```
Mono<ClientResponse> respuesta1 = webClient.get().uri("?queryParam={name}","SPEED".equals(param)?"SPEED":"STOP").exchange();
```

Finalmente creamos un objeto **Mono**  que será el resultado de los dos anteriores, usando la función `zip` .

Usando la función `map` devolveremos un objeto `ResponseEntity` con el código HTTP igual a **OK** si las dos llamadas han respondido con un 2XX o bien el código HTTP devuelto por el servidor y el mensaje devuelto 

Al ser **WebClient** reactivo las dos llamadas se realizan simultáneamente y por lo tanto veremos que aunque hagamos la llamada `curl http://localhost:8080/client/STOP` que realizara sendas llamadas al servidor con el texto **STOP**  con lo cual cada llamara costara 5 segundos, la respuesta de ambos la tendremos disponible en poco más de 5 segundos.

```
 All OK. Seconds elapsed: 5.092
```



### LLamada tipo POST

En la función `testURLs` hay un ejemplo de una llamada usando POST.

Esta función recibe en el cuerpo un `Map` que luego se meterá en las cabeceras de la petición. Además este `map` será mandado en la petición POST que se hará al servidor.

```
@PostMapping("")
	public Mono<String> testURLs(@RequestBody Map<String,String> body,
			@RequestParam(required = false) String url) {		

		log.debug("Client: in testURLs");
		WebClient.Builder builder = WebClient.builder().baseUrl(urlServer).
			defaultHeader(HttpHeaders.CONTENT_TYPE,MediaType.APPLICATION_JSON_VALUE);
		if (body!=null && body.size()>0 )
		{
			for (Map.Entry<String, String> map : body.entrySet() )
			{
				builder.defaultHeader(map.getKey(), map.getValue());
			}
		}
		WebClient webClient = builder.build();	
		String urlFinal;
		if (url==null)
			urlFinal="/server/post";
		else
			urlFinal="/server/"+url;

		Mono<String> respuesta1 = webClient.post().uri(urlFinal).body(BodyInserters.fromObject(body)).exchange()
			.flatMap( x -> 
			{ 
				if ( ! x.statusCode().is2xxSuccessful())
					return 	Mono.just("LLamada a "+urlServer+urlFinal+" Error 4xx: "+x.statusCode()+"\n");
				return x.bodyToMono(String.class);
			});		    	
		return respuesta1;		
	}	
```

Para insertar el cuerpo del mensaje usaremos la clase auxiliar `BodyInserters` si el mensaje estuviera en un objeto **Mono** se podría usar este código:

```
BodyInserters.fromPublisher(Mono.just(MONO_OBJECT),String.class);
```

Al realizar un `flatMap` capturaremos la salida del objeto `ClientResponse` y devolveremos un objeto **Mono** con la cadena que deseamos devolver. La función `flatMap` *aplanara* ese objeto **Mono** recogiendo el **String** que hay en su interior y es por ello que recibiremos un `Mono<String>` y no un `Mono<Mono<String>>` como pasaría su usáramos la función **map**. 

Realizando la siguiente llamada

```
> curl  -s -XPOST http://localhost:8080/client  -H 'Content-Type: application/json' -d'{"aa": "bbd"}'
```

Obtendremos  la salida siguiente:

```
the server said: {aa=bbd}
Headers: content-length:12
Headers: aa:bbd
Headers: accept-encoding:gzip
Headers: Content-Type:application/json
Headers: accept:*/*
Headers: user-agent:ReactorNetty/0.9.0.M3
Headers: host:localhost:8081

```

Esta salida es producida por la función `postExample` del servidor.

```
@PostMapping("post")
public ResponseEntity<String> postExample(@RequestBody Map<String,String> body,ServerHttpRequest  request) {
	String s="the server said: "+body+"\n";
	for (Entry<String, List<String>> map : request.getHeaders().entrySet())
	{
		s+="Headers: "+map.getKey()+ ":"+map.getValue().get(0)+"\n";		
	}		
	return ResponseEntity.ok().body(s);
}
```

Observar que al estar utilizando la librería **WebFlux** que no es totalmente compatible con **javax.servlet** debemos recibir un objeto `ServerHttpRequest` para recoger todas las cabeceras en crudo. El equivalente en  una aplicación no reactiva sería un objeto `HttpServletRequest`

Si ejecutamos la  sentencia:

```
curl  -s -XPOST http://localhost:8080/client?url=aa  -H 'Content-Type: application/json' -d'{"aa": "bbd"}'
```

El cliente intentara llamar a `http://localhost:8081/server/aa` lo cual provocara un error y recibiremos la siguiente salida:

```
http://localhost:8081/server/aa Called. Error 4xx: 404 NOT_FOUND
```

Y esto es todo sobre la clase **WebClient** de momento ;-) . En próximos artículos seguiré hablando de programación reactiva.






