---
title: Perfiles en Spring Boot
author: El Profe
type: post
date: 2019-02-28T11:52:51+00:00
url: /2019/02/28/perfiles-en-spring-boot/
categories:
  - java
  - jpa
  - profiles
  - rest
  - spring boot
tags:
  - java
  - profiles
  - spring boot

---
### Introducción

Cuando se hacen aplicaciones empresariales, lo normal es que, como mínimo, primero se desplieguen en un entorno de pruebas y después  en el entorno de producción. Cada entorno de pruebas tendrá diferentes bases de datos, diferentes URLs y toda una serie de parámetros específicos, con el fin de que una aplicación en desarrollo no acceda nunca a datos reales.

### Estableciendo perfiles en la aplicación

**Spring** provee una manera sencilla de gestionar esta situación haciendo uso de los **perfiles**.

Hay varias maneras de establecer el perfil a utilizar por una aplicación:


1. Pasándolo como variable de entorno en JAVA. Por ejemplo:  &#8220;<strong><em>java -jar -Dspring.profiles.active=MI_PROFILE application.jar&#8221;</em></strong>
2. Pasándolo como un argumento al programa. &#8220;<em><strong>java -jar application.jar &#8211;spring.profiles.active=MI_PROFILE&#8221;</strong></em>
3. Especificarlo con una anotación en el propio programa, en la etiqueta @SpringBootTest, pasandole el parametro &#8220;spring.profiles.active&#8221;. Muy útil para prueba unitarias.@SpringBootTest(&#8220;spring.profiles.active=test&#8221;)
4. Con la anotación @ActiveProfiles@ActiveProfiles(&#8220;MI_PROFILE&#8221;)

### Creando Beans

En el ejemplo que esta disponible en <a class="url" href="https://github.com/chuchip/profilestest" target="_blank" rel="noopener noreferrer">https://github.com/chuchip/profilestest</a> se pueden ver como una aplicación saca diferentes mensajes y accede a diferentes datos según el perfil establecido.

El proyecto es una simple aplicación web, que accede a una base de datos H2, mostrando diferentes datos según el perfil. Para crear el proyecto simplemente hay que añadir las siguientes dependencias: JPA, H2 y WEB

![Starters](/images/2019/02/captura_starters.png)

Empezando por la clase principal veremos como definir diferentes **beans** dependiendo del perfil.


	@SpringBootApplication
	@Configuration
	public class ProfilesTests {

		@Autowired
		private Environment environment;
		
		public static void main(String[] args) {
			SpringApplication.run(ProfilesTests.class, args);
		}

		@Bean
		@Profile("test")
		IWrite getWriterTest()
		{
			return new WriteImpl("..test.. "+getProfile());		
		}
		@Bean
		@Profile("default")
		IWrite getWriterDefault()
		{
			return new WriteImpl("..default.. "+getProfile());		
		}
		@Bean
		@Profile("other")
		IWrite getWriterOther()
		{
			return new WriteImpl("..other.. "+getProfile());		
		}
		String getProfile()
		{
			if (environment.getActiveProfiles()==null)
				return "default";
			String[] profiles=environment.getActiveProfiles();
			return profiles.length&gt;0?profiles[0]:"default"; 
		}
	}


En las funciones getWriterXXX, se devuelve siempre un **bean** que implementa el interface `IWrite`. La definición de este interfaz es muy simple, como se puede ver en el siguiente código:


	public interface IWrite
	{
		public void writeLog(String log);
		public String getProfile();
	}

Y su implementación en la clase `WriteImpl` es también muy sencilla:

    public class <strong>WriteImpl</strong> implements IWrite{
    	private String profile;
    	
    	public WriteImpl(String profile)
    	{
    		this.profile=profile;
    	}
    	
    	public String getProfile()
    	{
    		return profile;
    	}
    	@Override
    	public void writeLog(String log) {
    		System.out.println("Profile: "+profile+" -> "+ log);
    		
    	}
    }
    

Volviendo a la clase principal, se ve que la función **getWriterTest()** instancia un objeto `WriteImpl`, pasando al constructor el literal `..test..` y lo devuelto por la función **getProfile()**, la cual devuelve un **String** con el perfil activo.

La función **getWriterDefault()** hace lo mismo que la anterior pero instanciando un objeto `WriteImpl` al cual en el constructor se le pasa el literal `..default..`y lo devuelto por la función **getProfile()**. Y lo mismo con la función **getWriterOther()** pero pasando el literal `..other..`

Lo importante de estas funciones es la anotación **@Bean** con la cual le decimos a _Spring_ que cuando en alguna parte del código se deba inyectar el objeto devuelto en esa función que ejecute esta función para conseguirlo.

### Un poco de teoría

Cuando **Spring** encuentre una anotación **@Autowired** con un objeto tipo **IWrite** como ocurre en el siguiente código:

	@Autowired
	IWrite out;	


buscara de alguna manera un objeto que implemente ese _interface_ o que se llame igual que el objeto en si.

Así, para conseguir el objeto, **Spring** tiene dos opciones:

  * Buscar dentro del proyecto una _class_ que se llame **IWrite** o una clase que implemente el interfaz **IWrite** que este anotada con la etiqueta **@Component**
  * Buscar dentro de una clase anotada con la etiqueta **@Configuration** una función anotada con **@Bean** y que devuelva un objeto de ese tipo.

Se pueden dar varios casos en este momento:

  1. Encuentra un objeto y lo usa.
  2. No encuentra un objeto con lo cual da un error y la aplicación no se ejecutara.
  3. Encuentra más de un objeto y como no sabe con cual quedarse da un error y la aplicación no se ejecuta.

Como se puede ver, en el código hay tres funciones que devuelven un objeto tipo **IWrite,** con lo cual **Spring** daría un error, para evitarlo añadimos la etiqueta **@Profile** . De esta manera si el perfil de la aplicación no coincide con el literal dentro de **@Profile** esa función será ignorada. Y como se puede ver, cada función tiene su propia anotación **@Profile** con lo cual _Spring_ solo tratara una de ellas.

Destacar que si lanzamos la aplicación sin especificar ningún perfil, **Spring** elige el perfil **default**.

Por supuesto, si lanzáramos la aplicación con un perfil que no fuera **default**,**test** o **other** la aplicación fallaría, al no encontrar ningún objeto que implemente el interfaz **IWrite**

### Creando Componentes

Para tener un ejemplo del caso en que **Spring** deba buscar una clase, en el proyecto de ejemplo se puede ver como se define el interfaz **IRead** el cual es implementado por la clases **ReadDefImpl** y **ReadOtherImpl**.

Observar como ambas clases tienen las anotaciones **@Component** y **@Profile** antes de la definición de la clase. Lógicamente en una clase la etiqueta **@Profile** tiene el parámetro **default** y en otra el parámetro es **other**.

Esta es la clase **ReadDefImpl**

	@Component
	@Profile("default")
	public class <strong>ReadDefImpl</strong> implements IRead{

		@Autowired
		ProfileRepository customerRepository;

		@Autowired
		IWrite out;
		
		public String readRegistry(int id)
		{
			out.writeLog("entry in ReadImpl" );
			Optional&lt;ProfileEntity&gt; registroOpc=customerRepository.findById(id);
			if (!registroOpc.isPresent())
			{
				System.out.println("Customer "+id+" NOT found");
				return null;
			}
			ProfileEntity registro=registroOpc.get(); 
			out.writeLog("Name  customer "+id+" is: "+registro.getName());
			return registro.getName();
		}
	}

Y así empieza la clase **ReadOtherImpl**

	@Component
	@Profile("other")
	public class <strong>ReadOtherImpl</strong> implements IRead{

El cuerpo es de la clase es exactamente igual que la de la clase **ReadDefImpl**

### Leyendo ficheros de propiedades

El código de la clase `ProfilesController` es el siguiente:

    @RestController
    public class ProfilesController {
    	@Autowired 
    	IRead read;
    	
    	@Value("${app.ID}")
    	int id;
    	
    	@Autowired
    	IWrite write;
    	
    	@GetMapping("/hello")
    	public String get(@RequestParam(value="name",required=false) String name)
    	{
            return "Hello "+(name==null?"SIN NOMBRE":name)+
            "<br><br> The name of the profile number:"+ id+" in the database H2 is: "+read.readRegistry(id)+
            "<br> The profile used in the application is : "+write.getProfile();
    				
    	}
    }
    

En esta clase podemos ver, la anotación **@Value(&#8220;${app.ID}&#8221;)** con ella lo que le estamos diciendo a **Spring** es que inicialice la variable que hay a continuación ( &#8220;`id`&#8220;) con el valor que encuentre en un fichero de _properites_. Como no especificamos ninguno buscara dentro de `application.properties` y encontrara la siguiente línea:

    app.ID=1
    

con lo cual la variable `id` tendrá el valor **1**

Pero si nos fijamos en nuestro proyecto además del fichero `application.properties` también tenemos un fichero llamado `application-other.properties`, esto es porque _Spring_ en el caso de que el perfil sea **other** buscara primero el fichero `application-other.properties` para establecer las propiedades.

De esta manera, cuando el perfil sea **other** leerá el fichero `application-other.properties` y encontrara la línea:

    app.ID=2
    

con lo cual la variable **id** tendrá el valor **2**

Es importante recalcar que el fichero `application.properties` se leerá y procesara siempre, independientemente del perfil elegido, pero luego se buscara el del perfil activo. Si no queremos que lea ese fichero, una solución seria renombrar el fichero a `application-default.properties` con lo cual solo se usaría con el perfil **default**.

### Probando la aplicación

Para probar la aplicación en _Eclipse_ podremos crear dos configuraciones. Una para el perfil **default**
![Starters](/images/2019/02/captura1.png)

donde podemos ver que no especificamos ningún perfil con lo cual _Spring_ asignara el perfil **default** y otra para el perfil **other**
![Starters](/images/2019/02/captura2.png)


La del perfil **default** escuchara en el puerto 8080 y la de **other** en el 8081. Eso esta especificado con la línea `server.port=8080` en el fichero `application.properties` y con la línea `server.port:8081`en el fichero `application-other.properties`

Como se puede ver en las siguientes pantallas, la llamada a <a class="url" href="http://localhost:8080/hello?name=profesor" target="_blank" rel="noopener noreferrer">http://localhost:8080/hello?name=profesor</a> devuelve la siguiente salida:
![Starters](/images/2019/02/captura-navegador8080.png)


Se puede ver como el valor de la variable **id** es 1 y como lee en la base de datos H2, el valor del registro correspondiente.

En la salida estándar veremos el siguiente texto:

	Profile: ..default.. default -&gt; entry in ReadImpl
	Profile: ..default.. default -&gt; Name customer 1 is: default

Si llamamos a <a class="url" href="http://localhost:8081/hello?name=profesor" target="_blank" rel="noopener noreferrer">http://localhost:8081/hello?name=profesor</a>, veremos la siguiente salida en el navegador:

![Starters](/images/2019/02/captura-navegador8080.png)

En la salida estándar veremos el siguiente texto:

<pre>Profile: ..other.. other -&gt; entry in ReadImpl
Profile: ..other.. other -&gt; Name customer 2 is: other</pre>

En el [articulo siguiente][1] podéis ver el uso de los perfiles para testear la aplicación con **JUnit**

Hasta entonces, ¡¡ que la curiosidad no os abandone 😉 !!

 [1]: 2019/03/01/uso-de-perfiles-en-testing-en-spring-boot/