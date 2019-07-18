---
title: Uso de perfiles en testing en Spring Boot
author: El Profe
type: post
date: 2019-03-01T13:50:57+00:00
url: /2019/03/01/uso-de-perfiles-en-testing-en-spring-boot/
categories:
  - java
  - profiles
  - spring boot
tags:
  - java
  - profiles
  - spring boot

---
Como comentaba [en el articulo anterior][1] gracias al uso de perfiles es fácil personalizar las aplicaciones en **Spring Boot**. Uno de los casos mas habituales del uso de perfiles es para testear la aplicación. Partiendo del mismo código fuente anterior, que os recuerdo esta en <https://github.com/chuchip/profilestest> vamos a ver el uso de los perfiles en testing.

Mirando el código de la clase ProfilesTestsTest.java

<pre>@Profile("test")<br />@SpringBootTest(webEnvironment= WebEnvironment.RANDOM_PORT)<br />@ActiveProfiles("test")<br />@RunWith(SpringRunner.class)<br />public class ProfilesTestsTest {<br />	<br />	@Value("${app.ID}")<br />	int id;<br />	<br />	@Autowired <br />	IRead read;<br />	<br />	@Autowired<br />	Environment env;<br />	<br />	@Test<br />	public void inicio() {<br />		String name=read.readRegistry(id);<br />		assertEquals(name,"test");<br />		String principal=env.getProperty("app.principal","NULL");<br />		assertNotEquals(principal,"profe"); <br />		String test=env.getProperty("app.test","NULL");<br />		assertEquals(test,"test");<br />	}<br /><br />}</pre>

nos debemos de fijar en la etiqueta **@ActiveProfiles(&#8220;test&#8221;)** con ella indicamos a **Spring** que active el perfil **test** con lo cual ahora solo se procesaran las clases que tengan la etiqueta **@Profile(&#8220;test&#8221;)** o no tengan ninguna etiqueta **@Profile.**

Como se puede ver en el directorio de _test_,  también incluimos la clase _ReadTestImpl.java_ que es exactamente igual que la existente en el paquete **com.profesorp.profiletest.impl.def** pero con la etiqueta **@Profile(&#8220;test&#8221;)**.

En el directorio **resources** de **test** (_profilestest/src/test/resources/_) encontramos el fichero _application.properties_ que usara **Spring Boot ** cuando se lance el test, pues como hemos comentado ese fichero se ejecuta siempre, sin importar el perfil activo.  Sin embargo no procesara el fichero _application.properties_ general.

Para demostrarlo en el fichero &#8220;_/profilestest/src/**main**/resources/application.properties&#8221;_ se ha incluido la linea

<pre>app.principal=principal</pre>

Y en &#8220;_/profilestest/src/**test**/resources/application.properties&#8221;_ están las siguientes lineas

<pre>app.ID=3<br />app.test=test</pre>

En el test podemos ver como no encuentra la variable de entorno **app.principal** pero si que encuentra la variable **app.test**

Y esto es todo por esta entrada. Espero que haya servidor para aclarar un poco más el uso de perfiles (_profiles_) en **Spring Boot**.

¡¡ Hasta la próxima !!

 [1]: /2019/02/28/perfiles-en-spring-boot/