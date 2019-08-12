---
title: La Clase RestTemplate - 3 
draft: true
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

Para más detalles visita la pagina oficial de **Spring** [https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/client/RestTemplate.html](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/client/RestTemplate.html). donde 

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

La única diferencia con la anterior petición es que deberemos incluir el objeto a enviar. Ese objeto puede ser del tipo  [`HttpEntity`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/http/HttpEntity.html)  para poder incluir cabeceras en la petición.

```
HttpHeaders headers = new HttpHeaders();
headers.setContentType(MediaType.APPLICATION_JSON);
headers.set("my-id", "profe-test");
HttpEntity<Customer> httpEntity = new HttpEntity<>(customer,headers);
ResponseEntity<String> responseEntity = restTemplateInterceptor.postForEntity( localUrl, httpEntity,String.class);
```







```
curl -XPOST http://localhost:8080/client/ -H "content-type: application/json" -d '{"name": "Profe","address":"Spain"}'
```

