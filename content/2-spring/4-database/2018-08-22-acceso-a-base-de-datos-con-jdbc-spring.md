---
title: Acceso a Base de Datos con  Spring Data JDBC
pre: "<b>o </b>"
author: airec69
type: post
date: 2018-08-22T15:39:57+00:00
url: /2018/08/22/acceso-a-base-de-datos-con-jdbc-spring/
featured_image: /img/2018/08/springsource.jpg
categories:
  - java
  - jdbc
  - Sin categoria
  - spring
tags:
  - java
  - jdbc
  - spring

---
[En el anterior articulo](http://www.profesor-p.com/2018/08/21/conectando-con-postgresql-usando-jndi-y-spring-en-tomcat-parte-1/) explicaba como crear la conexion a la base de datos en un servidor de aplicaciones Tomcat . En este articulo explicare como acceder a esos datos a traves del paquete JDBC de Spring Data JDBC

El código fuente de este ejemplo esta en: https://github.com/chuchip/jdbc_jpa_tomcat

### Creando nuestro POJO y Repositorio
Ahora que ya tenemos nuestro acceso a la base de datos configurado y disponible, vamos a utilizarlo (por eso de que no se aburra 😉 )

Lo primero definimos nuestro POJO. Ya sabeis Plain Object Java Object, es decir Objeto Plano de Java, o en otras palabras ‘Clase Tonta donde almacenar y que no tiene nada o muy poco de lógica),

Este POJO que hará referencia a la tabla usuario, sera tal que así:

```
@Entity
@Table(name = "usuario",
        uniqueConstraints = {
            @UniqueConstraint(columnNames = {"login"})})
public class Usuario implements Serializable {

    @Id
    String login;

    @Column
    String nombre;

    public Usuario() {

    }

    public String getLogin() {
        return login;
    }

    public void setLogin(String login) {
        this.login = login;
    }

    public String getNombre() {
        return nombre;
    }

    public void setNombre(String nombre) {
        this.nombre = nombre;
    }

    public Usuario(String login, String nombre) {
        this.login = login;
        this.nombre = nombre;
    }
}
``` 
Creo que esto no hará falta explicarlo mucho. Si no entendéis lo que es una @Entitty y tal os recomiendo que leáis los siguientes manuales:

https://www.arquitecturajava.com/ejemplo-de-jpa/

https://www.oscarblancarteblog.com/2016/10/27/declarar-entidades-entity/

Una vez que ya tenemos una clase (una @Entity) donde almacenar los registros de nuestra tabla usuario, vamos a ver como trabajar usando el paquete JDBC de Spring. Para ser mas exactos, utilizando JDBC Templates de Spring.

En la clase JdbcEjemplo tenemos lo siguiente:

```
@Repository
public class JdbcEjemplo {

    @Autowired
    private JdbcOperations jdbc;

    @Bean
    public JdbcTemplate jdbcTemplate(DataSource dataSource) {
        return new JdbcTemplate(dataSource);
    }

    public Usuario findByUsername(String username) {
        return jdbc.queryForObject(
                "select login,nombre from usuario where login=?",
                new usuarioRowMapper(),
                username);
    }
    
    public List<Usuario> findAllUsernames() {
        return jdbc.queryForObject(
                "select login,nombre from usuario ",
                new usuarioListaRowMapper() );
    }
    private class usuarioRowMapper implements RowMapper<Usuario> {

        @Override
        public Usuario mapRow(ResultSet rs, int rowNum) throws SQLException {
            return new Usuario(
                    rs.getString("login"),
                    rs.getString("nombre"));
        }
    }
   private  class usuarioListaRowMapper implements RowMapper<List<Usuario>> {

        @Override
        public List<Usuario> mapRow(ResultSet rs, int rowNum) throws SQLException {
            ArrayList<Usuario> listaUsuarios = new ArrayList();
            do
            {
                listaUsuarios.add(new Usuario(rs.getString("login"),
                                    rs.getString("nombre")));

            } while (rs.next());
            return listaUsuarios;
        }
    }
}
```
Explico la clase, poco a poco. Lo primero es marcarla como repositorio, eso se hace con la anotación @Repository. Con esto conseguiremos que Spring cargue la clase y este disponible para otras clases a través del sistema de inyección de dependencias (realmente para nuestro ejemplo nos habría valido con marcarla como @Component)

Creamos la función jdbcTemplate(DataSource dataSource) , la cual nos devolverá un nuevo JdbcTemplate con el datasource que Spring ya tiene definido en su contexto. Como se puede ver, la función esta marcada con @Bean, para que la variable jdbc, que tenemos al principio de la clase, la llame y pueda asignarle un valor. JdbcOperations es el interface que usan las clases del paquete JDBC de Spring. La clase JdbcTemplate, por supuesto, lo implementa.

Nosotros usaremos la función findByUserName , para buscar el nombre del usuario (como nos gusta el ingles, madre mía 😉 ). Y esta función lo único que hará sera usando la variable global jdbc, invocando el método queryForObject. Este método usa los siguientes parámetros:

* La sentencia SQL a ejecutar, teniendo en cuenta que los parámetros a sustituir deben ser puestas con un ? , como si fuera un PreparedStatement, vamos.

* El objeto donde se van a guardar los resultados. Este objeto debe implementar el interface RowMapper. En nuestro ejemplo creamos la clase usuarioRowMapper donde definimos la función mapRow(ResultSet rs, int rowNum) la cual sera llamada por la función queryForObject, de tal manera que devuelva un objeto Usuario.

* Las variables a sustituir en la sentencia SQL. Tendra que haber tantas variables como ?  hemos puesto en nuestra sentencia SQL.

El caso es que cuando llamemos la función findByUserName nos devolverá una clase tipo Usuario o null si no encuentra nada.

En la siguiente función, llamada findAllUsernames buscaremos todos los usuarios que haya en la base de datos, por lo cual necesesitamos que devolver una lista de usuarios, es decir una List<Usuarios>.  Como se ve la llamada es casi igual que la de findByUserName , con la diferencia de que el RowMapper a devolver es usuarioListaRowMapper que como se ve en su función mapRow devuelve un objeto List que contiene Usuarios, es decir List<Usuarios>.

Obsérvese lo cómodo que es usar Templates JDBC. No tenemos que abrir Conexiones ni crear Statements ni nada, Spring lo hace todo por debajo. Simplemente ponemos la sentencia SQL y recibimos un objeto que contiene los resultados.

En el proximo articulo explicare como realizar busquedas en la base de datos a traves de JPA e Hibernate.