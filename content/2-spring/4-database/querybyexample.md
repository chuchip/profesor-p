---
title: Usar QBE en Spring Data
pre: "<b>o </b>"
author: profesorp
type: post
date: 2021-01-23
url: /spring/data/qbe
categories:
  - java
  - jpa
  - qbe
  - spring data JPA
  - spring boot
tags:
  - java
  - spring
  - jpa
  - qbe
  - spring data JPA

---

A menudo, cuando arranca nuestra aplicación,  tenemos que tener ciertos registros en algunas tablas de diccionarios. Por ejemplo, en la tabla 'paises' puede que debamos tener cargados los países del mundo. O en la tabla 'roles', debemos tener definidos una serie de roles.

Una de las maneras de cargar esos datos es definiendo una rutina en el programa, la cual se ejecutara al inicio y  que insertara esos registros necesarios. El problema que nos podemos encontrar es que entonces, debemos borrar todos los registros y luego volverlos a insertar para no tener registros duplicados. 

Estaría bien que pudiéramos comprobar fácilmente si el registro a insertar  ya existe y en caso de que sea así que no lo insertara. 

En este artículo vamos a utilizar **Query By Example** (https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#query-by-example) de **Spring Data JPA** para realizar esta tarea.

Empezaremos creando un proyecto en **Spring Boot**, con los siguientes *starters*:

- Spring Data JPA
- Lombok
- H2 (la base de datos que usaremos)

En este proyecto he creado dos entidades para las  pruebas. Son estas:

```java
@Entity @Setter @Getter @NoArgsConstructor @AllArgsConstructor @Builder @ToString
public class Alumno {
	@GeneratedValue(strategy=GenerationType.AUTO) 
	@Id	
	int id;
	
	String nombre;
	Integer edad;
	String clase;
}
```

```java
@Entity @Setter  @Getter @Builder @NoArgsConstructor @AllArgsConstructor @ToString
public class Profesor {
	@GeneratedValue(strategy=GenerationType.AUTO) 
	@Id
	Integer id;
	
	String nombre;
	String materia;
	String colegio;		
	Integer numeroClase;
}
```

También  he creado sus correspondientes interfaces para los repositorios.

```java
public interface AlumnoRepository extends JpaRepository<Alumno, Integer>{}
public interface ProfesorRepository extends JpaRepository<Profesor, Integer>{}
```

Ahora, en la clase JpaQueryByExampleApplication he añadido este código

```java
@Autowired 	AlumnoRepository alumnRepository;
@Autowired 	ProfesorRepository teacherRepository;

@PostConstruct
void iniciar() {
    log.info("\n\nINSERTANDO PROFESORES .... \n\n");
    var teacher = Profesor.builder().nombre("Nombre1").colegio("Colegio1").materia("Matematicas").numeroClase(55)
        .build();

    guardar(teacherRepository, teacher);

    teacher = Profesor.builder().nombre("Nombre2").colegio("Colegio2").materia("Lengua").numeroClase(55).build();

    guardar(teacherRepository, teacher);
    
    // Buscando registros cuyo nombre sea "Nombre1" sin importar los demas campos.
    var buscaNombre = Profesor.builder().nombre("Nombre1").build();
    guardar(teacherRepository, buscaNombre); 

    log.info("\n\nINSERTANDO ALUMNOS .... \n\n");
    var alumno = Alumno.builder().clase("1A").edad(12).nombre("Luis").build();
    guardar(alumnRepository, alumno);
    guardar(alumnRepository, alumno);

    alumno = Alumno.builder().clase("1A").edad(13).nombre("Luis").build();
    guardar(alumnRepository, alumno);
}
```

Como se puede apreciar, la función `iniciar` tiene la etiqueta **@PostConstruct**  lo que hará que **Spring Boot** ejecute esta función  una vez se haya inicializado la aplicación.

En ella, creamos diferentes instancias de **Profesor** y **Alumno** llamando a la función **guardar** donde esta lo interesante.

```java
void guardar(JpaRepository jpaRepository, Object entidad) {
    Example qbe = Example.of(entidad);
    var registros = jpaRepository.findAll(qbe);
    if (registros.size() > 0)
        log.warning("Ya existe Registro: " + entidad.toString());
    else {
        jpaRepository.save(entidad);
        log.info("Insertado registro: " + entidad.toString());
    }
}
```

Recibiremos un objeto que implemente la interfaz **JpaRepository** y un objeto que será donde estén los datos, es decir nuestra **entidad**.

A partir de la función estática `org.springframework.data.domain.Example.of()` (no confundir con `org.hibernate.criterion.Example`)  crearemos un objeto **Example** que contendrá los datos sobre los que realizaremos la búsqueda.

Una vez tenemos nuestro objeto **Example** creado, lo utilizamos para llamar a la función **findAll** del interface **JpaRepository** . 

Esto devolverá una **List**  con todos los registros cuyos valores sean igual a los valores mandados  en  **entidad**  siempre y cuando el valor del campo no sea **null**, en cuyo caso no se realizara la búsqueda por ese campo.

Es decir, en la primera llamada a `guardar` se ejecutará esta sentencia SQL:

```sql
 select
        profesor0_.id as id1_1_,
        profesor0_.colegio as colegio2_1_,
        profesor0_.materia as materia3_1_,
        profesor0_.nombre as nombre4_1_,
        profesor0_.numero_clase as numero_c5_1_ 
    from
        profesor profesor0_ 
    where
        profesor0_.nombre=? 
        and profesor0_.numero_clase=55 
        and profesor0_.materia=? 
        and profesor0_.colegio=?
```

 

Sin embargo cuando usamos la variable `buscaNombre` donde solo establecemos el valor para el campo, haciendolo igual 'Nombre1', se buscaran aquellos  registros cuyo nombre sea "Nombre1" sin importar los demás campos.  La *select* por tanto será  esta: 

```sql
   select
        profesor0_.id as id1_1_,
        profesor0_.colegio as colegio2_1_,
        profesor0_.materia as materia3_1_,
        profesor0_.nombre as nombre4_1_,
        profesor0_.numero_clase as numero_c5_1_ 
    from
        profesor profesor0_ 
    where
        profesor0_.nombre=?
```

Hay que tener en cuenta que la búsqueda se realizará sobre todos los campos que no tengan un valor igual **null**. Es por esto, que es importante que ninguno de los campos de la entidad este definido como un objeto primitivo (*int*, *double*, etc.). Si algún campo es de un tipo primitivo al realizar la búsqueda en caso de que no hayamos establecido el valor de ese campo, buscara aquel que tenga un valor igual **0**  lo cual, seguramente, no es lo que deseamos.

Creo que la salida mostrada al ejecutar la aplicación explica perfectamente la lógica:

```
INSERTANDO PROFESORES .... 


2021-01-23 18:14:23.493  INFO 5416 --- [           main] c.p.qbe.JpaQueryByExampleApplication     : Insertado registro: Profesor(id=1, nombre=Nombre1, materia=Matematicas, colegio=Colegio1, numeroClase=55)
2021-01-23 18:14:23.495  INFO 5416 --- [           main] c.p.qbe.JpaQueryByExampleApplication     : Insertado registro: Profesor(id=2, nombre=Nombre2, materia=Lengua, colegio=Colegio2, numeroClase=55)
2021-01-23 18:14:23.501  WARN 5416 --- [           main] c.p.qbe.JpaQueryByExampleApplication     : Ya existe Registro: Profesor(id=null, nombre=Nombre1, materia=null, colegio=null, numeroClase=null)
2021-01-23 18:14:23.501  INFO 5416 --- [           main] c.p.qbe.JpaQueryByExampleApplication     : 

INSERTANDO ALUMNOS .... 


2021-01-23 18:14:23.507  INFO 5416 --- [           main] c.p.qbe.JpaQueryByExampleApplication     : Insertado registro: Alumno(id=3, nombre=Luis, edad=12, clase=1A)
2021-01-23 18:14:23.509  WARN 5416 --- [           main] c.p.qbe.JpaQueryByExampleApplication     : Ya existe Registro: Alumno(id=3, nombre=Luis, edad=12, clase=1A)
2021-01-23 18:14:23.514  INFO 5416 --- [           main] c.p.qbe.JpaQueryByExampleApplication     : Insertado registro: Alumno(id=4, nombre=Luis, edad=13, clase=1A)
```



¡Y esto es todo!. Nos vemos en la próxima clase. 

