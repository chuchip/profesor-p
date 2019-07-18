---
title: Aplicacion CRUD con REST en Kotlin y SpringBoot
author: El Profe
type: post
date: 2018-10-11T06:01:11+00:00
url: /2018/10/11/aplicacion-crud-en-kotlin-con-springboot/
categories:
  - CRUD
  - jpa
  - kotlin
  - rest
  - spring boot
tags:
  - crud
  - jpa
  - kotlin
  - rest
  - spring boot

---
Buenas chavales,

En esta nueva entrada os enseñare como realizar un programa CRUD, que sirva peticiones REST, usando Spring Boot y como lenguaje de programación usar Kotlin.

Creo que no hará falta aclarar que una aplicación CRUD es la típica aplicación que permite consultar, insertar, modificar y borrar los datos de una tabla. Vamos, lo que se suele ir llamando un mantenimiento de una tabla.

Esta aplicación seria una variación del que explique en la entrada <a href="http://www.profesor-p.com/2018/10/06/aplicacion-rest-en-javaee/" target="_blank" rel="noopener">http://www.profesor-p.com/2018/10/06/aplicacion-rest-en-javaee/</a>  , donde se utilizaba JavaEE. Como frontend para  este programa podéis usar el realizado en Angular, que  explico en esta entrada <a href="http://www.profesor-p.com/2018/10/08/aplicacion-crud-en-angular/" target="_blank" rel="noopener">http://www.profesor-p.com/2018/10/08/aplicacion-crud-en-angular/</a> ya que lo he hecho totalmente compatible.

El código fuente lo tenéis en mi repositorio de GitHub, en: <a href="https://github.com/chuchip/restKotlin" target="_blank" rel="noopener">https://github.com/chuchip/restKotlin</a>

Bueno, !!manos a la obra!!

Lo primero sera cambiar de IDE, ya que mi querido NetBeans no soporta el lenguaje Kotlin. Por ello utilizaremos Eclipse, habiéndole instalado el plugin de Kotlin y de Spring Boot. Tampoco os voy a explicar como instalar estos plugins pues tenéis unos 600 manuales por la web y tampoco es cuestión de alargarme demasiado 😉

Utilizando el magnifico asistente de &#8220;Spring Starter Project&#8221; crearemos un proyecto Spring Boot, al que le tendremos que añadir los paquetes &#8220;H2&#8243;,&#8221;JPA&#8221; y &#8220;WEB&#8221;. ¡¡ Acordaros de elegir Kotlin como lenguaje de programación !!

Una vez tenemos nuestro proyecto creado, debemos comprobar la configuración del compilador de Kotlin, para ello nos iremos a las Propiedades del proyecto y elegiremos la pestaña &#8220;Kotlin Compiler&#8221;. Ahí deberemos especificar que que la versión de java para la que debe compilar es la 1.8  y deberemos incluir los plugins Spring y JPA. La pantalla deberá quedar como esta:

<img class="imagen_con_borde wp-image-370 size-full aligncenter" src="http://www.profesor-p.com/wp-content/uploads/2018/10/Captura-5.png" alt="" width="527" height="613" srcset="http://www.profesor-p.com/wp-content/uploads/2018/10/Captura-5.png 527w, http://www.profesor-p.com/wp-content/uploads/2018/10/Captura-5-258x300.png 258w" sizes="(max-width: 527px) 100vw, 527px" />

&nbsp;

Ahora ya podemos ponernos a teclear 😉

Eclipse nos habrá creado la clase de entrada , que sera algo como esto:

<pre>import org.springframework.boot.autoconfigure.SpringBootApplication
import org.springframework.boot.runApplication

@SpringBootApplication
class RestKotlinApplication

fun main(args: Array&lt;String&gt;) {
    runApplication&lt;RestKotlinApplication&gt;(*args)
}</pre>

Esta clase la dejaremos sin tocar, pues nos  sirve perfectamente.

Deberemos configurar el fichero **application.properties** de Spring Boot, para especificarle que queremos usar una base de datos H2 y ponerle los parámetros de conexión. Quedara algo como esto:

<pre># H2
spring.h2.console.enabled=true
spring.h2.console.path=/h2
# Datasource
spring.datasource.url=jdbc:h2:file:~/test
spring.datasource.username=sa
spring.datasource.password=
spring.datasource.driver-class-name=org.h2.Driver</pre>

Observar las anotaciones: **spring.h2.console.enabled=true** y **spring.h2.console.path=/h2** que nos permitira acceder a la consola de H2. Podremos acceder yendo a la URL: **http://localhost:8080/h2**

También he creado un fichero llamado **data.sql** que  Spring Boot ejecutara para precargar la base de datos con algunos valores, porque, como veis, la base de datos se crea solo en memoria.

El fichero deberá estar en el directorio **resources** y es algo parecido a esto:

<pre>insert into locales values('es_ES', 'español (España)');
insert into locales values('fr', 'frances');
insert into locales values('fr_BE', 'francés (Bélgica)');
insert into locales values('ca_ES', 'catalán (España)');
insert into locales values('es_AR', 'español (Argentina)');</pre>

La aplicación actualmente ya es ejecutable,  aunque solo podríamos acceder a la consola de H2. !! Podéis jugar con ella, que esta muy bien hecha y tiene mucha potencia!!

Ahora pasamos a crear nuestra entidad de Idiomas, a la que yo he llamado `locales.kt`

<pre>import com.fasterxml.jackson.annotation.JsonProperty
import javax.persistence.Basic
import javax.persistence.Column
import javax.persistence.Entity
import javax.persistence.Id
import javax.persistence.Table
import javax.validation.constraints.NotNull
import javax.validation.constraints.Size

@Entity
@Table(name = "locales")

data class Locales  (
   @Id  @Basic(optional = false)
    @NotNull
    @Size(min = 1, max = 5)
   @JsonProperty(value = "codigo")
    @Column(name = "loc_codi") val codigo: String,

   @JsonProperty(value = "nombre")
    @Column(name = "nombre") val nombre: String 
   )
{
	
}</pre>

Como veis es una clase muy normalita. Aclarar que he puesto algunas etiquetas que no son necesarias para que veáis como podriais usarlas. Así las etiquetas: @Table(name = &#8220;locales&#8221;), @JsonProperty(value = &#8220;codigo&#8221;) y @ColumName no son necesarias pues los nombres de las clases y variables coinciden con los de la base de datos.

Observar como se define la clase con la etiqueta **data class** y lo fácil que es crear una entidad en Kotlin al no tener que definir setters, getters  y definir las propias variables globales en el constructor. Vamos, que si os fijais, la clase como tal no tiene nada ;-).

Ahora definimos la clase que hara de repositorio, a la que yo he llamado: **LocaleRepository.kt**

<pre>import org.springframework.data.jpa.repository.Query
import org.springframework.data.repository.CrudRepository
import profesorp.kotlin.entity.Locales

interface LocaleRepository : CrudRepository&lt;Locales,String&gt; {
	@Query("SELECT l FROM Locales l where l.codigo like ?1 and upper(l.nombre) like ?2")
	fun findLike(codigo:String, nombre:String):Iterable&lt;Locales&gt;;
	
}</pre>

Otra clase super compleja 😉

¿Hay que explicar algo ?. Bueno, quizas que he creado una nueva función que usaremos para poder filtrar los países por nombre y/o código. Sí, esa funcion que he llamado **findLike**

Bueno, y por ahora ya vale. En la próxima entrada detallare la clase que sirve las peticiones REST.

&nbsp;

&nbsp;

&nbsp;

&nbsp;