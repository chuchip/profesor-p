---
title: Personalizar salidas de códigos HTTP en Spring Boot
pre: "<b>o </b>"
author: El Profe
type: post
date: 2018-11-20T17:46:55+00:00
url: /2018/11/20/personalizar-codigo-http-en-spring-boot/
categories:
  - java
  - rest
  - spring boot
tags:
  - java
  - rest
  - spring boot

---
En esta articulo os voy a explicar como devolver errores HTTP personalizados. Para ello crearemos un proyecto llamado **httpErrorPersonalizado.** Teneis el código fuente de este proyecto en mi página de  <a href="https://github.com/chuchip/httpErrorPersonalizado" target="_blank" rel="noopener">GitHub</a>

Cuando realizamos una petición HTTP a un recurso en Spring Boot, es común que esa petición tenga que considerar la opción de devolver un error.

Es el caso típico de que realizamos una petición RESTful para solicitar un registro, pero ese registro no existe. En este caso lo normal es devolver un código HTTP tipo 404 (Not Found) lo cual se hace lanzando una excepción que hayamos anotado con la etiqueta `@ResponseStatus(HttpStatus.NOT_FOUND)` lo que ocurre es que el objeto JSON que acompaña a esa respuesta 404 en Spring Boot será con un formato definido de este tipo:

    
    {
        "timestamp": "2018-11-20T11:46:10.255+0000",
        "status": 404,
        "error": "Not Found",
        "message": "bean: 8 not Found",
        "path": "/get/8"
    }
    

Si nosotros queremos que la salida sea algo así como esto:

    {
        "timestamp": "2018-11-20T12:51:42.699+0000",
        "mensaje": "bean: 8 not Found",
        "detalles": "uri=/get/8",
        "httpCodeMessage": "Not Found"
    }
    

tendremos que poner una serie de clases a nuestro proyecto. Aquí os explico como 😉

El código fuente lo tenéis en mi [repositorio de GitHub][1]

Partiendo de una proyecto basico de Spring Boot, donde tenemos una simple objeto llamado **MiBean** con solo dos campos: **codigo** y **valor** que es el que devolveremos en las peticiones rest al recurso &#8220;**/get**&#8220;, de tal manera que una petición a: http://localhost:8080/get/1 nos devolvera un objeto JSON como este:

    {
        "codigo": 1,
        "valor": "valor uno"
    }
    

Si intentamos acceder a un elemento superior al 3 nos devolverá un error pues solo 3 registros disponibles.

Aquí pong la clase _ErrorResource_ que procesa las peticiones al recurso &#8220;**/get**&#8220;


```
public class ErrorResource {

  @Autowired
  MiBeanService service;
  
  @GetMapping("/get/{id}")
  public MiBean getBean(@PathVariable int id) {
    MiBean bean = null;
    try 
    {
       bean = service.getBean(id);
    } catch (NoSuchElementException k)
    {
      throw new BeanNotFoundException("bean: "+id+ " not Found" );
    }
    return bean;
  }
}
```


Como se ve en **getBean()** se llama a la función **_getBean(int id)_** de la clase **MiBeanService**, la cual pego a continuación

    @Component
    public class MiBeanService {
      private static  List<MiBean> miBeans = new ArrayList<>();
    
      static {
        miBeans.add(new MiBean(1, "valor uno"));
        miBeans.add(new MiBean(2, "valor dos"));
        miBeans.add(new MiBean(3, "valor tres"));
      }
      
      public MiBean getBean(int id) {
        MiBean miBean =
            miBeans.stream()
             .filter(t -> t.getCodigo()==id)
             .findFirst()
             .get();
            
        return miBean;
      }
    
    }
    

Observe que la función **getBean(int id)** lanzara una excepción tipo **NoSuchElementException** si no encuentra el código en la _List_ **miBeans** . Esta excepción será capturada en el controlador el cual lanzara una excepción tipo **BeanNotFoundException**

La clase **BeanNotFoundException** es la siguiente:

    @ResponseStatus(HttpStatus.NOT_FOUND)
    public class BeanNotFoundException  extends RuntimeException {
      public BeanNotFoundException(String message) {
        super(message);
      }
    }
    
    

Una simple clase que extiende **RuntimeException** y que esta anotada con al etiqueta `@ResponseStatus(HttpStatus.NOT_FOUND)`con lo cual al ser lanzada devolvera un código HTTP 404 (Not Found).

Si dejáramos así el proyecto al pedir un código superior a 3, seria esta:

![captura-1](/img/2018/11/Captura1.png")

pero como hemos dicho queremos que el mensaje de error sea personalizado.

Para ello vamos a crear una nueva clase donde definiremos los campos de nuestro mensaje de error. Esta clase el proyecto es **ExceptionResponse** la cual es un simple pojo como se puede ver en el código que adjunto:

    public class ExceptionResponse {
      private Date timestamp;
      private String mensaje;
      private String detalles;
      private String httpCodeMessage;
    
      public ExceptionResponse(Date timestamp, String message, String details,String httpCodeMessage) {
        super();
        this.timestamp = timestamp;
        this.mensaje = message;
        this.detalles = details;
        this.httpCodeMessage=httpCodeMessage;
      }
    
      public String getHttpCodeMessage() {
        return httpCodeMessage;
      }
    
      public Date getTimestamp() {
        return timestamp;
      }
    
      public String getMensaje() {
        return mensaje;
      }
    
      public String getDetalles() {
        return detalles;
      }
    
    }
    

Ahora se definirá la clase que indicara a Spring que objeto JSON debe devolver en caso de que se produzca lance una excepción del tipo **BeanNotFoundException** . Esa clase es: **CustomizedResponseEntityExceptionHandler** la cual adjunto a continuación:

    //@ControllerAdvice
    // @RestController
    @RestControllerAdvice
    public class CustomizedResponseEntityExceptionHandler extends ResponseEntityExceptionHandler {
    
      @ExceptionHandler(BeanNotFoundException.class)
      public final ResponseEntity<ExceptionResponse> handleNotFoundException(BeanNotFoundException ex, WebRequest request) {
        ExceptionResponse exceptionResponse = new ExceptionResponse(new Date(), ex.getMessage(),
            request.getDescription(false),HttpStatus.NOT_ACCEPTABLE.getReasonPhrase());
        return new ResponseEntity<ExceptionResponse>(exceptionResponse, HttpStatus.NOT_ACCEPTABLE);
      }
    
    }
    

Esta clase debe heredar de _ResponseEntityExceptionHandler_ la cual ya tratara las excepciones más comunes.

La deberemos anotarla con las etiquetas `@ControllerAdvice` y `@RestController`o como me sugirió Marcelo Martins  en [DZone][2] sustistuir esas dos por la etiqueta: `@RestControllerAdvice`

`@ControllerAdvice` es una etiqueta derivada de `@Component` que se usara para clases que traten excepciones. Al tener la clase la etiqueta `@RestContoller` tratara las excepciones lanzadas en los controladores de peticiones REST.

Y crearemos la función donde especificar el objeto a utilizar cuando se produzca un tipo de excepción.

Así, en el ejemplo, hemos definido que cuando se se lance la excepción `BeanNotFoundException` será devuelto un objeto `ExceptionResponse`. Esto se hace creando un objeto `ResponseEntity`convenientemente iniciado.

Es importante observar que también definimos el código HTTP devuelto. En este caso devolveremos el código 406, en vez del 404. De hecho en nuestro ejemplo podríamos quitar la etiqueta `@ResponseStatus(HttpStatus.NOT_FOUND)`a la clase **BeanNotFoundException** y todo seguiría funcionando igual.

Y así tendremos una salida personalizada como se ve en la siguiente imagen:

![Captura-2](/img/2018/11/Captura2.png")

Y esto es todo por hoy. ¡¡ Nos vemos en la próxima clase !!

 [1]: https://github.com/chuchip/httpErrorPersonalizado.git
 [2]: https://dzone.com/articles/customize-error-responses-in-spring-boot