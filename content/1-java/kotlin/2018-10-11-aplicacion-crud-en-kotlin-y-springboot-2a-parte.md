---
title: Aplicacion CRUD con REST en Kotlin y SpringBoot (2ª Parte)
author: El Profe
type: post
weigth: 11
date: 2018-10-11T09:50:19+00:00
url: /2018/10/11/aplicacion-crud-en-kotlin-y-springboot-2a-parte/
categories:
  - CRUD
  - jpa
  - kotlin
  - spring boot
tags:
  - crud
  - jpa
  - kotlin
  - spring boot

---
Hola de nuevo, chicos.

Continuo con la entrada <a href="http://www.profesor-p.com/2018/10/11/aplicacion-crud-en-kotlin-con-springboot/" target="_blank" rel="noopener">http://www.profesor-p.com/2018/10/11/aplicacion-crud-en-kotlin-con-springboot/</a> para ver como realizar las peticiones REST , con protocolo JSON que es el estándar de facto 😉 en Kotlin.

Una vez que ya tenemos la lógica de acceso a nuestra base de datos, tenemos que hacer la parte Web. Pues, aunque os parezca increíble, esto se hace con una sola clase y ademas de muy pocas lineas.

La clase en cuestión es `ApiController.kt`

```
@RestController
@RequestMapping("/api/")
class ApiController {
       @Autowired
	lateinit var localeRepository: LocaleRepository

	@CrossOrigin("http://localhost:4200")
	@GetMapping("/")
	fun getAll(): Iterable&lt;Locales&gt;
	{
		return localeRepository.findAll();		
	}


​	
	.....
}
```


Bueno, lo primero es poner la anotación **@RestController** para que Spring sepa que esta clase responderá a las peticiones REST y despues especificar la ruta que tratara esta clase con la anotación** @RequestMapping(&#8220;/api/&#8221;). **Es decir esta clase responderá en la URL: **http://localhost:8080/api/**

Inyectamos una referencia a la clase **LocaleRepository **con la etiqueta _@AutoWired_, para poder acceder a nuestro repositorio (la base de datos, vamos). Observar el modificador  **lateinit** para que Kotlin no se queje de que la variable es null.

  * Función **getAll**

Esta responderá a las peticiones GET  en &#8220;/&#8221; devolviendo una lista con todos los objetos  tipo Locales disponibles. Observar la anotación **@CrossOrigin(&#8220;http://localhost:4200&#8221;) **para que se pueda acceder desde un navegador solo si el cliente esta accediendo a http://localhost:4200.

Esto es porque, por seguridad, los navegadores no permiten hacer peticiones AJAX (con las cuales se hacen las peticiones REST en angular normalmente) a una dirección que no sea la de la URL principal. Vamos, para que si tu estas en www.google.com, la página por debajo no pueda acceder a www.elpais.com. Sin embargo con la etiqueta @**CrossOrigin **nuestra aplicación informara al navegador  de que permita acceder siempre y cuando la URL de origen sea la especificada.

Tener en cuenta que esta limitación la imponen los navegadores Web (Chrome, Firefox, IE, etc), si accedemos directamente con otro programa a la dirección** http://localhost:8000/api** no tendrá efecto esta directiva.

Aquí os muestro un pantallazo de la salida de  PostMan:

![Captura6](/img/2018/10/Captura-6.png)

  * Función **getByQuery**

```
@CrossOrigin("http://localhost:4200")
@GetMapping("/{codigo}/{nombre}")
fun getByQuery(@PathVariable codigo:String,@PathVariable nombre:String): Iterable&lt;Any&gt;
{
    return localeRepository.findLike("%"+codigo+"%","%"+nombre.toUpperCase()+"%"); 
}
```

Aquí vemos de nuevo la etiqueta @CrossOrign y la etiqueta **@GetMapping(&#8220;/{codigo}/{nombre}&#8221;)** con lo cual especificamos que esta función tratara las llamadas a la URL: **http://localhost:8080/api/XX/YY  **. Observar las etiquetas **@PathVariable **en los parámetros de la función para especificar que debe pasar las diferentes partes de la ruta a las variables de la función.

Llamamos a la función **findLike** de nuestro clase repositorio la cual nos devolverá todos los objetos Locales que cumplan los criterios de la búsqueda.

  * Función **insertar**

```
    @CrossOrigin("http://localhost:4200")
	@PostMapping("/")	
	fun insertar(@RequestBody locales:Locales): ResponseEntity&lt;Any&gt;
	{
		if (localeRepository.existsById(locales.codigo ) )	
			return ResponseEntity(HttpStatus.CONFLICT);

		localeRepository.save(locales)
		
		return ResponseEntity
			.created( URI("/api/"+locales.codigo)).body("");
	}
```
Esta función sera invocada cuando la petición HTTP sea de tipo POST, a la ruta &#8220;/&#8221;, como así se indica con la etiqueta **@PostMapping(&#8220;/&#8221;). **En la función incluimos la etiqueta **@RequestBody **para indicarle a Spring que en el cuerpo de la petición HTTP ira un objeto tipo Locales. en formato  JSON, al no especificar lo contrario.

Comprobamos si el código ya existe, para devolver una respuesta con el código CONFLICT, con lo cual indicaríamos al cliente que hay un error.

Después guardamos la entidad y devolvemos un código CREATED, con la URL que indica el código del país insertado.

Esto es un pantallazo donde se ve como insertaríamos un nuevo país en la base de datos.

![captura-7](/img/2018/10/Captura-7.png)

  * Función **deleteByCodigo** 

```
@CrossOrigin("http://localhost:4200")
	@DeleteMapping ("/{codigo}")
	fun deleteByCodigo(@PathVariable codigo:String):ResponseEntity&lt;Any&gt;
	{
		if (!localeRepository.existsById(codigo) )	
			throw NotFoundException(codigo)
		localeRepository.deleteById(codigo);
		return ResponseEntity( HttpStatus.OK)
	}
```


Esta función sera llamada cuando recibamos una petición HTTP tipo DELETE, a la ruta /XX. Es decir responderá con una petición DELETE como la del pantallazo que adjunto:

![Captura-8](/img/2018/10/Captura-8.png)

Observar que si el país no existe lanzamos una excepción tipo **NotFoundException . **

Detallo la clase a continuación:

```
@ResponseStatus(HttpStatus.NOT_FOUND)
class NotFoundException: RuntimeException {

	constructor(codigo: String?): super("No encontrados registro "+codigo);
}
```


Esta clase tiene la etiqueta **@ResponseStatus **que indica que tipo de respuesta recibirá el cliente cuando se lance esta excepción. En este caso recibirá un tipo NOT_FOUND (404), con el mensaje **&#8220;No encontrados registro xxx&#8221;, **como se ve en la siguiente pantalla.

![imagen-9](/img/2018/10/Captura-9.png)

  * Función **update**

```
    @CrossOrigin("http://localhost:4200")
	@PutMapping ("/{codigo}")
	fun update(@PathVariable codigo:String,@RequestBody locales:Locales):ResponseEntity&lt;Any&gt;
	{
		if (!localeRepository.existsById(codigo) )	
			throw NotFoundException(codigo)

		if (!codigo.equals(locales.codigo))
			throw ConflictException(codigo)
		localeRepository.save(locales);
		return ResponseEntity( HttpStatus.OK)
	}
```

Esta función actualizara el nombre de un país mandado. Observar que recibe en la ruta el código del país a modificar y en el cuerpo de la petición (Body) un objeto Locales.

Si el código del país  mandado es diferente al del objeto, lanza una excepción tipo **ConflictException **que es muy parecida a **NotFoundException **pero devolviendo un código HTTP **CONFLICT.**

Y ya esta chicos. Para probar la aplicación usaremos el programa realizado en Angular, que explicaba en la entrada <http://www.profesor-p.com/2018/10/08/aplicacion-crud-en-angular/> , teniendo cuidado de cambiar el valor de la variable URL de la clase **datosserver.service.ts** para que apunte a donde escucha nuestra aplicación.

```
  export class DatosserverService {

    url:string;
  constructor(private _http:HttpClient) { 
	this.url="http://localhost:8080/api/";
  }
....
```

Y veremos como apenas 4 sencillas clases tenemos una aplicación totalmente funcional. ¿ A que es increíble la potencia de Spring y más si la juntamos con Kotilin?. ¡¡ Pues a programar que es la mejor manera de aprender 😉 !!
	

![captura-4](/img/2018/10/Captura-4.png)
