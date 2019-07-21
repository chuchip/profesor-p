---
title: Usando Lambdas
author: airec69
type: post
date: 2018-08-23
url: /2018/08/23/usando-lambdas/
categories:
  - java
  - lambda
tags:
  - java
  - lambda

---
En esta breve entrada explicare como mejorar el [ejemplo anterior][1], con el uso de lambdas.
 <!--more--> 

Como siempre el código fuente de este ejemplo lo tenéis en: <a href="https://github.com/chuchip/jdbc_jpa_tomcat" target="_blank" rel="noopener">https://github.com/chuchip/jdbc_jpa_tomcat</a>

Las lambdas, son un mecanismo introducido en Java 8, pero que realmente en el mundo de la programación no es nuevo.

Gracias a las lambdas se puede entre otras cosas, evitar, en gran medida el uso de las clases auxiliares. Tenéis muchos manuales en la web, solo preguntarle a google, que lo sabe &#8216;casi&#8217; todo ;-), pero os aconsejo que le echéis un vistazo al siguiente enlace:

<a href="https://www.oracle.com/technetwork/es/articles/java/expresiones-lambda-api-stream-java-2633852-esa.html" target="_blank" rel="noopener">https://www.oracle.com/technetwork/es/articles/java/expresiones-lambda-api-stream-java-2633852-esa.html</a>

Yo, lo que os voy a dar un ejemplo de como usar lambda, para mejorar el uso de Swing Data JDBC. ¡¡Vamos a ello!!.

En nuestra clase anterior teníamos el siguiente código:

<pre>public class JdbcEjemplo {
.....
       
    public List&lt;Usuario&gt; findAllUsernames() {
        return jdbc.queryForObject(
                "select login,nombre from usuario ",
                new usuarioListaRowMapper() );
    }
   
   private  class usuarioListaRowMapper implements RowMapper&lt;List&lt;Usuario&gt;&gt; {

        @Override
        public List&lt;Usuario&gt; mapRow(ResultSet rs, int rowNum) throws SQLException {
            ArrayList&lt;Usuario&gt; listaUsuarios = new ArrayList();
            do
            {
                listaUsuarios.add(new Usuario(rs.getString("login"),
                                    rs.getString("nombre")));

            } while (rs.next());
            return listaUsuarios;
        }
    }
}</pre>

Ahora vamos a ver como usar una expresion lambda de tal manera que no tengamos que crear la clase **usuarioListaRowMapper** y, ademas nuestro código quede mucho mas limpio.

<pre>public List&lt;Usuario&gt; findAllUsernames() {
         return  jdbc.query(
                "select login,nombre from usuario ",
                (rs, rowNum) -&gt; new Usuario(rs.getString("login"),rs.getString("nombre"))
        );
}</pre>

¿ Sorprendido ?. Sí, gracias al uso de expresiones lambda hemos dejado nuestro código anterior en solo esas lineas. Y por supuesto no es necesario tener la clase **usuarioListaRowMapper **

Lo que hemos hecho es cambiar la llamada a la función **queryForObject** por la llamada a la función **query** que esta definida de tal manera

<pre>&lt;T&gt; List&lt;T&gt; query(String sql, RowMapper&lt;T&gt; rowMapper) throws DataAccessException;</pre>

Es decir espera una sentencia SQL y un clase que implemente la interfaz funcional **RowMapper** (que es una interfaz funcional esta explicado en la página anteriormente referenciada)

Pues bien, hemos creado esa clase a través de la expresión lambda, y nos hemos ahorrado un montón de código y dejado todo mucho más limpio.

Chicos, aprender a usar expresiones lambda. ¡¡ Mejoraran vuestro código !!

 [1]: /2018/08/22/acceso-a-base-de-datos-con-jdbc-spring/