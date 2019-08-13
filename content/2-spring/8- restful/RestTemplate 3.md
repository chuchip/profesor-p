---
title: La Clase RestTemplate - 3 
draft: false
weight: 30
pre: "<b>o </b>"
author: El Profe
url: /clase-resttemplate-3/
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

Esta es la tercera y última  entrada sobre la clase **RestTemplate**. Puedes ver las anteriores entradas en [Trabajando con la clase RestTemplate](/2019/08/03/trabajando-con-la-clase-resttemplate/) y en
[La clase RestTemplate-2](/clase-resttemplate-2/).

En esta ocasión hablare sobre las funciones más utilizadas en esta clase y como usarlas.

Para más detalles visita la pagina oficial de **Spring** [https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/client/RestTemplate.html](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/client/RestTemplate.html). 

Antes de enumerar como realizar las diferentes tipos de peticiones insistiré en que c siempre podremos recibir directamente el cuerpo el mensaje  o un **ResponseEntity**. Como comente anteriormente siempre es mejor recibir el **ResponseEntity** pues de otra manera no tendremos información sobre el resultado de la petición.  Para recibir el cuerpo podremos usar las funciones tipo **getForObject**,  **postForObject**, etc.

### 1. Peticiones GET

Este tipo de petición es la más sencilla y en las anteriores entradas ya se han realizado varias.

Añadir que también disponemos de la función

```
public <T> ResponseEntity<T> getForEntity(String url,
                                          Class<T> responseType,
                                          Map<String,?> uriVariables)
                                   throws RestClientException
```

En ella, podremos pasar un objeto `Map<String,?>` para hacer sustituciones sobre la URL.

Es decir si la URL mandada, como en el proyecto de ejemplo es: `http://8080/server?queryParam={queryParam}` mandaremos un `Map`  con el valor deseado para la clave `queryParam`.

```
Map<String,String> sustitute=new HashMap<>();
sustitute.put("queryParam",path);
```

### 2. Peticiones POST

La única diferencia con la anterior petición es que deberemos incluir el objeto a enviar. Ese objeto puede ser del tipo  [`HttpEntity`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/http/HttpEntity.html)  si necesitamos incluir cabeceras en la petición.

```
HttpHeaders headers = new HttpHeaders();
headers.setContentType(MediaType.APPLICATION_JSON);
headers.set("my-id", "profe-test");
HttpEntity<Customer> httpEntity = new HttpEntity<>(customer,headers);
ResponseEntity<String> responseEntity = restTemplateInterceptor.postForEntity( localUrl, httpEntity,String.class);
```
En el proyecto de ejemplo si ejecutamos lo siguiente.
```
> curl -XPOST http://localhost:8080/client/ -H "content-type: application/json" -d '{"name": "Profe","address":"Spain"}'
Http Status: 200 OK -> {"name":"Profe","address":"Spain"}
```

En los logs del programa veremos:

```
Client - Received custom request type POST. Customer: Customer(name=Profe, address=Spain)
===========================request begin================================================
URI         : http://localhost:8080
Method      : POST
Headers     : [Accept:"text/plain, application/json, application/*+json, */*", Content-Type:"application/json", my-id:"profe-test", Content-Length:"34"]
Request body: {"name":"Profe","address":"Spain"}
==========================request end================================================
============================response begin==========================================
Status code  : 200 OK
Status text  : 
Headers      : [Content-Type:"application/json", Transfer-Encoding:"chunked", Date:"Tue, Response body: {"name":"Profe","address":"Spain"}
=======================response end=================================================

```



### 3. Otras peticiones.

La clase RestTemplate tiene funciones para realizar cualquier tipo de petición. Así si deseamos realizar una petición tipo **put** podremos utilizar.

```
public void put(URI url, @Nullable  Object request) throws RestClientException
```

Si la petición a realizar tiene que ser tipo **delete** podremos utilizar 

```
public void delete(String url) throws RestClientException
```

y así con todos los tipos de peticiones como **patch**, **head**, etc.


### 4. Peticiones avanzadas con Exchange

Sin embargo hay algunos casos donde no tendremos la combinación necesaria para ejecutar una petición. Por ejemplo, si deseamos realizar una petición tipo **get** incluyendo cabeceras en la petición no podremos usar la función **getForEntity**. Para esos casos disponemos de la función **exchange**. 

```
public <T> ResponseEntity<T> exchange(RequestEntity<?> requestEntity,
                                      Class<T> responseType)
                               throws RestClientException
```

Esta función siempre recibe un objeto **HttpEntity** donde podremos especificar el tipo de petición (**POST**,**GET**, etc), incluir cabeceras y por supuesto el objeto a mandar.

Hay diversas variantes de esta función para cubrir diferentes situaciones.

### 5. Recibir contenedores de  objetos genéricos.

Si el servidor debe devolver un contenedor de objetos genérico, como puede  ser un **List\<T>**  o un **Map <T,R>** no podremos especificar el objeto a devolver como hemos visto hasta ahora.

Supongamos que el servidor devuelve una lista de **Strings**. En java no se puede poner **List\<String>.class** pues nos dará un error de sintaxis. Tampoco deberíamos escribir nuestro código así:

```
ResponseEntity<List<String>> myObject=restTemplate.getForEntity(url,List.class);
```

pues se nos quejara de que hemos especificado que vamos a recibir un objeto **List** y no un objeto **List\<String>**

Por supuesto podríamos poner lo siguiente:

```
ResponseEntity<List> myObject=restTemplate.getForEntity(url,List.class);
```

pero nos saldrá un aviso de que en **List** deberíamos especificar el tipo a recibir y nos puede dar problemas cuando **Spring** intente  convertir el objeto **JSON** recibido.

Para evitar este problema se debe utilizar la siguiente llamada.

```
public <T> ResponseEntity<T> exchange(RequestEntity<?> requestEntity,
                                      ParameterizedTypeReference<T> responseType)
                               throws RestClientException
```

En la clase **ParameterizedTypeReference** especificaremos el objeto  a devolver.

En la propia documentación de **Spring** hay un excelente ejemplo que clarifica la llamada.

```
MyRequest body = ...
RequestEntity request = RequestEntity
     .post(new URI("https://example.com/foo"))
     .accept(MediaType.APPLICATION_JSON)
     .body(body);
ParameterizedTypeReference<List<MyResponse>> myBean =
     new ParameterizedTypeReference<List<MyResponse>>() {};
ResponseEntity<List<MyResponse>> response = template.exchange(request, myBean);
```

Como se ve, el truco es crear un objeto que extienda de la clase **ParameterizedTypeReference** (la cual es abstracta) y especificar el tipo del objeto a devolver. 



Y con esto doy por terminada la serie de artículos sobre **RestTemplate**.

Recomiendo mirar todas las funciones de esta clase en la [documentación oficial](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/client/RestTemplate.html) de **Spring** donde estan todas las funciones de las que he hablado y algunas más que si bien son útiles no he tratado en esta serie, como puede ser la función `public void setMessageConverters(List<HttpMessageConverter<?>> messageConverters) `para especificar los conversores a usar.

Un saludo y hasta la próxima entrada.