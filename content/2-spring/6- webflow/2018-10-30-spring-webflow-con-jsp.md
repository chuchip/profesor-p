---
title: Spring WebFlow con JSP
pre: "<b>o </b>"
weight: 20
author: El Profe
type: post
date: 2018-10-30T17:12:49+00:00
url: /2018/10/30/spring-webflow-con-jsp/
categories:
  - bootstrap
  - java
  - mvc
  - seguridad
  - spring
  - tomcat
  - webflow
tags:
  - bootstrap
  - java
  - mvc
  - seguridad
  - spring
  - tomcat
  - webflow

---
En el articulo anterior <http://www.profesor-p.com/2018/10/29/spring-webflow-con-jsp-configuracion/> explicaba como configurar el programa para que Spring WebFlow funcionara. En este articulo explicare como hacer el flujo en si.

La página principal del programa no esta dentro de ningún flujo y sus peticiones son respondidas por Spring MVC, en la clase **MyController,** la cual podemos encontrar en el paquete _profesorp.webflow.controller._ Esta clase anotada con la etiqueta @**Controller** responde en la función **`indice1`** a las peticiones de los recursos &#8220;/&#8221; e &#8220;index&#8221;

Aquí podéis ver el código:

<pre>@RequestMapping(value = {"/", "index"})
    public ModelAndView indice1(ModelAndView mod) {
        logger.info("request index");
        mod.addObject("cliente", cliente);
        mod.setViewName("index");
        return mod;
    }</pre>

Como veis devuelve un objeto **ModelAndView **donde guarda el objeto **cliente**, la cual es una @**entity** de la tabla **Clientes**, que ya vimos en la entrada anterior. La vista devuelta es &#8220;**index.jsp**&#8221; que esta en el directorio **webapp/WEB-INF/jsp/**

En &#8220;**index.jsp**&#8221; encontramos la etiqueta **<sec:authorize access=&#8221;isAuthenticated()&#8221;>** que esta dentro del paquete **spring-security-taglibs**. Si el usuario no esta autentificado mostrara la pantalla de login, en caso contrario saludara al usuario  y permitirá realizar el traspaso de dinero.

Al usar el  paquete _security,_ y para evitar ataques CSRF,  Spring solo permite realizar peticiones tipo POST,  al recurso /logout. Por ello uso la librería **JQuery** con la cual creo esa petición en la función de javascript `function post(path, parameters)`

En todas las peticiones tipo POST debemos incluir la variable **_csrf.parameterName**  con el valor **_csrf.token** para evitar estos mismos ataques CSRF. Si no  las ponemos Spring Security no aceptara nuestras peticiones, pues entenderá que es un posible ataque.

<pre>&lt;input type="hidden" name="${_csrf.parameterName}" value="${_csrf.token}"/&gt;</pre>

Esta es la pantalla de entrada , solicitando las credenciales.

![](/img/2018/10/Captura-23.png")

Y está, la pantalla de entrada , una vez el usuario esta registrado.

![](/img/2018/10/Captura-24.png)

Si pulsamos el enlace &#8220;_Transferencia_&#8221; iremos al recurso &#8220;**traspaso**&#8221; que estará ubicado en la URL **http://localhost:8080/webflow/traspaso**. Este recurso es tratado por Spring Webflow, por lo cual se cargara el fichero  **WEB-INF/flows/traspaso/traspaso.xml** pues así lo definimos en la clase **WebFlowConfig** anteriormente vista.

Todo fichero XML que defina un webflow deberá empezar con las siguientes lineas.

<pre>&lt;?xml version="1.0" encoding="UTF-8"?&gt;
&lt;flow xmlns="http://www.springframework.org/schema/webflow"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xsi:schemaLocation="http://www.springframework.org/schema/webflow
                          http://www.springframework.org/schema/webflow/spring-webflow.xsd"&gt;
    
  </pre>

A continuación defino la variable **traspasoBean** con una nueva instancia del objeto **profesorp.webflow.controller.TraspasoBean.** 

<pre>&lt;var name="traspasoBean" class="profesorp.webflow.controller.TraspasoBean"/&gt;</pre>

Esta sentencia seria el equivalente al código java `<strong>traspasoBean  = new profesorp.webflow.controller.TraspasoBean()</strong>`

Seguidamente. indico que al cargar el flujo de trabajo debe evaluar la expresión **logicaService.getCliente()** cuyo resultado debe guardar en la variable cliente, la cual sera  de ámbito flow (_flowScope.cliente)._ Una explicación de los ámbitos la tenéis en <a href="https://docs.spring.io/spring-webflow/docs/current/reference/html/el.html#el-variable-flowScope" target="_blank" rel="noopener">la pagina oficial de Spring</a> (en ingles me temo). Baste decir, de momento que una variable definida dentro del ámbito flowscope es accesible en todas las paginas que estén definidas en nuestro flujo.

<pre>&lt;on-start&gt;       
        &lt;evaluate expression="logicaService.getCliente()" result="flowScope.cliente" /&gt;              
  &lt;/on-start&gt;</pre>

Tener en cuenta que para poder acceder al objeto **logicaService**  lo hemos tenido que definir como un _Bean_ en nuestro programa. En este ejemplo lo tenéis en la clase **LogicaService,** que muestro a continuación.

<pre>package profesorp.webflow.services;

@Service
@SessionScope
public class LogicaService {
....
Clientes cliente;
...
 public Clientes getCliente()
  {
         return cliente;
  }
}</pre>

A continuación indico la primera vista que se debe cargar, que sera el fichero **cuentaOrigen.jsp**

<pre>&lt;view-state id="cuentaOrigen" model="traspasoBean"&gt;        
        &lt;on-render&gt;
            &lt;evaluate expression="logicaService.getCuentasByCliente()" result="flowScope.cuentas" /&gt;
        &lt;/on-render&gt;
        &lt;transition on="activate" to="importe"/&gt;       
    &lt;/view-state&gt;</pre>

Con el parámetro **model** especifico  las variables que reciba de la vista, deberán ser puestas en el objeto **traspasoBean.** 

Como en la vista  definimos la variable **cuentaOrigen** (en un campo tipo select),y  el bean **profesorp.webflow.controller.TraspasoBean** tiene a su vez la variable **cuentaOrigen** definida, se llamara a la función **setCuentaOrigen**() de esa clase, con el valor introducido , por el usuario en la vista.

Si tuviéramos más variables en el formulario que coincidieran con sus correspondientes setters, también serian llamadas.

La vista, que detallo a continuación, es un formulario bastante simple donde se pide el numero de cuenta origen:

<pre>&lt;form method="post" action="<strong>${flowExecutionUrl}</strong>"&gt;
                    &lt;input type="hidden" name="${_csrf.parameterName}" value="${_csrf.token}"/&gt;
                    &lt;input type="hidden" name="<strong>_eventId</strong>" value="<strong>activate</strong>"&gt;

                    &lt;div class="form-group"&gt;                   
                        &lt;label class="etiqueta" for="<strong>Cuenta</strong>"&gt;Elija Cuenta Origen: &lt;/label&gt;
                        &lt;select name="<strong>cuentaOrigen</strong>"&gt;
                            &lt;c:forEach items="${<strong>cuentas</strong>}" var="item"&gt;
                                &lt;option value="${item}"&gt;${item}&lt;/option&gt;
                            &lt;/c:forEach&gt;         
                        &lt;/select&gt;
                    &lt;/div&gt;
                    &lt;div class="col "&gt;
                        &lt;input class="btn btn-primary" type="submit" value="Siguiente" /&gt;
                    &lt;/div&gt;
                 <strong>  &lt;a class="nav-link" href="${flowExecutionUrl}&_eventId=cancel"&gt;Cancelar&lt;/a&gt;</strong>
        &lt;/form&gt;</pre>

Así se ve la vista en el navegador

![](/img/2018/10/Captura-25.png(


Lo primero es ver como la URL a llamar por el formulario es el resultado de la variable **${flowExecutionUrl}**. Esta variable es puesta automáticamente por _Spring WebFlow_ y apuntara  a la dirección donde continuara el flujo actual.

La variable &#8220;**_eventId**&#8221;  es la que _Spring WebFlow_ buscara para decidir que acción llevar a cabo. En este caso le asigno el valor &#8220;**activate&#8221;** que coincide con  el valor de la etiqueta **transition** `<transition on="activate" to="importe"/> `

Con esto lo que hacemos es configurar que cuando la variable **_eventId** tenga el valor &#8220;**activate&#8221;** el flujo vaya al ID **importe.** Por supuesto, el ID deberá existir en nuestro flujo.

Aclarar que &#8220;**activate&#8221;** es un literal libre, que igual podria ser &#8216;**MI_SALIDA**&#8216; o &#8216;**SEGUIR**&#8216;.

Lo normal, en una vista, es que pueda devolver diferentes valores. Así en  la ultima linea donde declaramos un enlace (<a href&#8230;&#8221;>) vemos como se llama a **${flowExecutionUrl} **pasando el parámetro **_eventId=cancel** . En el caso de que pulsemos ese enlace lo que se ejecutara sera el siguiente código que esta al final del fichero **traspaso.xml**

<pre>&lt;global-transitions&gt;
        &lt;transition on="cancel" to="cancel" /&gt;
  &lt;/global-transitions&gt;</pre>

Esto es así porque hemos definido que cualquier evento tipo &#8220;**cancel**&#8221; dentro del flujo, llame al ID **cancel** del que luego hablare.

<pre>&lt;evaluate expression="logicaService.getCuentasByCliente()" result="flowScope.cuentas" /&gt;</pre>

La etiqueta **<on-render>** hace que lo que haya dentro se ejecute justo antes de renderizar la vista. En este caso llamara a la función `logicaService.getCuentasByCliente()`y el valor devuelto lo almacenara en `flowScope.cuentas`

<pre>public Iterable&lt;String&gt; getCuentasByCliente()
   {
       return cuentasClientesRepository.findCuentasByCliente(cliente.getId());
   }</pre>

Este objeto **cuentas** que es un Array de strings que se usa en la vista para crear el desplegable  con las cuentas de origen disponibles, en el código `<c:forEach items.....>`

Lo siguiente define la vista con ID **importe**, que es donde ira el usuario cuando pulse el botón siguiente, como hemos definido con las etiquetas `<transition on="activate" to="importe"/>` anteriores.

<pre>&lt;view-state id="importe" model="traspasoBean"&gt;    
        &lt;on-render&gt;
            &lt;evaluate expression="false" result="traspasoBean.puestoPeriodico" /&gt;
        &lt;/on-render&gt;  
        &lt;transition on="salir" to="comprobarImporte"&gt;
            &lt;evaluate expression="logicaService.hasCredit(traspasoBean)" /&gt;
        &lt;/transition&gt;
    &lt;/view-state&gt;</pre>

Como en la anterior vista, los valores introducidos en el formulario serán introducidos en **traspasoBean,** al incluirse la etiqueta **model.**

Antes de mostrar la vista se llama a la función **traspasoBean.puestoPeriodico()** con el valor _false_ debido a la etiqueta **on-render.**

Si el valor de la variable &#8220;**_eventId**&#8221; es &#8220;**salir&#8221;** el flujo sera dirigido al ID **comprobarImporte**, pero antes de ir se llamara a la función `logicaService.hasCredit(traspasoBean)`. Si esa función devuelve _true_ se realizara el salto a **comprobarImporte** en caso contrario se volvera a mostrar la vista actual, es decir **importe.**

La vista **importe** esta definida en el fichero **importe.jsp** del cual pongo un extracto a continuación

<pre>&lt;form method="post" action="${flowExecutionUrl}"&gt;
                    &lt;input type="hidden" name="${_csrf.parameterName}" value="${_csrf.token}"/&gt;
                    &lt;input type="hidden" name="_eventId" value="salir"&gt;
                    &lt;div class="form-group"&gt;                 
                        &lt;label class="etiqueta" for="CuentaOrigen"&gt;Cuenta Origen &lt;/label&gt;
                        &lt;input type="text" name="cuentaOrigen" value="${traspasoBean.cuentaOrigen}" disabled &gt;
                    &lt;/div&gt;
                    &lt;div class="form-group"&gt;
                        &lt;label class="etiqueta"  for="Importe"&gt;Introduzca Importe &lt;/label&gt;
                        &lt;input type="text" class="importe" name="importe" value="${traspasoBean.importe}"  placeholder="Importe Traspaso" required&gt;    
                        &lt;c:if test="${traspasoBean.getMsgImporte()!=null}"&gt;
                            &lt;div class="alert alert-warning"&gt;
                                &lt;strong&gt;Atencion!&lt;/strong&gt; ${traspasoBean.getMsgImporte()}
                            &lt;/div&gt;
                        &lt;/c:if&gt;                    
                    &lt;/div&gt;
                    &lt;div class="form-group"&gt;
                        &lt;label class="etiqueta" for="CuentaDestino"&gt;Cuenta Destino &lt;/label&gt;
                        &lt;input type="text" name="cuentaFinal" value="${traspasoBean.cuentaFinal}" &gt;
                    &lt;/div&gt;
                    &lt;div class="form-group"&gt;
                        &lt;label class="etiqueta"  for="Periodico"&gt;Traspaso Periodico&lt;/label&gt;
                        &lt;input type="checkbox" name="periodico" value="true"
                               &lt;c:if test="${traspasoBean.periodico}"&gt;
                                   checked 
                               &lt;/c:if&gt;
                               &gt;
                    &lt;/div&gt;
                    &lt;div class="col"&gt;
                        &lt;input class="btn btn-primary" type="submit" value="Siguiente" /&gt;&nbsp;
                    &lt;/div&gt;
               &lt;/form&gt;</pre>

Resaltar que como la variable **periodico** esta en un campo **checkbox** solo sera mandada cuando el usuario seleccione esa opción, por lo cual si no es marcada, la funcion setPeriodico() no sera llamada, pero no se pondrá a false la correspondiente variable en el Bean. Es por ello que antes de mostrar la vista, con la etiqueta `<on-render>` se pone a false una variable auxiliar que utilizara el programa para saber que no se ha seleccionado la opción &#8220;_Traspaso Periódico_&#8220;

Una imagen de como se muestra la vista en el navegador.

![](/img//2018/10/Captura-26.png)

El id **comprobarImporte** se define a continuación,

<pre>&lt;decision-state id="comprobarImporte"&gt;
        &lt;if test="logicaService.checkImporte(traspasoBean)"
            then="periocidad"
            else="importe"/&gt;
    &lt;/decision-state&gt;</pre>

En este estado que es tipo decisión, se llama a la función **checkImporte**(**traspasoBean**) del objeto **logicaService** en caso de que devuelva true se ira al ID **periocidad**, en caso contrario se volverá al ID **importe .** Esta función comprueba que el cliente tenga suficiente dinero en la cuenta.

El ID **periocidad** es otro estado tipo decisión que dependiendo del resultado de la función **isPeriodico**, la cual comprueba si se ha selecionado la opción **&#8220;Traspaso Periodico&#8221;** en la vista, ira a **periodico** o a **confirmar**

<pre>&lt;decision-state id="periocidad"&gt;
        &lt;if test="logicaService.isPeriodico(traspasoBean)"
            then="periodico"
            else="confirmar"/&gt;
    &lt;/decision-state&gt;
   &lt;subflow-state id="periodico" subflow="traspaso_time"&gt;
        &lt;input name="traspasoBean" /&gt;
        &lt;transition on="salir" to="confirmar"&gt;            
        &lt;/transition&gt;
    &lt;/subflow-state&gt;</pre>

El ID **periodico** es una llamada al flujo **traspaso_time,** al cual se saltara pasándole el objeto **traspasoBean** debido a la etiqueta **input.**

Si ese flujo sale con el ID **salir** se saltara al ID **confirmar**

A continuación detallo el contenido del flujo **traspaso_time** que esta definido en el fichero **traspaso_time.xml**

<pre>&lt;?xml version="1.0" encoding="UTF-8"?&gt;
&lt;flow xmlns="http://www.springframework.org/schema/webflow"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xsi:schemaLocation="http://www.springframework.org/schema/webflow
                          http://www.springframework.org/schema/webflow/spring-webflow.xsd"&gt;
    
    &lt;input name="traspasoBean" required="true" /&gt;  
    
    &lt;view-state id="tiempo" model="traspasoBean"&gt;         
        &lt;transition on="salir" to="salir"/&gt;       
        &lt;transition on="cancelar" to="cancel" /&gt;        
    &lt;/view-state&gt;
    &lt;end-state id="salir" /&gt;
    &lt;end-state id="cancel" /&gt;
&lt;/flow&gt;</pre>

Lo primero que declaramos es que debemos recibir un objeto **traspasoBean** . Si no lo recibieramos el programa fallara.

Es decir el flujo lo podríamos llamar con la URL: **http://localhost:8080/webflow/traspaso_time** pero si lo hacemos al no recibir el objeto  **traspasoBean ** fallara.

![](/img//2018/10/Captura-27-1024x261.png)

En el flujo mostramos la vista **tiempo** y si el _<span style="text-decoration: underline;">event_id</span>_ es **salir **se saltara al ID **salir.**

Este ID es tipo **end-state** por lo cual  se volverá  al anterior  flujo con el ID **salir.** En el flujo **traspaso** entonces  se saltara al ID **confirmar**, como esta definido por la etiqueta: `<transition on="salir" to="confirmar"> `

Si el _event_id_ es **cancelar** también volverá al flujo anterior, pero  a través del ID **cancel** y por lo tanto se saltara al ID **cancel** , como se definió con las etiquetas anteriormente mostradas.

<pre>&lt;global-transitions&gt;
        &lt;transition on="cancel" to="cancel" /&gt;
    &lt;/global-transitions&gt;</pre>

La vista **tiempo** se muestra así en el navegador.

![](/img//2018/10/Captura-28.png)

Por último la vista **confirmar,** muestra todos los datos introducidos y solicita la confirmación. En caso de darla se ira al ID **salir** con lo cual saldremos del flujo redirigiendonos al recurso **index** pasando la variable **transferencia ** con el valor &#8220;1&#8221;. En caso de que en algún momento hayamos cancelado la transferencia se saldrá del flujo dirigiéndonos a **index** pero la variable **transferencia** tendrá el valor &#8220;0&#8221;

<pre>&lt;view-state id="confirmar"&gt;        
        &lt;transition on="salir" to="salir"/&gt;
    &lt;/view-state&gt;
    
    &lt;end-state id="salir" view="externalRedirect:./index?transferencia=1" /&gt;
    &lt;end-state id="cancel" view="externalRedirect:./index?transferencia=0" /&gt;</pre>

Esto se nos mostrara en pantalla una vez hayamos confirmado la transferencia

<img class="imagen_con_borde aligncenter wp-image-489 size-full" src="http://www.profesor-p.com/wp-content/uploads/2018/10/Captura-29.png" alt="" width="670" height="326" srcset="http://www.profesor-p.com/wp-content/uploads/2018/10/Captura-29.png 670w, http://www.profesor-p.com/wp-content/uploads/2018/10/Captura-29-300x146.png 300w" sizes="(max-width: 670px) 100vw, 670px" />

Y con esto termino esta entrada tan larga, pero donde creo que he explicado bastantes conceptos de WebFlow.

Como siempre, espero vuestros comentarios y mejoras, con la esperanza de haberme explicado bien.

¡¡ Hasta otra, alumnos!!

 