---
title: La Clase RestTemplate - 2 
draft: false
weight: 20
pre: "<b>o </b>"
author: El Profe
url: /clase-resttemplate-2/
type: post
date: 2019-08-06
categories:
  - java
  - rest
  - spring
  - web
tags:  
  - java
  - rest
  - spring
 
---

En [la anterior entrada](/2019/08/03/trabajando-con-la-clase-resttemplate/) vimos como lanzar una  petición HTTP contra un servidor externo, pero al lanzarla surgieron algunas dudas. Dos ya fueron resueltas así que continuemos resolviendo las siguientes.<!--more-->

### 3. Y si el servidor devuelve un OK, pero lo devuelto no es un objeto del tipo mandado,  ¿ qué pasara ?

Como hemos visto cuando se realiza la petición uno de los parámetros  mandados es el objeto que se espera devolver. **Spring** creara un objeto del tipo mandado e intentara cargar los variables que lo definen.

En el proyecto de ejemplo se trabaja con el objeto  `Customer` que solo tiene dos campos `name` y `address`, por lo tanto **Spring**  recogerá el cuerpo de la respuesta en formato JSON (puede trabajar en otros formatos como XML, pero en este articulo nos ceñiremos a JSON) e intentara establecer los valores a esos dos campos.

Para simular este caso en el ejemplo si realizamos esta llamada: 

```
curl -s http://localhost:8080/ACCEPT
```

El servidor lanzara una excepción pero el código HTTP será 202 (**ACCEPTED**) con lo cual la clase **RestTemplate** no dará ningún error, sin embargo el cuerpo devuelto será algo como esto:

```
 {"timestamp":"2019-08-05T11:02:18.314+0000","status":202,"error":"Accepted","message":"Don't send me accepts!!","trace":"com.profesorp.restTemplate.MyAcceptedException: Don't send me accepts!!..... }
```

Como **RestTemplate** no encontrara ningún valor para los campos  `name` o `address`, la salida de nuestro programa será la siguiente:

````
Http Status: 202 ACCEPTED -> Customer(name=null, address=null)
````

Contestando a la pregunta: *No pasara nada* simplemente los campos del objeto `Customer` tendrán el valor NULL, pues no se llamara a las correspondientes funciones *setter*.

### 4. ¿Cómo podría tener un registro de lo enviado y recibido por el servidor ?

La respuesta esta en la función `setInterceptors` de la clase `RestTemplate` con ella podemos definir interceptores que serán ejecutados cuando realicemos peticiones.

En la función `createRestTemplateInterceptor` se define un `RestTemplate` añadiéndole un interceptor.

````
@Bean
@Qualifier("restInterceptor")
public RestTemplate createRestTemplateInterceptor(CustomResponseErrorHandler errorHandler) {
    RestTemplate restTemplate = new RestTemplate(new BufferingClientHttpRequestFactory(new SimpleClientHttpRequestFactory()));				
    List<ClientHttpRequestInterceptor> interceptors = new ArrayList<>();		
    interceptors.add(new LoggingRequestInterceptor());
    restTemplate.setInterceptors(interceptors);
    return restTemplate;
}
````

La clase donde se define el interceptor es la siguiente:

```
public class LoggingRequestInterceptor implements ClientHttpRequestInterceptor {	
	@Override
	public ClientHttpResponse intercept(HttpRequest request, byte[] body, ClientHttpRequestExecution execution)
			throws IOException {
		traceRequest(request, body);
		final ClientHttpResponse response = execution.execute(request, body);	
		traceResponse(response);
		return response;
	}
	private void traceRequest(HttpRequest request, byte[] body) throws IOException {
	
		log.debug("===========================request begin================================================");
		log.debug("URI         : {}", request.getURI());
		log.debug("Method      : {}", request.getMethod());
		log.debug("Headers     : {}", request.getHeaders());
		log.debug("Request body: {}", new String(body, "UTF-8"));
		log.debug("==========================request end================================================");
	}
	private void traceResponse(ClientHttpResponse response) throws IOException {
		log.debug("============================response begin==========================================");
		log.debug("Status code  : {}", response.getStatusCode());
		log.debug("Status text  : {}", response.getStatusText());
		log.debug("Headers      : {}", response.getHeaders());
		StringBuilder body=new StringBuilder();;
		try {					
			BufferedReader bufferedReader = new BufferedReader(new InputStreamReader(response.getBody(), "UTF-8"));
			String line = bufferedReader.readLine();
			while (line != null) {
				body.append(line);
				body.append('\n');
				line = bufferedReader.readLine();
			}	
		} catch (Exception k) {
			k.printStackTrace();
		}
		log.debug("Response body: {}", body==null?"":body.toString());
		log.debug("=======================response end=================================================");
	}
}
```

En esta clase que debe implementar el interface **ClientHttpRequestInterceptor** se debe definir la función `intercept` que será llamada cuando se realice la petición HTTP.

Como se ve en ella primero escribimos logs con los datos de la petición, después se continua el proceso con la sentencia `execution.execute` recogiendo el objeto `ClientHttpResponse` del cual se escriben una serie de datos en el log.

Es importante recalcar que si queremos imprimir el cuerpo del mensaje devuelto por el servidor deberemos utilizar la función `getBody` de la clase `ClientHttpResponse`  la cual devuelve un `InputStream` . Lógicamente si leemos ese *stream* lo consumiremos, con lo cual si en otra clase se intenta leer esos datos ya no estarán disponibles. Por eso si intentamos coger el cuerpo del menaje de la clase `ResponseEntity` veremos que nos devuelve NULL.

Para evitar esto al crear la clase RestTemplate le pasamos al constructor una clase `BufferingClientHttpRequestFactory`. Esa clase permite poder realizar diferentes lecturas del *body*  sin consumirlo, con lo cual ya podremos mostrar el cuerpo del mensaje en el log y luego recogerlo en el `ResponseEntity`.

##### 4.1 Practica

Ejecutamos el siguiente comando

```
curl -s http://localhost:8080/prueba
```

El cual nos produce la siguiente salida:

```
Http Status: 200 OK -> {"name":"Customer prueba","address":"Address Customer prueba"}
```

En el log del cliente obtendremos algo como esto:

``` 
===========================request begin================================================
URI         : http://localhost:8080?queryParam=prueba
Method      : GET
Headers     : [Accept:"text/plain, application/json, application/*+json, */*", Content-Length:"0"]
Request body: 
==========================request end================================================
============================response begin==========================================
Status code  : 200 OK
Status text  : 
Headers      : [Content-Type:"application/json", Transfer-Encoding:"chunked", Date:"Tue,  Response body: {"name":"Customer prueba","address":"Address Customer prueba"}
=======================response end=================================================
```

{{% notice tip %}}
Recordar lanzar la aplicación con el parámetro **-Dlogging.level.com.profesorp=debug**  para mostrar los logs del tipo **DEBUG**.
{{% /notice %}}



### 5. ¿Me devolverá el objeto tipo `Customer` en el cuerpo de la respuesta aunque no sea OK el estado de esta?

Pues depende :smile:

Como hemos visto durante los artículos, Si has establecido un **ErrorHandler** sí te lo devolverá. En caso contrario no, ya que te saltara una excepción tipo `HttpClientErrorException`

Y con esto quedan todas las preguntas respondidas, ¿ verdad ?  :wink:

Pues en próximas entradas hablare de como realizar peticiones que no sean tipo GET,  poder poner cabeceras y como recibir objetos que sean [genéricos](https://docs.oracle.com/javase/tutorial/java/generics/types.html) 



