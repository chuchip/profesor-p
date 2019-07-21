---
title: 'Aplicación CRUD usando REST en JavaEE  (2ª Parte)'
author: El Profe
type: post
weight: 2
date: 2018-10-07T05:49:37+00:00
url: /2018/10/07/aplicacion-rest-en-javaee-2a-parte/
categories:
  - glassfish
  - java
  - javaee
  - jpa
  - rest
tags:
  - glassfish
  - java
  - javaee
  - jpa
  - rest

---
Buenas otra vez.

En esta entrada continuo explicando la aplicación CRUD, utilizando peticiones REST, con JavaEE.

Recordar que tenéis el código fuente en <a href="https://github.com/chuchip/crudJavaEE" target="_blank" rel="noopener">https://github.com/chuchip/crudJavaEE</a>

En la anterior entrada, que podéis ver en  en <http://www.profesor-p.com/2018/10/06/aplicacion-rest-en-javaee/> habíamos visto como consultar los lenguajes disponibles. Ahora vamos a ver como añadir nuevos, modificarlos y borrarlos.

  * Funcion **create**

<pre>@POST
    @Consumes(MediaType.APPLICATION_JSON)
    public Response create(Locales locale) {

        if ( localeController.exists(locale.getCodigo())) {
            return Response.status(Response.Status.CONFLICT).build();
        }
        localeController.create(locale);
        URI location = UriBuilder.fromResource(LocaleResource.class)
                .path("/{locale}")
                .resolveTemplate("locale", locale.getCodigo())
                .build();
        return Response.created(location).build();
    }</pre>

Como podéis observar lo primero es especificar el tipo de petición HTTP que se va  a aceptar,  en este caso POST. Especificamos que queremos tratar solo peticiones tipo JSON y ya declaramos la función, que recibe un objeto tipo `Locales`

Después comprobamos si ese Locale no exista en la base de datos , devolviendo una respuesta tipo CONFLICT en caso de que ya exista.

En caso contrario, insertamos en la base de datos con la función `create` de la clase **LocaleController**, para despues devolver una respuesta tipo **created** (código 201) con la dirección donde se podría consultar el registro recién creado.

Y como siempre, una imagen mejor que cien palabras 😉

<img class="alignnone size-full wp-image-353" src="http://www.profesor-p.com/wp-content/uploads/2018/10/Captura-2.png" alt="" width="886" height="626" srcset="http://www.profesor-p.com/wp-content/uploads/2018/10/Captura-2.png 886w, http://www.profesor-p.com/wp-content/uploads/2018/10/Captura-2-300x212.png 300w, http://www.profesor-p.com/wp-content/uploads/2018/10/Captura-2-768x543.png 768w" sizes="(max-width: 886px) 100vw, 886px" />

En este pantallazo podéis ver como se hace la petición POST, mandando en el cuerpo el objeto JSON, donde se especifica el código y el nombre. La aplicación devuelve un 201 como estado y  Location vemos que tenemos &#8220;**http://localhost:8080/restExample/api/locale/es-BO&#8221;** que seria donde podemos consultar el registro recién creado.

  * Funcion **update**

<pre>@PUT
    @Path("/{codigo}")
    public Response update(@PathParam("locale") String codigo, Locales locales) {
        if (!Objects.equals(codigo, locales.getCodigo())) {
            throw new BadRequestException("Propiedad 'codigo' de Objeto Locale debe coincidir con el parámetro mandado.");
        }
        localeController.update(locales);
        return Response.ok().build();
    }
</pre>

Con esta función  se actualizara el registro mandado. Mirar como se solicita el código del país, ademas del objeto tipo locales . Esto es para comprobar que el objeto a modificar coincida con el código del país mandado. En caso contrario devolverá una excepción tipo **BadRequestException** la cual devolverá un código  400 (Bad Request).

Si todo va bien, devolvemos un código 200 (OK).

Aquí tenéis un pantallazo actualizando el registro anteriormente insertado.

<img class="size-full wp-image-354 aligncenter" src="http://www.profesor-p.com/wp-content/uploads/2018/10/Captura-3.png" alt="" width="886" height="626" srcset="http://www.profesor-p.com/wp-content/uploads/2018/10/Captura-3.png 886w, http://www.profesor-p.com/wp-content/uploads/2018/10/Captura-3-300x212.png 300w, http://www.profesor-p.com/wp-content/uploads/2018/10/Captura-3-768x543.png 768w" sizes="(max-width: 886px) 100vw, 886px" />

  * Funcion **delete**

<pre> @DELETE
    @Path("/{codigo}")
    public Response delete(@PathParam("codigo") String codigo) {
        logger.log(Level.INFO, "Borrando Locale: "+codigo);
        if ( ! localeController.exists(codigo)) {
            return Response.status(Response.Status.NOT_FOUND).build();
        }
        localeController.delete(codigo);
        return Response.ok().build();
    }</pre>

Esta función sera tratada cuando la petición HTTP sea del tipo DELETE ya que así esta especificado en la etiqueta **@DELETE** y deberá recibir un parámetro  con el código del país a borrar. Si el país en cuestión no existe devolverá un código error tipo NOT_FOUND y si es borrado correctamente, un 200 (OK).

Y con esto ya tendremos nuestra maravillosa aplicación CRUD,  a través de peticiones REST.

La lógica de acceso a la base de datos no es diferente a como se haría en Spring o con cualquier otro framework, así que si tenéis curiosidad de lo que hay detrás, podéis bajaros el código fuente de mi página de <a href="https://github.com/chuchip/crudJavaEE" target="_blank" rel="noopener">GitHub</a>.

¡¡ Hasta la próxima !!