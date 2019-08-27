In this article we will speak about the [WebClient](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/web/reactive/function/client/WebClient.html) class of the **SpringBoot** framework.

You have the source about what I write in the post at https://github.com/chuchip/webClientExample](https://github.com/chuchip/webClientExample)

This class would be the equivalent of RestTemplate class, but it work with asynchronous requests.

If you want to use this class, you should put this dependencies in your maven file.

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-webflux</artifactId>
</dependency>
```

This is so why you need to use **WebFlux** which is available in the version 5 of Spring. Remember that this version of Spring requires **Java 8** or higher.

With [WebFlux](https://docs.spring.io/spring-framework/docs/5.0.0.BUILD-SNAPSHOT/spring-framework-reference/html/web-reactive.html) which is based on the project [Reactor](https://projectreactor.io/) you can write reactive applications. This kind of applications is characterized because the requests aren't blocking and because the [functional programming](https://en.wikipedia.org/wiki/Functional_programming) is widely used.

If you want to understand this article you should have certain notions about Reactor and the **Mono** class.   Although if you have used **Streams** in **Java**  you may think that a **Mono** object is like a **Stream** that can emit a value or an error.

But i am not going to delve into these issues  that are beyond the scope of this article. It will be sufficient to say that using the **WebClient** class you can make several calls in parallel, so if each request is answered in 2 seconds  and you make 5 calls, you can get all the answers in just over 2 seconds, instead of 10.

#### Parallel Request 

In the example project I have written an server and a client. The server will be running in the 8081 port and the client will listen in the 8080 port.

This is the code that execute the server application.

```
@SpringBootApplication
public class WebServerApplication {
	public static void main(String[] args) {
		new SpringApplicationBuilder(WebServerApplication.class).
				properties(Collections.singletonMap("server.port", "8081")).run(args);
	}
}
```

if you make a request to http://localhost:8080/client/XXX the function `testGet` will be executed.

```java
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

This is a simple controller that performs  two requests  to the URL  http://localhost:8081 . In the first request the parameter passed is that the function received in the **param** variable.  In the second the sentence "**STOP**" is sent if the **param** variable  is different of the word **SPEED**.

The server upon receiving the parameter "**STOP**" will wait 5 seconds and then will answer.

At the moment of create the instance of the class **WebClient**  I specify the URL of the request.

```java
WebClient webClient = WebClient.create(urlServer+"/server/");
```

Then it execute the call of the type **GET** to the server passing the parameter **QueryParam**. At the end, executing the function **exchange** it will receive a **Mono** object containing a **ClientResponse** class that  is equivalent to the **ResponseEntity** object of the **RestTemplate** class. The class **ClientResponse** will have the HTTP code, the body and the headers sent by the server.

```java
Mono<ClientResponse> respuesta = webClient.get().uri("?queryParam={name}", param).exchange();
```

Wait a minute .. , did I say it will execute ?. Well I lied. Really only what I want to do has been declared.  In reactive programming until someone does not subscribe to a request nothing is executed, so the request to the server has not yet been made.

On the next line the second request to the server is defined.

```java
Mono<ClientResponse> respuesta1 = webClient.get().uri("?queryParam={name}","SPEED".equals(param)?"SPEED":"STOP").exchange();
```

Finally, a  **Mono** object is created, which will be the result of the previous two, using the `zip` function.

Using the `map` function a **ResponseEntity** object with the HTTP code equal to **OK** will be returned if the two requests have answered a 2XX code, otherwise the code and answer of the server will be returned.

Being **WebClient**  reactive the two requests are realized simultaneously and therefore you will be able see that if you execute this code: `curl http://localhost:8080/client/STOP` you have the answer in just over 5 seconds, even if the sum of the call time is greater than 10 seconds.

```
 All OK. Seconds elapsed: 5.092
```

#### A POST request

In the testURLs function there is an example of a call using POST.

This function receives a `Map` in the body  that will then be placed  in the request headers. In addition this `map` will be sent in the body of the request that will be made to the server.

```java
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

To insert the body of the message the auxiliary class `BodyInserters` will be used. If the message were on an object **Mono** this code could be used:

```java
BodyInserters.fromPublisher(Mono.just(MONO_OBJECT),String.class);
```

When performing a `flatMap`, the output of the `ClientResponse` object will be captured and an **Mono** object will be returned with the string to be returned. . The `flatMap` function  will flatten  that **Mono**  object and will extract  the string inside it and that is why a `Mono <String>` will be received and not a `Mono <Mono <String>>` as It would happen if we used the function `map`.

Making the following call:

```bash
> curl  -s -XPOST http://localhost:8080/client  -H 'Content-Type: application/json' -d'{"aa": "bbd"}'
```

The following output will be obtained:

```bash
the server said: {aa=bbd}
Headers: content-length:12
Headers: aa:bbd
Headers: accept-encoding:gzip
Headers: Content-Type:application/json
Headers: accept:*/*
Headers: user-agent:ReactorNetty/0.9.0.M3
Headers: host:localhost:8081
```

This output is produced by the function `postExamle` of the server

```java
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

Note that when using the **WebFlux** library that is not fully compatible with **javax.servlet** we must receive a `ServerHttpRequest` object to collect all raw headers. The equivalent in a non-reactive application would be an `HttpServletRequest` object.

If you execute the sentence:

```bash
> curl  -s -XPOST http://localhost:8080/client?url=aa  -H 'Content-Type: application/json' -d'{"aa": "bbd"}'
```

The client  will try to call http://localhost:8081/server/aa  which will cause an error and the following will be received.

```bash
http://localhost:8081/server/aa Called. Error 4xx: 404 NOT_FOUND
```

The original article  was written in Spanish and you can read it at http://profesor-p.com/webclient