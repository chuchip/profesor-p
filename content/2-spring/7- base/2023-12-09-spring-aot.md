---
title: AOT en Spring
pre: "<b>o </b>"
author: profesor-p
type: post
weight: 300
date: 2023-12-09T07:57:07+00:00
url: /springboot/aot/
categories:
  - aot
  - java
  - spring
tags:
  - aot
  - java  
  - spring

---
Creo que **Aspect Oriented Programing** (*AOP*) no es muy conocido. **Spring** lo usa mucho y, a menudo, cuando pones etiquetas en tu código, **Spring** usará AOT.

No quiero explicar que es **AOP** porque hay mucha documentación al respecto. En este artículo, quiero darte algunos ejemplos y casos en los que el uso de AOP podría mejorar tu código. Quiero mostrarte una nueva herramienta para programar. Veamos si puedo hacerlo.

En primer lugar, te estarás preguntando, ¿por qué debería usar **AOP**? ¿Qué lo diferencia?

Bueno, lo más destacable de **AOP** es que te permite ejecutar tu propio código cuando un código externo va a ser ejecutado.

Pero antes de que puedas hacer esta magia en tu código debes incluir las dependencias necesarias en tu proyecto. Sólo tienes que añadir estas líneas a tu archivo pom.xml (estamos hablando de maven, ¿verdad?)

```xml
<dependencia>
     <groupId>org.springframework.boot</groupId>
     <artifactId>spring-boot-starter-aop</artifactId>
</dependencia>
```

En nuestro ejemplo, digamos que tenemos este archivo java:

```java
@Service
public class TestServiceImpl implements  TestService{
    @Override
    public void doSomething()
    {
        log.info("Here, I'll do something");
    }
    @Override
    public void doSomethingMore()
    {
        log.info("Here, I'll do something More");
    }
    @Override
    public  String doSomethingWithParams(String param1)
    {
        log.info("I've received the param `{}` ",param1);
        return "\nI've received the param '"+param1+"'";
    }
}
```

Ahora, imagina que quieres tomar el control cuando la función "**doSomething**" es llamada. ¿No sería genial si pudieras escribir código para que cuando esta función sea llamada, tu propio código también se ejecute?.

Ok, eso es lo que **AOP** te permite hacer.

Veamos el caso más simple. ¿Quieres escribir un log cuando la función "**doSomething**" es llamada?. Fácil. Escribe una clase como esta:

```java
import org.springframework.stereotype.Component;
import org.aspectj.lang.annotation.Aspect;

@Aspect
@Component
@Slf4j
public class LogAspect {
   
    @Before("execution(* com.profesorp.springaop.service.TestService.doSomething(..))")
    public void logAll()
    {
        log.info("I'm the Aspect logAll");
    }
}
```

Sí, eso es todo.

Necesitas escribir una clase con las etiquetas "**@Component**" y "**@Aspect**" y luego, crear una función con la etiqueta `@Before("execution(* com.profesorp.springaop.service.TestService.doSomething(..))")`

Ahora, cuando el código llame a la función "**doSomething**" el código de la función "**logAll**" se ejecutará antes que el código de "**doSomething**".

Utilizaremos la etiqueta "**before**" cuando necesitemos ejecutar código *antes* de que se ejecute el código de destino.
La palabra "**execution**" es obvia, ¿verdad? Exacto, lo que viene después será la clase con la que queremos interactuar. En este caso, el primer "*****" indica que la función puede devolver cualquier cosa, después se prondrá el nombre de la clase y entre los corchetes, pones "**..**" para indicar que el número de argumentos puede ser cualquiera.

Pero, el poder viene con la etiqueta " **@Around**" . Con esta etiqueta no sólo puedes ser un espectador, puedes ser un jugador.

Digamos que queremos ejecutar algún código cuando la función "**doSomethingWithParams**" es llamada pero, también quieres, ver el parámetro con que fue llamada, cambiar el valor del parámetro o, incluso, ¿por qué no?, cancelar la llamada. Y, recuerda, no quieres (o puedes) tocar el código de la función "**doSomethingWithParams**".

Vale. Hagámoslo. En la clase anterior 'LogAspect' vamos a incluir este código:

```java
@Around("execution(public * com.profesorp.springaop.service.TestService.*(..))")
public Object takeControl(ProceedingJoinPoint proceedingJoinPoint) throws  Throwable
{
 Object[] args =proceedingJoinPoint.getArgs();
 log.info("Number of Args of function: {} ",args.length);
 if (args.length==1)
 {
  if ( "skip".equals(args[0].toString())) {
   log.info("I'm going to skip the call  of Args of function: {} ", args.length);
   return "Skip in aspect!";
  }
  if ( "add".equals(args[0].toString()))
  {
   Object result=proceedingJoinPoint.proceed(new Object[] {"You've sent  the string add "});
   log.info("Executed target added 'add'");
   return result;
  }
 }
 Arrays.stream(args).forEach(arg -> log.info("Arg of the function: {}",arg.toString()));
 Object result=proceedingJoinPoint.proceed();
 if (result==null)
  log.info("There is not result");
 else
  log.info("Result of join is {}",result.toString());
 return result;
}
```

En este caso, quiero hacer algo cuando se llama a alguna función '**public**' de la clase "**com.profesorp.springaop.service.TestService**".

Observa que la función '**takeControl**" recibe el objeto '**ProceedingJoinPoint**'. La clase '**org.aspectj.lang.ProceedingJoinPoint**' es la que vamos a utilizar para la 'magia'.

En primer lugar, uso la función "**getArgs**()" para obtener el número de parámetros que la función fue llamada. Si el número de argumentos es uno, y es '**skip**" entonces devuelvo la cadena "*Skip in aspect!*". Hay una cosa importante aquí: la función '**doSomethingWithParams**' no se ejecutará y el objeto devuelto será '*Skip in aspect!*'.

¿Estás un poco confundido? No te preocupes. Permítanme tratar de explicarlo mejor.

Supongamos, que tienes este controlador:

```java
@RestController
@RequiredArgsConstructor
@Slf4j
public class TestController {
    final TestService service;
   
    @GetMapping("something/{param1}")
    public String doSomethingWithParam(@PathVariable String param1)
    {
        String var1= service.doSomethingWithParams(param1);
        return STR."I called doSomethingWithParams with param1 '\{param1}' and it returned: '\{var1}'";
    }
}j
```

Ahora, cuando se llama a este *endpoint*, la línea "**service.doSomethingWithParams(param1)**" se ejecutará y el resultado de la ejecución se almacena en la cadena "var1", ¿correcto?

Ok, entonces el valor de "var1" será:  '*Skip in aspect!*'.

![Captura1](/img/2023/aot/aot1.png)

Como hemos dicho, se puede cambiar el valor utilizado para llamar a la función de destino. En nuestro ejemplo, si enviamos la cadena '**add**' usaremos la función '**proceed**' de '**proceedingJoinPoint**' para ejecutar la función con nuestros propios parámetros.

![Captura2](/img/2023/aot/aot2.png)

Te recomiendo que clones mi repositorio donde podrás encontrar todo el código que has visto  en este artículo. Visita: https://github.com/chuchip/springAop
Encontrarás más ejemplos y aprenderás más cosas interesantes sobre AOP.

Una última cosa: En algunos ejemplos, puede que  veas algo parecido a este código:

```java
@Pointcut("execution(public * com.profesorp.springaop.service.TestService.*(..))")
public void pointCut1(){
 log.info("This, in pointCut1, is never executed.");
}
@Around("pointCut1()")
public Object aroundPoint1(ProceedingJoinPoint proceedingJoinPoint) throws  Throwable
{
...
}
```

No te preocupes. En este caso, la etiqueta "@Pointcout" se utiliza porque, probablemente usted quiere tener más que sólo una función para esa clase, por lo que, ahora puede utilizar "@Around", "@Before" , etc, con el nombre de la función de su PointCut. Es sólo una manera de escribir menos código algunas veces.

Gracias por leer este artículo y espero que te haya resultado interesante.