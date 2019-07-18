---
title: Aplicación REST en JavaEE
author: El Profe
type: page
date: 2018-10-06T21:13:33+00:00

---
Buenas, chicos, en esta ocasión voy a explicar como hacer una aplicación CRUD, que sirva peticiones REST con el protocolo JSON, usando JavaEE y Glasfish como servidor de aplicaciones. En esta aplicación actualizaremos y consultaremos una pequeña tabla a través de diferentes peticiones REST.

Si bien es cierto que JavaEE no soporta oficialmente JSON en sus especificaciones, la realidad es que es muy fácil el realizar una aplicación totalmente funcional y muy fácilmente con la libreria JACKSON.

Lo primero, aclarar que en  <a href="https://github.com/chuchip/crudJavaEE" target="_blank" rel="noopener">https://github.com/chuchip/crudJavaEE</a> teneis código fuente del que hablo en esta entrada. Y ahora, pongámonos manos a la obra.

Como ya he dicho vamos a utilizar la libreria Jackson que viene incluida con JavaEE 8.0, por lo cual no necesitaremos incluir ninguna librería externa excepto las necesarias para acceder a la base de datos, a través de JPA. En este caso utilizaremos EclipseLink, por lo que deberemos incluir en nuestro fichero pom.xml las siguientes dependencias:

<pre>&lt;dependency&gt;
		&lt;groupId&gt;org.eclipse.persistence&lt;/groupId&gt;
		&lt;artifactId&gt;org.eclipse.persistence.core&lt;/artifactId&gt;
		&lt;version&gt;2.7.3&lt;/version&gt;
		&lt;scope&gt;provided&lt;/scope&gt;
  &lt;/dependency&gt;     
  &lt;dependency&gt;
		&lt;groupId&gt;org.eclipse.persistence&lt;/groupId&gt;
		&lt;artifactId&gt;org.eclipse.persistence.jpa.jpql&lt;/artifactId&gt;
		&lt;version&gt;2.7.3&lt;/version&gt;
		&lt;scope&gt;provided&lt;/scope&gt;
	&lt;/dependency&gt;
	&lt;dependency&gt;
		&lt;groupId&gt;org.eclipse.persistence&lt;/groupId&gt;
		&lt;artifactId&gt;javax.persistence&lt;/artifactId&gt;
		&lt;version&gt;2.2.1&lt;/version&gt;
		&lt;scope&gt;provided&lt;/scope&gt;
	&lt;/dependency&gt;
	&lt;dependency&gt;
		&lt;groupId&gt;org.eclipse.persistence&lt;/groupId&gt;
		&lt;artifactId&gt;org.eclipse.persistence.jpa&lt;/artifactId&gt;
		&lt;version&gt;2.7.3&lt;/version&gt;
		&lt;scope&gt;provided&lt;/scope&gt;
	&lt;/dependency&gt;

</pre>

También incluiremos las dependencias para las libreria LOMBOK y WEB.

<pre>&lt;dependency&gt;
	&lt;groupId&gt;javax&lt;/groupId&gt;
	&lt;artifactId&gt;javaee-web-api&lt;/artifactId&gt;
	&lt;version&gt;8.0&lt;/version&gt;
	&lt;scope&gt;provided&lt;/scope&gt;
&lt;/dependency&gt;
 &lt;dependency&gt;
	&lt;groupId&gt;org.projectlombok&lt;/groupId&gt;
	&lt;artifactId&gt;lombok&lt;/artifactId&gt;
	&lt;version&gt;1.16.22&lt;/version&gt;
	&lt;scope&gt;provided&lt;/scope&gt;
&lt;/dependency&gt;</pre>

Vale, ya tenemos nuestras librerias. Ahora empecemos con el programa.Primero  voy a crear la clase donde definimos el punto de entrada de nuestras peticiones REST.  Esto lo hacemos en la clase **RestConfiguration.java**

<pre>package profesorp.restexample.resource;
import javax.ws.rs.ApplicationPath;
import javax.ws.rs.core.Application;

@ApplicationPath("api")
public class RestConfiguration  extends Application {
    
}</pre>

Como podéis ver es una clase supersencilla. Tan solo tenemos que crear una clase que herede de Application y añadir la etiqueta @ApplicationPath. Con esto ya especificaremos que todas las peticiones que vayan a la trayectoria (Path para los ingleses) **api** seran tratadas por el servlet de la librería JACKSON.

Ahora vamos a especificar los diferentes recursos dentro de esta trayectoria o camino. Esto lo haremos en la clase **LocaleResource**, la cual detallo a continuación:

<pre>@Path("locale")
@Stateless
public class LocaleResource {
    
    @Inject   
    private  LocaleController localeController;
        
    private  Logger logger=  Logger.getLogger(LocaleResource.class.getName());
<strong>    ..... FUNCIONES ....</strong>
  
}</pre>

Y como decía, Jack El Destripador, vayamos por partes 😉

Lo primero es observar la etiqueta **@Path(&#8220;locale&#8221;)** que indicara la trayectoria a la que responderá esta clase. En este caso indicamos la trayectoria `locale`. Es decir, esta clase sera usada cuando vayamos a la URL: &#8220;_http://localhost:8080/restExample/api/locale_&#8221; . Aclarar que _restExample_ es donde desplegaremos la aplicación en el servidor Glassfish .

La etiqueta @**Stateless** es usada para poder inyectar las dependencias correctamente (vosotros, de momento, hacerme caso y creer que es necesario. ¡¡ Un acto de fe!! 😉

Dentro de la clase, con la etiqueta **@Inject** inyectaremos la clase  **LocaleController**, la cual utilizaremos para interactuar con la base de datos.

Ahora vamos a explicar, una a una las diferentes funciones de  esta clase.

  *  <span style="font-size: 14pt;">Funcion </span>**findAllLocales**

<pre>@GET    
 @Produces(MediaType.APPLICATION_JSON)
  public Response findAllLocales() {        
        logger.log(Level.INFO, "Buscando todas las locales");
        return Response.ok(localeController.findAll()).build();
    }</pre>

Ahora definimos nuestra primera función que nos devolverá todos los Locales (países) disponibles en nuestra base de datos. Como se ve, ponemos la etiqueta @GET para especificar que debe ser llamada solo en peticiones HTTP tipo GET, y después añadimos la etiqueta
  
@Produces(MediaType.APPLICATION_JSON), para especificar que devuelve un resultado en formato JSON.

Dentro de la función ponemos usamos la clase **javax.ws.rs.core.Response** que a través de su metodo **ok**, al que debemos pasar el objeto que debe construir en formato JSON, cuando invoquemos a la función  **build**. En este caso el objeto pasado a la función **ok **  es una colección de clases de tipo Locales. Es decir, le pasamos esto: Collection<Locales>, al llamar a la función **findAll** de la clase **LocaleController **

Como se puede ver en el pantallazo, usando el excelente programa postman, podemos ver el resultado de la llamada a esta función:

<img class="size-full wp-image-342 aligncenter" src="http://www.profesor-p.com/wp-content/uploads/2018/10/Captura.png" alt="" width="545" height="688" srcset="http://www.profesor-p.com/wp-content/uploads/2018/10/Captura.png 545w, http://www.profesor-p.com/wp-content/uploads/2018/10/Captura-238x300.png 238w" sizes="(max-width: 545px) 100vw, 545px" />

  *  Función **findLocale**

<pre>@GET
    @Path("/{locale}")
    @Produces(MediaType.APPLICATION_JSON)
    public Response findLocale(@PathParam("locale") String locCodi) {   
        Locales l= localeController.findById(locCodi);

        try {
            l.getNombre();
        } catch (javax.persistence.EntityNotFoundException k)
        {            
            return Response.status(Response.Status.NOT_FOUND).build();
        }
        return Response.ok(l).build();
      
    }</pre>

Aquí devolveremos el registro que indicamos en el parámetro mandado. Utilizaremos una vez más la etiqueta @GET y despues añadiremos la etiqueta **@Path(&#8220;/{locale}&#8221;)** de tal manera que a esta función se le llamara cuando se haga una peticion HTTP tipo GET y que tenga un unico parametro

En la declaración de la función vemos la anotación **@PathParam**, con la cual indicamos que a la variable `<em>locCodi</em>`, se le debe asignar el valor del parámetro &#8220;locale&#8221;, que anteriormente hemos especificado en la anotación **@Path**.

La función como tal no tiene muchos misterios. Llamamos a la funcion findByID para buscar en la base de datos el codigo mandado. En caso de que no exista, al intentar recoger el nombre, con la linea `l.getNombre()`se lanzara una excepción tipo **EntityNotFoundException,** en cuyo caso devolveremos una respuesta tipo NOT_FOUND. En caso contrario devolveremos el objeto tipo Locales devuelto.

Y por hoy ya vale&#8230; otro dia continuare con las funciones que añaden, actualizan y borran los registros.

Hasta pronto  !!
  
&nbsp;