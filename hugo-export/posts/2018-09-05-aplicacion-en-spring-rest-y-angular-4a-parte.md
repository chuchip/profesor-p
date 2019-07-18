---
title: Aplicación en Spring REST y Angular – 4ª Parte
author: El Profe
type: post
date: 2018-09-05T10:16:08+00:00
url: /2018/09/05/aplicacion-en-spring-rest-y-angular-4a-parte/
categories:
  - java
  - json
  - lambda
  - rest
  - spring
  - tomcat
tags:
  - java
  - json
  - rest
  - spring
  - tomcat

---
Continuo con la serie de artículos explicando una aplicación donde la parte de servidor esta creada con Java, apoyándose en el framework Spring y la parte del cliente usara Angular. Para la comunicación entre la aplicación usare peticiones REST, por supuesto utilizando el protocolo JSON.

En <a href="http://www.profesor-p.com/2018/09/04/aplicacion-en-spring-rest-y-angular-3a-parte/" target="_blank" rel="noopener">la anterior entrada</a> empece a explicar como se desplegaría la aplicación y en que URLs se procesarían las diferentes peticiones. Ahora explicare como funcionan las diferentes peticiones.

En la clase **YagesController  **es donde se hace toda la magia 😉

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

Como ya comente, lo primero es la anotación @RestController, que le dice a Spring que nuestra clase va a procesar peticiones tipo REST. A continuación, la anotación  @RequestMapping(&#8220;/rest&#8221;) especifica que debe responder a las peticiones que vayan a la URL **/rest **

Como dije en la entrada anterior, por claridad en los ejemplos vamos a suponer que nuestra aplicación responde en la dirección (URL) : http://localhost:8080/yagesserver . Aclarar, que el 8080 es el puerto donde escuchara y **yagesserver** es el directorio o contexto base donde estará nuestra aplicación.

La clase  define un único objeto tipo **YagesBussines  **a través del sistema de inyección de Spring y tiene dos funciones, que explico a continuación:

  * #### **Función: getAno**

<pre>@RequestMapping(value = "/{anoId}", method=RequestMethod.GET,
            produces="application/json")
	public VentasAnoBean getAno(@PathVariable int anoId) {
		System.out.println("getAno.Año: "+anoId);
		return yagesBussines.getVentasAno(anoId);
	}</pre>

Como se ve, la función esta anotada con la directiva **@RequestMapping**. Con esta directiva vamos a especificar que dirección (dentro de /rest/ )  vamos a tratar en esta función. Así, en este caso le decimos que vamos a procesar aquellas peticiones que tengan un solo parámetro y, que ademas,  la petición sea del tipo _GET_. También especificamos que la respuesta a la petición sera del tipo _application/json._

Es decir, el esta función sera llamada ( y su código ejecutado ) cuando hagamos una petición del tipo **http://localhost:8080/yagesserver/rest/2018** . Aclarar que no sera llamada si el numero de parámetros no es exactamente uno. Por ejemplo esta petición **http://localhost:8080/yagesserver/rest/2018/2** no sera tratada ya que recibe dos parámetros  (2018 y 2).

Cuando definimos la función, se añade la anotación **@PathVariable** que indica que la variable puesta a continuación deberá contener el parámetro de la URL. Así en la petición **http://localhost:8080/yagesserver/rest/2018  **nuestra variable **anoId** tendrá el valor 2018. Lógicamente, esto obliga a que el parámetro sea del tipo entero, pues hemos definido que nuestra variable anoId es del tipo **int.**

Esta función, devuelve un objeto _VentasAnoBean_ que esta definida en la siguiente clase:

<pre>@Data
public class VentasAnoBean {
   private ArrayList&lt;VentasMesBean&gt; ventasMes = new ArrayList();
   private double kilosVentaAct, impVentaAct, kilosVentaAnt, impVentaAnt,gananciaAct,gananciaAnt;
....
}</pre>

Como se ve, la clase es una simple clase POJO, con unas variables, accesibles a través de sus correspondientes métodos getter, que la librería Lombok, a través de la anotación @Data creara. Teneis mas detalles de la librería  Lombok en [otra entrada de mi pagina][1].

Al haber  especificado que debe devolver un objeto con la codificación _json,  _Spring, apoyándose en las librerías  de **jackson, **devolverá la correspondiente cadena de texto, formateada adecuadamente. No voy a hablar de como  se codifica o descodifica una cadena JSON  en esta entrada y realmente tampoco nos importa. De eso ya se encargaran las correspondientes librerías. Lo que importa es que esta función un objeto del tipo _VentasAnoBean_ en un formato que luego una aplicación cliente va a saber reconstruir.

<img class="size-full wp-image-202 alignleft" src="http://www.profesor-p.com/wp-content/uploads/2018/09/yages2-1.png" alt="" width="734" height="749" srcset="http://www.profesor-p.com/wp-content/uploads/2018/09/yages2-1.png 734w, http://www.profesor-p.com/wp-content/uploads/2018/09/yages2-1-294x300.png 294w" sizes="(max-width: 734px) 100vw, 734px" />

&nbsp;

Aquí se ve una captura de pantalla de como accediendo a la URL:

**http://localhost:8080/yagesserver/rest/2018 **es devuelto un objeto VentasAnoBean, que incluye un array de objetos tipo _ventasMes, _y las variables _kilosVentaAct, impVentaAct, kilosVentaAnt, impVentaAnt,gananciaAct y gananciaAnt._

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

Si usamos el comando **curl**, para hacer la petición HTTP, veremos que la salida, en crudo,  seria la siguiente:

<span style="font-size: 10pt; font-family: georgia, palatino, serif;"><img class="size-large wp-image-204 aligncenter" src="http://www.profesor-p.com/wp-content/uploads/2018/09/curl1-1024x215.png" alt="" width="1024" height="215" srcset="http://www.profesor-p.com/wp-content/uploads/2018/09/curl1-1024x215.png 1024w, http://www.profesor-p.com/wp-content/uploads/2018/09/curl1-300x63.png 300w, http://www.profesor-p.com/wp-content/uploads/2018/09/curl1-768x162.png 768w, http://www.profesor-p.com/wp-content/uploads/2018/09/curl1.png 1131w" sizes="(max-width: 1024px) 100vw, 1024px" /></span>

  * #### **Función: getSemanas**

<pre>@RequestMapping(value ="/{anoId}/{mesId}",
           method=RequestMethod.GET, 
           produces="application/json")
	public ArrayList&lt;VentasSemanaBean&gt; getSemanas(@PathVariable int anoId,@PathVariable int mesId)	
	{
		 System.out.println("GetSemanas. Mes "+mesId+" Año: "+anoId);
		 return yagesBussines.getDatosSemana(mesId, anoId);
	}</pre>

En esta función se van a tratar todas aquellas peticiones como la siguiente **localhost:8080/yagessever/rest/2018/2 **es decir, aquellas que tengan dos parametros.

Esto se especifica por la anotación @RequestMapping donde especificamos que el valor debe ser _&#8220;/{anoId}/{mesId}&#8221;, _despues, en la declaración de nuestra función, especificamos en que variables deben ser introducidos los parámetros de la URL. Así, el primer parámetro sera introducido en la variable _anoId_ y el segundo, en la variable _mesId._

Esta función devuelve un Array de objetos tipo VentasSemanaBean, que no es sino un simple POJO definido en la siguiente clase:

<pre>@Data
public class VentasMesBean {
    private int ano, mes;
    private double kilosVentaAct, impVentaAct, kilosVentaAnt, impVentaAnt;
    private double gananAct, gananAnt;
....
}</pre>

Si realizamos una petición, usando curl, obtendremos la siguiente salida:

<img class="alignright size-large wp-image-205" src="http://www.profesor-p.com/wp-content/uploads/2018/09/curl2-1024x152.png" alt="" width="1024" height="152" srcset="http://www.profesor-p.com/wp-content/uploads/2018/09/curl2-1024x152.png 1024w, http://www.profesor-p.com/wp-content/uploads/2018/09/curl2-300x45.png 300w, http://www.profesor-p.com/wp-content/uploads/2018/09/curl2-768x114.png 768w, http://www.profesor-p.com/wp-content/uploads/2018/09/curl2.png 1145w" sizes="(max-width: 1024px) 100vw, 1024px" />

En la ultima entrada  de la parte servidor, explicare como se consiguen los objetos  _VentasMesBean_ y _VentasAnoBean._

¡¡ Hasta pronto !!

&nbsp;

 [1]: http://www.profesor-p.com/2018/08/24/jpa-con-lombok/