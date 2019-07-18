---
title: JPA con Lombok, simplificando.
author: airec69
type: post
date: 2018-08-24T10:53:55+00:00
url: /2018/08/24/jpa-con-lombok/
categories:
  - Sin categoría

---
En un [entrada anterior][1], explique que para usar JPA hay que tener nuestros objetos POJO definidos .  En esta entrada hablare de como mejorar la la definicion de nuestro objeto POJO, con la libreria Lombok.

Recordar que el código fuente de de este ejemplo esta en: <a href="https://github.com/chuchip/jdbc_jpa_tomcat" target="_blank" rel="noopener">https://github.com/chuchip/jdbc_jpa_tomcat</a>

Anteriormente teniamos definido nuestro objeto de esta manera:

<pre>@Entity
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
}</pre>

Ahora amos a hacer la clase mas facil, gracias a la Lombok. Para ello, a mi proyecto Maven (en el fichero pom.xml) añado la siguiente dependencia:

<pre>    &lt;dependency&gt;
            &lt;groupId&gt;org.projectlombok&lt;/groupId&gt;
            &lt;artifactId&gt;lombok&lt;/artifactId&gt;
            &lt;version&gt;1.18.2&lt;/version&gt;
            &lt;scope&gt;provided&lt;/scope&gt;
        &lt;/dependency&gt;</pre>

Teneis documentación de este proyecto en <a href="https://projectlombok.org/" target="_blank" rel="noopener">su pagina web</a>. De momento baste decir que gracias a la anotacion @Data nuestra clase **usuario** se queda así.

<pre>import lombok.Data;


@Data
@Entity
@Table(name = "usuario",
        uniqueConstraints = {
            @UniqueConstraint(columnNames = {"login"})})
public class Usuario implements Serializable {

    @Id
    String login;

    @Column
    String nombre;

   public Usuario(String login, String nombre) {
        this.login = login;
        this.nombre = nombre;
    }
}</pre>

Mucho mas limpia, ¿ a que si ?. Lombok se encargara de crear el constructor vacio, los setters y getters gracias la anotacion @Data.

Gracias a Lombok, a la hora de compilar Java crea funciones al vuelo, para que nosotros no tengamos que teclear tanto y no nos cansemos 😉

<a href="http://jnb.ociweb.com/jnb/jnbJan2010.html" target="_blank" rel="noopener">En esta página</a> (una vez más en un perfecto ingles) teneis las diferentes anotaciones que proporciona la libreria Lombok

&nbsp;

&nbsp;

 [1]: http://www.profesor-p.com/2018/08/21/conectando-con-postgresql-usando-jndi-y-spring-en-tomcat-parte-1/