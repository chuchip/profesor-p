---
title: Lambdas en JDBC Data
pre: "<b>o </b>"
author: El Profe
weight: 100
type: post
date: 2018-09-04T11:04:46+00:00
url: /2018/08/22/acceso-a-base-de-datos-con-jdbc-spring/
categories:
  - java
  - jdbc
  - lambda

---

## Expresiones Lamba en  Spring JDBC Data. Mejorando la explicación.

<a href="/2018/08/23/usando-lambdas/" rel="noopener">En una entrada anterior</a>, puse un ejemplo de como usar expresiones Lambas, como me parece que es un tema interesante, este de la programación funcional, voy a insistir en este tema.

Una cosa  muy común en Java es  tener que pesarle como argumento a una función externa, una objeto que implemente una función donde nosotros pondremos el código a ejecutar en nuestra  aplicación.

Un ejemplo practico es en el uso de  la clase <span class="pl-smi"><a href="https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/jdbc/core/JdbcOperations.html" target="_blank" rel="noopener">JdbcOperations</a>, que es utilizada por Spring para poder realizar operaciones JDBC, la cual tiene la función: </span>

```
T query(java.lang.String sql, java.lang.Object[] args, ResultSetExtractor rse)   throws DataAccessException
```

Lo que hay que entender es que como funciona realmente esta función **query **<span class="pl-smi">. Esta función necesita tres parametros:</span>

  * <span class="pl-smi">sentencia SQL  a ejecutar, </span>
  * Argumentos de la sentencia SQL
  * Clase ResultSetExtractor donde nuestro programa hará algo con los resultados obtenidos.

Así, voy a suponer que queremos realizar una consulta sobre una tabla, buscando el nombre de un usuario, a través de su identificador. La tabla **usuarios, **tendría solo estos dos campos:

  * **identificador**, que seria un varchar de 15 caracteres (un texto para los que no conocen SQL)
  * **nombre**, que seria otro de varchar  de 50 caracteres (otro texto, vamos).

Es decir queremos buscar, el nombre de un usuario, sabiendo su identificador. La sentencia SQL para realizar esta búsqueda seria

<pre>select nombre from usuarios where identificador = ?</pre>

La interrogación final hace referencia al parámetro que pasaremos, es decir el identificador del usuario.

Por lo tanto, en nuestro programa pondremos una sentencia, como la siguiente:

```
jdbc.query("select nombre from usuarios where identificador = ?", new[] { "usuario1" },  OBJETO_RESULTSETEXTRACTOR)
```

El parámetro **<span class="pl-k">new</span> <span class="pl-smi">Object</span>[] { &#8220;usuario1&#8221; } **simplemente crea un array de Objetos con los diferentes parámetros que necesitara nuestra sentencia SQL. En este caso solo necesita un parámetro que es el identificador del usuario. Vamos a buscar el usuario que tiene como identificador **usuario1.**

Y nos queda por definir el objeto del tipo **ResultSetExtractor**, donde interactuaremos con los resultados de nuestra sentencia SQL.

Ese tipo de objeto esta definido en el interface [ResultSetExtractor][1] el cual solo tiene una función, que detallo a continuación:

```T mapRow(java.sql.ResultSet rs,  int rowNum)            throws java.sql.SQLException```

Antes de Java 1.8, deberíamos crear una clase que implemente el interfaz [ResultSetExtractor][1]. Esa clase seria algo así como:

<pre>class miResultSetExtractor implements <a title="interface in org.springframework.jdbc.core" href="https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/jdbc/core/ResultSetExtractor.html">ResultSetExtractor</a>
{
   @Override
   public T mapRow(java.sql.ResultSet rs, int rowNum) throws java.sql.SQLException { 
          System.out.println("El nombre del Usuario es: "+rs.getString("nombre")); // <strong>CODIGO A EJECUTAR</strong>
    }
}</pre>

Y ejecutaríamos la llamada a la función **query** de esta manera:

<pre>jdbc.query("select nombre from usuarios where identificador = ?", <span class="pl-k">new</span> <span class="pl-smi">Object</span>[] { "usuario1" },new miResultSetExtractor ());</pre>

También podríamos crear una clase abstracta, a la hora de llamar a la función **query, ** con una sentencia como esta:

<pre>jdbc.query("select nombre from usuarios where identificador = ?", <span class="pl-k">new</span> <span class="pl-smi">Object</span>[] { "usuario1" }, new ResultSetExtractor()
{
      @Override
      public T mapRow(java.sql.ResultSet rs, int rowNum) throws java.sql.SQLException { 
          System.out.println("El nombre del Usuario es: "+rs.getString("nombre")); // <strong>CODIGO A EJECUTAR</strong>
      }
});</pre>

El caso es que tenemos que poner un montón de código auxiliar  cuando nosotros solo quisiéramos poner nuestro System.out.println.

Y ahora es cuando viene al rescate las expresiones Lamba.

<pre>jdbc.query("select nombre from usuarios where identificador = ?", <span class="pl-k">new</span> <span class="pl-smi">Object</span>[] { "usuario1" },
             (param1, param2) -&gt;  System.out.println("El nombre del Usuario es: "+param1.getString("nombre"));</pre>

La expresión lambda se compone de dos partes. Los parámetros a mandar y el código a ejecutar.

Así, nosotros mandaremos dos parametros: **param1 **y **param2**. (le podriamos haber puesto los nombres que quisieramos)

Como se ve, no hay que especificar de que tipo son esos parámetros, Java, se encargara de eso. Te estarás preguntado, pero, ¿ como funciona esto ?.

Bueno, en primer lugar, Java, cuando compile nuestro código buscara que que objeto debe crear, dependiendo del parámetro esperado por la función **jdbc.query(). **Vera que espera un objeto que implemente el interface **ResultSetExtractor **y comprobara que ese interface solo tiene una función, por lo cual, sabrá fácilmente que parámetros se tipo de objetos se deben pasar como objetos.

Es por eso que solo se pueden aplicar este tipo de expresiones lambda cuando el interface implemente una única función.

Una vez que Java, sabe que objetos debe pasar, el sustituirá nuestras variables **param1 **y **param2 **y las usara en el cuerpo de nuestra expresión Lambda. Por ello ahora nuestro código: **System.out.println(&#8220;El nombre del Usuario es: &#8220;+param1.getString(&#8220;nombre&#8221;))  **cogera la variable **param1 **y la usara sabiendo que es un objeto tipo ResultSet. Observar que aunque la variable **param2**, no la usamos en nuestro código, debemos definirla igualmente.

Espero haberme explicado bien, y no dudéis en preguntar o seguirme en mi <a href="https://twitter.com/chuchip" target="_blank" rel="noopener">cuenta de twitter</a>.

¡ Hasta la próxima !!

 [1]: https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/jdbc/core/ResultSetExtractor.html "interface in org.springframework.jdbc.core"