---
title: Aplicación en Spring REST y Angular – 3ª Parte
author: El Profe
type: post
date: 2018-09-04T05:44:12+00:00
url: /2018/09/04/aplicacion-en-spring-rest-y-angular-3a-parte/
categories:
  - java
  - rest
  - spring
  - tomcat
tags:
  - java
  - rest
  - spring
  - tomcat

---
En esta entrada, continuare con la parte servidor, que ya comencé en [el articulo anterior.][1]

Voy a desarrollar la parte donde se responden a las peticiones REST. Para el que no sepa que es eso de REST, podéis empezar leyendo <a href="https://es.wikipedia.org/wiki/Transferencia_de_Estado_Representacional" target="_blank" rel="noopener">este articulo de la wikipedia</a>, pero os podeis quedar con la idea de que es como una petición web normal, solo que en vez de trabajar con paginas HTML enteras, se trabaja con intercambio de datos más o menos en crudo.

En este ejemplo, el codigo que trata esas peticiones esta en la clase: **YagesController**

<pre>@RestController
@RequestMapping("/rest")
public class YagesController {

	@Autowired
	private YagesBussines yagesBussines;

	@RequestMapping(value = "/{anoId}", method=RequestMethod.GET,
            produces="application/json")
	public VentasAnoBean getAno(@PathVariable int anoId) {
		System.out.println("getAno.Año: "+anoId);
		return yagesBussines.getVentasAno(anoId);
	}
	
	@RequestMapping(value ="/{anoId}/{mesId}", method=RequestMethod.GET,produces="application/json")
	public ArrayList&lt;VentasSemanaBean&gt; getSemanas(@PathVariable int anoId,@PathVariable int mesId)	
	{
		 System.out.println("GetSemanas. Mes "+mesId+" Año: "+anoId);
		 return yagesBussines.getDatosSemana(mesId, anoId);
	}
}</pre>

Con solo este código vamos a ser capaces de responder a las peticiones.

Lo primero es anotar la clase como del tipo @RestContoller. Después especificamos la ruta que este controlador debe responder. Eso lo hago con la anotación @RequestMapping(&#8220;/rest&#8221;). Eso hará que todas las peticiones que vayan a la URL  _MIDOMINIO/MICONTEXTO/rest_ sean tratadas por esta clase.

Como dije anteriormente un servidor REST, atiende peticiones HTTP, como son las que se utilizan habitualmente en un navegador web, y como cualquier petición web, siempre debe haber una URL (o dirección) de la página que debemos visitar. Así, cuando visitamos Google, en nuestro navegador ponemos la dirección: http://www.google.com. Pues bien, si nuestro servicio REST responde, por ejemplo en la pagina www.profesor-p.com, para interactuar con él deberíamos hacer una petición web (es decir visitar la pagina web) : http://www.profesor-p.com/rest/LO\_QUE\_SEA

¿ Y que es entonces eso de MI_CONTEXTO ? . Bueno, es muy común que en un servidor de aplicaciones (Tomcat, por ejemplo) haya varias aplicaciones corriendo al mismo tiempo, sobre todo, para optimizar recursos. De esta manera si tenemos un servidor de Tomcat corriendo en la dirección **www.servidor-profesorp.com**, cuando queramos acceder a la aplicación de contabilidad pondremos: **www.servidor-profesorp.com/contabilidad** y si queremos acceder a la aplicación de nominas iremos a www.servidor-profesorp.com/**nominas** y así con las diferentes aplicaciones. Es decir todas las peticiones de la aplicación web de **contabilidad** responderán bajo el contexto de **www.servidor-profesorp.com/contabilidad** y todas las peticiones de la aplicación **nominas** responderán bajo el contexto de **www.servidor-profesorp.com/****nominas.** A todos los efectos, la idea es como cuando se ejecuta una aplicación  de escritorio en Windows, como puede ser Word, Excel o el Bloc de Notas, cada aplicación del escritorio tiene su propia ventana, pues en una aplicación web, cada  una tiene su contexto o directorio donde responde a las peticiones.

Aclarar, que hay veces, que el  contexto puede no existir en el caso de que solo tengamos una aplicación corriendo en esta dirección web o bien, porque es la dirección principal y no haga falta ponerla. En esos casos las peticiones REST serán respondidas en MIDOMINIO/rest.

Para el resto del articulo vamos a suponer que nuestra aplicación esta respondiendo en la URL **http://localhost:8080/yagesserver** que es donde normalmente se realizan las pruebas. Localhost, como todo el mundo sabe, es la dirección interna  del servidor (que sera probablemente tu ordenador personal en este ejemplo) donde se esta corriendo el programa. Y **yaggesserver** es el contexto donde responderá la aplicación.

Y como un ejemplo, vale más que mil palabras, aquí vemos un ejemplo de la una petición REST, haciendo uso de la función **getAno()** de la clase **YagesController.**

<img class="size-full wp-image-169 aligncenter" src="http://www.profesor-p.com/wp-content/uploads/2018/09/yages2.png" alt="" width="715" height="545" srcset="http://www.profesor-p.com/wp-content/uploads/2018/09/yages2.png 715w, http://www.profesor-p.com/wp-content/uploads/2018/09/yages2-300x229.png 300w" sizes="(max-width: 715px) 100vw, 715px" />

&nbsp;

Y como esta entrada me esta quedando un poco larga, continuare con ella en un próximo articulo.

¡¡ A seguir estudiando !!

 [1]: http://www.profesor-p.com/2018/09/03/aplicacion-en-spring-rest-y-angular-2-parte/