---
title: JPA + Hibernate en Spring
pre: "<b>o </b>"
author: airec69
type: post
date: 2018-08-25T18:14:42+00:00
url: /2018/08/25/jpa-hibernate-en-spring/
categories:
  - java
  - jdbc
  - jpa
  - spring

---
En un [entrada anterior][1], explique como crear nuestra conexión a la base de datos, usando JNDI. En esta entrada explicare como usar esa conexión con JPA.

Recordar que el código fuente de de este ejemplo esta en: <a href="https://github.com/chuchip/jdbc_jpa_tomcat" target="_blank" rel="noopener">https://github.com/chuchip/jdbc_jpa_tomcat</a>

Importante recalcar que este ejemplo **solo** funciona con Java 1.8 o superior.

Lo primero explicar un poco de que va esto de JPA.  **JPA** son las siglas de  Java Persistence API. Es decir la API de persistencia en Java. Vale, que te has quedado como antes, ¿ no ? 😉 . Bueno, la idea es tener una metodología de tener objetos en nuestro entorno Java y que esos objetos sean un reflejo de las diferentes tablas de la base de datos, de tal manera que nosotros modifiquemos nuestros objetos y en la base de datos se vean reflejados esos cambios.

La idea parece interesante, ¿ verdad ?. Bueno, pues encima no es nada complicado el hacerlo.

Manos a la obra.

Lo primero vamos a crear el objeto que representa la tabla de nuestra base de datos. Así, en nuestra base de datos tenemos la tabla **usuario** que solo tiene dos campos, el **login** y el **nombre** del usuario. Ambos campos son cadenas de caracteres (String, vamos). En nuestra tabla  **usuario** no puede haber dos **login** igual, es lo que se llama un indice unico o &#8220;_unique constraint&#8221;_ que dicen los ingleses.

Definamos entonces nuestro objeto (usando la librería <a href="https://projectlombok.org/" target="_blank" rel="noopener">Lombok</a>, de la que ya hablaba [en otra entrada][2])

```
@Data
@Entity
@Table(name = "usuario",   uniqueConstraints = {   @UniqueConstraint(columnNames = {"login"})}) // Esto no es necesario en este ejemplo
public class Usuario implements Serializable {

    @Id
    String login;

    @Column
    String nombre;
   public Usuario() {}
   public Usuario(String login, String nombre) {
        this.login = login;
        this.nombre = nombre;
    }
}
```

Como se ve fácilmente, es una simple clase POJO, que implementa el interface Serializable. Solo tiene dos campos y unas cuantas anotaciones, que ahora explicare.

Lo primero es declarar la clase del tipo **Entidad**, para eso usaremos la anotación **@Entity**. Gracias  a eso, JPA marcara esa clase para poder usarla en su entorno de persistencia.

El siguiente parámetro @Table indica a que tabla de nuestra base de datos, hace referencia esa clase, ademas de cual es su  indice único . Si, como es el caso, el indice es simple y ademas la tabla se llama como nuestra clase no hace falta poner ese parámetro pero yo lo he puesto para que sepáis que exista ( y por manías 😉 )

Luego, dentro de la clase veis que antes de la definición de la variable **login** tenemos la anotación **@Id**,  eso indicara que ese es nuestro campo indice o único. Antes de la definición de la variable **nombre** tenemos la anotación **@Column** para indicar que es una columna más de nuestra tabla. Aclarar que todas esas anotaciones permiten más parámetros.

Hay muchísima documentación en la red, sobre JPA, podéis empezar, por ejemplo, por este <a href="https://aulavirtual.um.es/access/content/group/3871_G_2011_N_N/Teoria/T5B%20-%20JPA.pdf" target="_blank" rel="noopener">PDF de la Universidad de Oviedo</a>

Bien, una vez tenemos nuestra entidad vamos a trabajar con ella. Por hacerlo simple vamos a hacer uso Spring y de su interfaz <a href="https://docs.spring.io/spring-data/commons/docs/current/api/org/springframework/data/repository/CrudRepository.html" target="_blank" rel="noopener">CrudRepository</a>. Este interfaz nos permite con apenas 3 lineas de código acceder a los datos que representa nuestro POJO.

Si, aunque parezca increíble, con solo crear esta interface ya podremos acceder a nuestra base de datos.

<pre>public interface UsuarioRepositorio extends CrudRepository<Usuario, String> { }</pre>

Veis que he creado un interface, al que he llamado **UsuarioRepositorio** que extiende del interface **CrudRepository** y le he añadido el nombre de la clase **U****suario**, que es mi @Entity, y el tipo de campo (String) que es el indice único.

¿ Ya esta ?. Pues sí, ya esta. Ya podéis acceder a vuestra base de datos. ¿ Qué como ?. Pues lógicamente usando las funciones que implementa el interfaz **CrudRepository.** Por ejemplo existe la función **findById,** si ejecutáis esa función pasándole el nombre del usuario os devolverá un objeto  **Optional<****usuario** > . Os lo explico mejor con un ejemplo.

Creamos esta clase en nuestro proyecto.

```
public class buscaUsuario
{

   @Autowired
   UsuarioRepositorio usuRep;

  public String getNombreUsuario(String loginUsuario)
  {
        Optional usu=usuRep.findById(loginUsuario);

        String usuario = (usu.isPresent() ? usu.get().getNombre() : "Usuario "+loginUsuario+" No encontrado");  }
  }
}
```

Teniendo inyectada  una referencia, con la  anotación @Autowired,  a nuestro interface  UsuarioRepositorio (Spring hace la magia), en la función getNombreUsuario, invocamos a **findById ,** la cual nos devuelve una clase tipo <a href="https://docs.oracle.com/javase/8/docs/api/java/util/Optional.html" target="_blank" rel="noopener">Optional</a>. Este tipo de objeto es para poder encapsular un posible valor null de tal manera que siempre se devuelva algo, aun en el caso de que no se encuentre ningún valor.  Para saber si se ha encontrado algo usaremos **isPresent** el cual nos devolverá **true** si ha encontrado algún registro.

Os estaréis preguntando, como mínimo, dos cosas:

1. &#8211; ¿ Como hace esto Java, si realmente no hemos creado ninguna clase, solo un interfaz ?

2.- ¿ Qué pasa si yo quiero acceder a a campos que no son indices o varios campos simultáneamente ?

La respuesta a la primera pregunta, esta en la Programación Orientada a Objetos de Spring (echar un vistazo a JAspect). Gracias a esta tecnología, cuando Spring recorre nuestras clases (al desplegarse la aplicación en nuestro servidor de aplicaciones) y ve un interfaz que extiende de la clase  **CrudRepository** (hay otras clases que mejoran esta, pero no hablare de ellas en  este ejemplo) crea _al vuelo_ una clase que implementa vuestro interfaz (en este caso **UsuarioRepositorio**) con sus correspondientes funciones. Por eso, en nuestras clases, podemos ejecutar algo que ni siquiera hemos creado.

La respuesta a la segunda pregunta es la anotación @Query. Gracias a ella en nuestro interfaz podemos crear nuevas funciones, que ejecuten las sentencias SQL que necesitemos.

Así si ponemos la siguiente función en nuestra clase **UsuarioRepositorio**

```
@Query("select u from Usuario u where u.nombre like :nombre order by u.nombre")
List<Usuario> buscaPorNombre(@Param("nombre") String nombre);
```

Podremos buscar todos los usuarios cuyo nombre contenga el String pasado a la función **buscaPorNombre.**

Observar que la sentencia SQL no es una sentencia SQL estándar sino que hace uso de <a href="https://es.wikipedia.org/wiki/Java_Persistence_Query_Language" target="_blank" rel="noopener">Java Persistence Query (JPQ)</a> Este lenguaje es muy parecido a SQL pero tiene sus particularidades 😉

Para facilitar más las cosas,  Spring es capaz de interpretar, a través del nombre de la función lo que quieres hacer con esa función.

Así, en el ejemplo anterior, si a nuestra función le llamamos findIsLikeNombreOrderByNombre, haría lo mismo, pero no tendriamos que tener la anotación @Query. Spring la crea por nosotros, interpretando el nombre de la función.

Así nuestra clase quedaría así:

```
public interface UsuarioRepositorio extends CrudRepository<Usuario, String> {
    
  
    @Query("select u from Usuario u where u.nombre like :nombre order by u.nombre")
    List<Usuario>buscaPorNombre(@Param("nombre") String nombre);
    
    /**
     * Esta funcion hace exactamente lo mismo que la funcion buscaPorNombre pero utilizando DSL (Domain Specificic Lenguage) de Spring
     * @param nombre Nombre de usuario a buscar (sin wildcards, ya lo pone JPL)
     * @return Lista de Usuarios a buscar
     */
    List<Usuario> findIsLikeNombreOrderByNombre(String nombre);
}
```

Esto se hace gracias a la Spring y su Domain Specificic Lenguage (DSL). Tenéis una referencia de como formar Querys usando esta nomenclatura en <a href="https://docs.spring.io/spring-data/data-jpa/docs/1.0.0.M1/reference/html/#jpa.query-methods.query-creation" target="_blank" rel="noopener">esta página de Spring</a>

Y nada más por hoy, solo recordaros que tenéis muchísima documentación sobre JPA y Spring en la web, os dejo un par de enlaces (en español) donde se habla más de ello:

  * <a href="http://acodigo.blogspot.com/2017/03/spring-data-jpa-acceso-datos-simple-y.html" target="_blank" rel="noopener">http://acodigo.blogspot.com/2017/03/spring-data-jpa-acceso-datos-simple-y.html</a>
  * <a href="https://www.adictosaltrabajo.com/tutoriales/spring-data-jpa/" target="_blank" rel="noopener">https://www.adictosaltrabajo.com/tutoriales/spring-data-jpa/</a>

Espero que os haya gustado la entrada.

Un saludo

El profe.

 [1]: http://www.profesor-p.com/2018/08/21/conectando-con-postgresql-usando-jndi-y-spring-en-tomcat-parte-1/
 [2]: http://www.profesor-p.com/2018/08/24/jpa-con-lombok/