---
title: Beans avanzados en Spring
author: El Profe
type: post
date: 2019-04-18T21:26:57+00:00
url: /2019/04/18/beans-avanzados-en-spring/
categories:
  - spring
tags:
  - beans
  - spring

---
En esta ocasión he cogido un proyecto de [SimpleProgramming][1] el cual tiene un video en [Youtube][2] donde explica como cargar Beans dinámicamente usando Spring (en Ingles).

Imaginemos que tenemos un programa que dependiendo de unos parámetros deba cargar un clase u otra, donde está definida la lógica a seguir. Por supuesto podemos anidar _condiciones_ e instanciar las clases debidas, pero eso tiene un problema y es que si mañana debemos añadir una lógica nueva, deberemos incluir una condición más para cargar la nueva clase, y podríamos introducir errores en el código.

Lo ideal seria que tuviéramos una interfaz, y después una serie de clases que implementaran esa interfaz. Después, de alguna manera, deberíamos cargar la clase adecuada sin tener una sola condición en el código.

Pues con Spring eso se puede hacer usando la clase **ServiceLocatorFactoryBean**. Esta clase definida en la siguiente URL <https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/beans/factory/config/ServiceLocatorFactoryBean.html> implementa una fabrica de _Beans_, creando un _proxy_ de tal manera que devuelve una clase tipo [`BeanFactory`][3], la cual nos permitirá crear nuestros Beans dinámicamente.

Esto que parece bastante confuso explicado así es realmente muy fácil de realizar. Pongámonos manos a la obra.

En el ejemplo que tenéis en <https://github.com/chuchip/springboot-servicelocatorfactorybean> esta la interfaz **AdapterService**, cuyo código como veis a continuación es muy simple:

<pre><code class="language-java" lang="java">public interface AdapterService
{
	public String process();
}
</code></pre>

Se crean 4 clases que implementan ese interface **BikeService**, **BusService**, **TruckService** y **CarService**. A continuación tenéis el código de **CarService**

<pre><code class="language-java" lang="java">@Service("Car")
public class CarService implements AdapterService {
	int numberExecution=1;
	
	@Override
	public String process() {		
		return "inside car service -  number of executions: "+(numberExecution++);
	}
}
</code></pre>

Lo importante de esta clase es la etiqueta `@Service("car")` pues especificamos el nombre con el que deberemos cargar la clase.

En las demás clases existen las etiquetas: `@Service("Bike")`, `@Service("Bus")` y `@Service("Truck")`siendo su código prácticamente idéntico, solo cambiando el mensaje a devolver.

Ahora se debe crear el interface que pasaremos a la clase **ServiceLocatorFactoryBean** en este ejemplo es la clase **ServiceRegistry**

<pre><code class="language-java" lang="java">public interface ServiceRegistry {
	public AdapterService getService(String serviceName);
}
</code></pre>

En esta clase deberemos definir la función `getService` que es la que **ServiceLocatorFactoryBean** invocara. Esta función debe recibir un parámetro que será el nombre del _bean_ (el definido con las etiquetas `@service`), además devolverá un objeto que implemente el interfaz adecuado (en este caso **AdapterService)**.

En la clase **VehicleConfig** definimos la función que nos devolverá el **FactoryBean**

<pre><code class="language-java" lang="java">@Configuration
public class VehicleConfig {
	@Bean
	public FactoryBean&lt;?&gt; factoryBean() {
		final ServiceLocatorFactoryBean bean = new ServiceLocatorFactoryBean();
		bean.setServiceLocatorInterface(ServiceRegistry.class);
		return bean;
	}
}
</code></pre>

En ella, instanciamos una clase tipo **ServiceLocatorFactoryBean**, especificando el interface que debe devolver y luego retornamos ese objeto **ServiceLocatorFactoryBean** que por supuesto implementa el interfaz **FactoryBean**

Para ver la funcionalidad de estos beans, en el proyecto de ejemplo, hemos creado un controlador **rest** muy simple.

    @RestController
    public class VehicleController {
    	@Autowired
    	private ServiceRegistry serviceRegistry;
    	
    	@GetMapping("{vehicle}")
    	public String  processGet(@PathVariable String vehicle) {
    		return serviceRegistry.getService(vehicle).process();
    	}
    }
    

En ella, dependiendo de la URL llamada, que será volcada en la variable `vehicle` se llamara a una de las clases que implementan el interfaz **AdapterService** y todo ello sin un solo `if`

Así si ejecutamos:

    $ curl  -s  http://localhost:8080/Car
    inside car service -  number of executions: 1
    $ curl  -s  http://localhost:8080/Car
    inside car service -  number of executions: 2
    

se puede observar como es cargada la clase `CarService`y como se mantiene en el contexto de _Spring_, pues se puede ver como el numero de ejecuciones aumenta con cada llamada.

Si ejecutamos las siguientes sentencias:

    $ curl  -s  http://localhost:8080/Truck
    inside truck service -  number of executions: 1
    $ curl  -s  http://localhost:8080/Truck
    inside truck service -  number of executions: 2
    $ curl  -s  http://localhost:8080/Car
    inside car service -  number of executions: 3
    

podemos observar como funciona todo correctamente.

Por supuesto si intentamos cargar un _Bean_ no definido dará un error:

    $ curl  -s  http://localhost:8080/Boat
    {"timestamp":"2019-04-18T21:07:23.139+0000","status":500,"error":"Internal Server Error","message":"No bean named 'Boat' available","path":"/Boat"}
    

Y de esta manera tan simple. gracias a la magia de Spring, se pueden crear programas fácilmente ampliables y modulares.

Hasta otra y no olvidéis seguirme en [Twitter][4] para estar al tanto de nuevas entradas de este blog.

 [1]: https://github.com/SimpleProgramming/springboot-servicelocatorfactorybean
 [2]: https://www.youtube.com/watch?v=rHk5pijFymo
 [3]: https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/beans/factory/BeanFactory.html
 [4]: https://twitter.com/chuchip