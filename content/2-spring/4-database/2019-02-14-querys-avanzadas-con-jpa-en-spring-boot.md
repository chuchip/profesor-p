---
title: Querys avanzadas con JPA en Spring Boot
pre: "<b>o </b>"
author: El Profe
type: post
date: 2019-02-14T15:37:15+00:00
url: /2019/02/14/querys-avanzadas-con-jpa-en-spring-boot/
categories:
  - hibernate
  - java
  - jpa
  - spring boot
tags:
  - hibernate
  - java
  - jpa
  - spring boot

---
Hay veces en que las campos sobre los que restringir una consulta pueden variar en tiempo de ejecución. En ese caso si queremos usar JPA no podemos usar una sentencia <strong>@Query</strong> definida en nuestro repositorio pues no sabemos los campos sobre los que se aplicaran condiciones en la consulta. Además es bastante común que el usuario pueda elegir el criterio de búsqueda sobre un campo, deseando que el valor de un campo deba ser igual, mayor o menor, respecto al valor introducido .

En <strong>Spring Boot</strong> podemos dar una solución a este problema usando un la clase <strong>CriteriaBuilder</strong> de nuestro <strong>EntityManager</strong> . En esta entrada os mostrare como hacerlo fácilmente.

Para ello he creado un proyecto que he dejado en <a href="https://github.com/chuchip/CustomJpaQuery">https://github.com/chuchip/CustomJpaQuery</a>

En este programa podremos hacer una petición REST a la URL <a href="http://localhost:8080/get" rel="nofollow">http://localhost:8080/get</a> donde podremos pasar los siguientes parámetros, todos ellos opcionales:

- Identificador del cliente: <strong>idCustomer</strong>
- Nombre del Cliente: <strong>nameCustomer</strong>
- Dirección del cliente: <strong>addressCustomer</strong>
- Fecha creación del registro: <strong>createdDate</strong>. La fecha se debera mandar en formato español, es decir: &#8220;dd-MM-yyyy&#8221;. Por ejemplo: 31-01-2018.
- Condición del campo anterior: <strong>dateCondition</strong>. Tiene que ser una de estas tres cadenas de texto: <strong>&#8220;greater&#8221;,&#8221;less&#8221;, &#8220;equal&#8221;</strong> En caso de no poner ninguna condición o poner una condición no valida se usara <strong>greater</strong>
 
URLs de búsqueda podrían ser:

http://localhost:8080/get?createdDate=21-01-2018&dateCondition=equal
http://localhost:8080/get?createdDate=21-01-2018&dateCondition=greater  
  
http://localhost:8080/get?nameCustomer=Smith&createdDate=21-01-2018
 
El programa usa una base de datos <strong>H2</strong> para crear una tabla simple de clientes (<strong>customers</strong>) con los campos: <strong>id</strong>,<strong>name</strong>,<strong>address</strong>,<strong>email</strong> y <strong>created_date</strong>. Llena después la tabla con los datos que podemos ver en el fichero <strong>data.sql</strong>

Para realizar nuestra QUERY personalizada, en primer lugar, se crea un interface en <strong>CustomersRepository</strong> que extiende de <strong>JpaRepository</strong> . En este interface definimos la función <strong>getData</strong> como se ve en el siguiente código:
    
```
public interface CustomersRepository extends JpaRepository<CustomersEntity, Integer> {
	public List<CustomersEntity> getData(HashMap<String, Object> conditions);	
}
```

La función <strong>getData</strong> recibirá un <strong>HashMap</strong> donde iremos poniendo las condiciones de búsqueda. Así si queremos buscar los clientes cuyo código de cliente sea igual a 1, añadiremos una la llave &#8216;id&#8217; y el valor &#8216;1&#8243;
```
HashMap<String,Object> hm= new HashMap<>();
hm.put("id",1);
```  
 
Si deseamos que el nombre sea como &#8216;Smith&#8217;, añadiríamos este elemento al <strong>HashMap</strong>:
```
hm.put("name","Smith");
```
Y así sucesivamente con todos los campos o condiciones deseadas.
Una vez definido nuestro repositorio creamos una clase a la que obligatoriamente deberemos llamar <strong>CustomersRepositoryImpl</strong> es decir se debe llamar igual que nuestro interface del repositorio pero añadiendo la terminación <strong>impl</strong> (de implementación). En esta clase deberemos tener una función igual que la definida en el repositorio pues es la función que <strong>Spring Boot</strong> ejecutara cuando llamemos a la función definida en el interface.
Este es el código de la clase que permitirá personalizar nuestra query:

```
public class CustomersRepositoryImpl{
@PersistenceContext
private EntityManager entityManager;
	
public List<CustomersEntity> getData(HashMap<String, Object> conditions)
{
	CriteriaBuilder cb = entityManager.getCriteriaBuilder();
	CriteriaQuery<CustomersEntity> query= cb.createQuery(CustomersEntity.class);
	Root<CustomersEntity> root = query.from(CustomersEntity.class);
		
	List<Predicate> predicates = new ArrayList<>();
	conditions.forEach((field,value) ->
	{
		switch (field)
		{
			case "id":
				predicates.add(cb.equal (root.get(field), (Integer)value));
				break;
			case "name":
				predicates.add(cb.like(root.get(field),"%"+(String)value+"%"));
				break;
			case "address":
				predicates.add(cb.like(root.get(field),"%"+(String)value+"%"));
				break;
			case "created":
				String dateCondition=(String) conditions.get("dateCondition");					
				switch (dateCondition)
				{
					case GREATER_THAN:
						predicates.add(cb.greaterThan(root.<Date>get(field),(Date)value));
						break;
					case LESS_THAN:
						predicates.add(cb.lessThan(root.<Date>get(field),(Date)value));
						break;
					case EQUAL:
						predicates.add(cb.equal(root.<Date>get(field),(Date)value));
                        break;
				}
				break;
			}
		});
		query.select(root).where(predicates.toArray(new Predicate[predicates.size()]));
		return entityManager.createQuery(query).getResultList(); 		
	}
}
```
  
Como se ve, lo primero es inyectar una referencia al objeto <strong>EntityManager</strong> con la etiqueta <strong>@PersistenceContext</strong>. En la función sobre el <strong>EntityManager</strong> crearemos un objeto <strong>CriteriaBuilder</strong> y sobre este objeto creamos un <strong>CriteriaQuery</strong> donde iremos poniendo las diferentes condiciones de nuestra <strong>Query</strong>. Para poder buscar las columnas sobre las que queremos realizar la consulta necesitaremos un objeto <strong>Root</strong> , que crearemos a partir del anterior objeto <strong>CriteriaQuery</strong>

Ahora creamos una lista de objeto <strong>Predicate</strong> . En esa lista irán todos los <strong>Predicate</strong> que no son sino las condiciones de nuestra query.

Utilizando Lambdas y Streams para hacer el código mas limpio y sencillo, vamos recorriendo el <strong>HashMap</strong> y añadiendo a la lista de <strong>Predicates</strong> las condiciones definidas.

Partiendo del objeto <strong>CriteriaQuery</strong> se ira llamando a la función deseada según el criterio a aplicar. De esta manera, si queremos establecer como condición que un campo sea igual a un valor llamaremos a la función <strong>equal</strong>(), pasando como primer parámetro la <strong>Expresion</strong> que hace referencia al campo de la entidad, y después el valor deseado. El objeto <strong>Expresion</strong> se creara simplemente cogiendo del objeto <strong>Root</strong> anteriormente definido, el nombre de la columna sobre el que se establecerá la condición.

Si deseamos añadir una condición donde un campo sea <em>como</em> a un texto introducido se llamara a la función <strong>like</strong>(). En caso de que deseemos que el campo tenga un valor superior al introducido se usara <strong>greaterThan</strong>() y así sucesivamente.

Si el campo es de tipo <strong>Date</strong>, es necesario especificar el tipo de dato del campo como se muestra en el código <code>root.<Date>get(field)</code>, pues de otra manera no sabrá parsear correctamente la fecha.

Resaltar que el nombre del campo es el definido en nuestra entity que lógicamente no tiene porque ser el de la columna en la base de datos. Por ejemplo, el campo de fecha en la entity del proyecto de ejemplo, esta creada con las siguientes sentencias:
  
  
```
@Column(name="created_date")
@Temporal(TemporalType.DATE)
Date created;
```
De tal manera que en la base de datos la columna se llamara <code>created_date</code> pero todas las referencias a la entidad se harán a través del nombre <code>created</code> y es por ello que cuando busquemos el nombre del campo deberemos en <strong>Root</strong> deberemos buscar el campo <code>created</code> y no el campo <code>created_date</code>que no lo encontraría y nos daría error.

Una vez tenemos las condiciones de la consulta establecidas no tenemos más que preparar la consulta llamando a la función <strong>select</strong> a la que primero le indicaremos el <strong>Root</strong> con la entidad a consultar y después, las condiciones establecidas en el ArrayList de <strong>Predicate</strong>, el cual deberemos convertir previamente a un simple Array. Esto lo haremos con la sentencia: &#8216;<code>query.select(root).where(predicates.toArray(new Predicate[predicates.size()]));</code>

Ahora ejecutaremos la select y recogeremos los resultados en un objeto <strong>List</strong> con el comando <code>entityManager.createQuery(query).getResultList()</code>

Listo, ya tendremos nuestra Query personalizada funcionado. 🙂
Como siempre no dudéis en hacer cualquier consulta o mandar feedbacks. ¡¡ Hasta otra!!
