---
title: Cacheando resultados con Spring Boot
pre: "<b>o </b>"
author: airec69
type: post
date: 2019-05-12T07:57:07+00:00
url: /2019/05/12/cacheando-datos-en-spring-boot/
categories:
  - cache
  - java
  - spring boot
tags:
  - cache
  - java
  - spring boot

---
Vamos a imaginar una aplicación web, donde por cada petición recibida, debe leer ciertos datos de configuración desde una base de datos. Esos datos no cambiaran normalmente pero nuestra aplicación, en cada petición, debe conectarse, ejecutar las sentencias adecuadas para leer los datos, traerlos por la red, etc. Imaginemos, además, que la base de datos a la que nos conectamos esta saturada o la conexión de red que nos une a la base de datos es inestable. ¿Qué pasaría?. Pues que tendríamos una aplicación lenta por el hecho de leer continuamente unos datos que sabemos que apenas cambian.

Para solucionar ese problema podríamos utilizar una **Cache**, pero ¿ como implementarlo ?. En este articulo explicare como usar una cache básica en **Spring Boot**.

#### Un poco de teoría

La cache se aplica sobre funciones, donde para un mismo valor de entrada esperamos un mismo valor de salida. Es por ello que siempre debemos tener al menos un parámetro de entrada y una salida.

Un ejemplo típico seria este:

```
@Cacheable(cacheNames="headers")
 public int funcionCacheada(int valor)
 {
  ... calculos muy complejos y costosos ....
  return N;
 }
```

Y ahora supongamos que tenemos el siguiente código donde llamamos a esa función:

    int valor=funcionCacheada(1);
    int otroValor=funcionCacheada(2);
    int tercerValor=funcionCacheada(1);
    

Al ejecutar el programa, en la primera línea, **Spring**, ejecutara la función y guardara el resultado que devuelve. En la segunda línea, como no sabe el valor que se debe devolver para la entrada con valor &#8220;2&#8221; hará lo mismo. Sin embargo en la tercera línea **Spring** detectara que una función marcada con **@Cacheable** con el nombre de cache **&#8220;headers&#8221;** ya ha sido llamada con el valor &#8220;1&#8221; y no ejecutara la función, simplemente devolverá el valor que en la primera llamada guardo.

El nombre de la cache es importante pues, entre otras cosas, nos permite tener diferentes caches independientes, las cuales podremos, entre otras cosas limpiar, para obligar a a **Spring Boot** a ejecutar de nuevo las funciones.

Así, la idea básicamente es que en cada llamada a una función marcada como @**Cacheable** se guarda en una tabla interna los resultados para cada llamada, de tal manera que si ya tiene la salida para una entrada, no llama a la función.

#### Practica

Y ahora, vamos a la práctica:

El proyecto de ejemplo sobre el que esta basado este articulo esta en: <https://github.com/chuchip/cacheExample>

Lo primero que se necesita es incluir la siguiente dependencia en nuestro proyecto:

```
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-cache</artifactId>
</dependency>
```

Ahora ya podremos utilizar las etiquetas que nos permitirán usar **Cache** en nuestra aplicación.

La primera etiqueta a poner es **@EnableCaching**. Con esta etiqueta le indicamos a Spring que prepare el soporte para usar cache. Si no se la ponemos simplemente no la usara, independientemente de si indicamos posteriormente que cachee los resultados de unas funciones.

```
@SpringBootApplication
@EnableCaching
public class CacheExampleApplication {
	public static void main(String[] args) {
		SpringApplication.run(CacheExampleApplication.class, args);
	}
}
```

En este ejemplo se leerán unos datos de una base de datos a través de unas peticiones REST.

Los datos como tal se leen en la clase `CacheDataImpl.java` que esta en el paquete `com.profesorp.cacheexample.impl`

La función que lee los datos es la siguiente:
```
@Cacheable(cacheNames="headers", condition="#id > 1")
 public DtoResponse getDataCache(int id) {	
		try {
			Thread.sleep(500);
		} catch (InterruptedException e) {
		}		
		DtoResponse requestResponse=new DtoResponse();		
		Optional<Invoiceheader> invoice=invoiceHeaderRepository.findById(id);
		..... MAS CODIGO NO IMPORTANTE ...
	}
```

Como se puede ver tenemos la etiqueta **@Cacheable(cacheNames=&#8221;headers&#8221;, condition=&#8221;#id > 1&#8243;)**

Con ella, le estamos indicando a Spring dos cosas.

- Que deseamos que cachee el resultado de esta función.
- Le ponemos como condición, que solo cachee los resultados si el valor de entrada es superior a 1.

Más adelante, en la función `flushCache`, le ponemos la etiqueta @**CacheEvict** la cual limpia la cache indicada. En este caso, además, le indicamos que borre todas las entradas que tengan cacheadas.

    @CacheEvict(cacheNames="headers", allEntries=true)
    public void flushCache() {	}	
    

En la función `update` actualizamos la base de datos y con la etiqueta **@CachePut** le indicamos a Spring que actualice los datos para el valor que hay en **dtoRequest.id**

    @CachePut(cacheNames="headers", key="#dtoRequest.id")
    public  DtoResponse update(DtoRequest dtoRequest)
    {
    .... ACTUALIZADA LA BASE DE DATOS ...
    }
    

Por supuesto esta función tiene que devolver un objeto del mismo tipo que el de la marcada con la etiqueta **@Cacheable** y debemos indicarle el valor de entrada, para el que se desea actualizar los datos.

#### Funcionando

Para entender mejor la aplicación vamos a arrancarla y realizarle algunas peticiones.

La aplicación al arrancar guarda en la tabla `invoiceHeader` las cabeceras de 4 facturas (podéis ver como lo hace en el fichero `data.sql`).

Vamos a ejecutar la función `get` de la clase `PrincipalController`, para ello escribimos:

    > curl -s http://localhost:8080/2
    

Nos devolverá lo siguiente:

    {"interval":507,"httpStatus":"OK","invoiceHeader":{"id":2,"activo":"N","yearFiscal":2019,"numberInvoice":2,"customerId":2}}
    

El campo `interval` el tiempo en milisegundos que le ha costado realizar la consulta. Como se puede ver le ha costado mas de medio segundo, pues en la función `getDataCache` de `CacheDataImpl.java`tenemos un sleep de 500 milisegundos.

Ahora ejecutamos de nuevo la llamada:

```
curl -s http://localhost:8080/2
{"interval":1,"httpStatus":"OK","invoiceHeader":{"id":2,"activo":"N","yearFiscal":2019,"numberInvoice":2,"customerId":2}}
```

Ahora el tiempo que ha tomado la llamada es _1_ , porque realmente **Spring** **NO** ha ejecutado el código de la función y simplemente ha devuelto el valor que tenia cacheado.

Sin embargo si solicitamos el **id** 1 como hemos indicado que no lo cachee siempre ejecutara la función y por lo tanto tendremos un tiempo superior a 500 milisegundos:

    >curl -s http://localhost:8080/1
    {"interval":503,"httpStatus":"OK","invoiceHeader":{"id":1,"activo":"S","yearFiscal":2019,"numberInvoice":1,"customerId":1}}
    >curl -s http://localhost:8080/1
    {"interval":502,"httpStatus":"OK","invoiceHeader":{"id":1,"activo":"S","yearFiscal":2019,"numberInvoice":1,"customerId":1}}
    >curl -s http://localhost:8080/1
    {"interval":503,"httpStatus":"OK","invoiceHeader":{"id":1,"activo":"S","yearFiscal":2019,"numberInvoice":1,"customerId":1}}
    

Si llamamos a la función `flushcache` limpiaremos la cache y por lo tanto la próxima llamada a la función cacheada deberá ejecutar la función:

    > curl -s http://localhost:8080/flushcache
    Cache Flushed!
    > curl -s http://localhost:8080/2
    {"interval":508,"httpStatus":"OK","invoiceHeader":{"id":2,"activo":"N","yearFiscal":2019,"numberInvoice":2,"customerId":2}}
    > curl -s http://localhost:8080/2
    {"interval":0,"httpStatus":"OK","invoiceHeader":{"id":2,"activo":"N","yearFiscal":2019,"numberInvoice":2,"customerId":2}}
    

Por ultimo veremos como si cambiamos el valor del campo `activo` a `N` , como la función que realiza el cambio esta marcada con `@CacheEvict` nos actualizara el valor de la cache, pero en la próxima llamada a la función `getDataCache` se seguirá sin ejecutar el código, devolviendo, sin embargo, el objeto actualizado.

<pre><code class="language-curl" lang="curl">>  curl -X PUT   http://localhost:8080/   -H "Content-Type: application/json"   -d "{\"id\": 2, \"active\": \"N\"}"
>curl -s http://localhost:8080/2
{"interval":0,"httpStatus":"OK","invoiceHeader":{"id":2,"activo":"N","yearFiscal":2019,"numberInvoice":2,"customerId":2}}
</code></pre>

### Conclusiones

**Spring Boot**, sin ninguna dificultad, nos permite cachear los resultados de las funciones, sin embargo hay que tener en cuenta que esa cache es muy básica y la realiza en memoria. Sin embargo **Spring Boot** permite usar librerías externas que nos permitirán cachear en disco, en bases de datos, etc.

En <https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-caching.html> tenéis las diferentes implementaciones de Cache que **Spring Boot** soporta, entre las que esta [EhCache][1] , con la cual podréis definir diferentes tipos de _backend_ para los datos, así como especificar tiempos de validez para los datos y muchas más opciones.

Como veis, todo un mundo para explorar.

¡¡Nos vemos en la proxima entrada!!



 [1]: https://www.ehcache.org/