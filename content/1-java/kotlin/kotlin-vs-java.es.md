---
title: Kotlin vs Java
author: airec69
type: post
date: 2018-08-23T19:37:38+00:00
url: /2018/08/23/kotlin-vs-java/
categories:
  - java
  - kotlin
tags:
  - java
  - kotlin

---
Ultimamente se habla mucho de Kotlin, que si Google lo ha marcado como lenguaje preferente en la IO 2017, que si es un lenguaje muy mega-guay para programar en Android, que si soluciona los problemas de Null Pointer Exception.

En esta entrada voy a dar mi opinión (sin que nadie me pague 😉 ) comparando ambos lenguajes y diciendo, en mi opinión si es tan guay Kotlin, o es mas humo e intereses comerciales.
 <!--more--> 
En primer lugar, decir que Kotlin es compatible con Java, es decir, utiliza la máquina virtual de Java y ademas es compatible con este, vamos que tu puedes tener tu programita en Kotlin y ejecutar código Java ya creado, con lo cual todas las librerias de Java funcionan en Kotlin. Ahora voy a enumerar las ventajas de Kotlin y ver si realmente son tan trascendentes.

1. **Kotlin soluciona el problema de los Null Pointer Exception.**

A este tema se le ha dado mucha publicidad. En Kotlin, efectivamente, cuando defines una variable tienes que especificarle un valor y si no,  marcar la variable como que  va a aceptar nulos, Si la marcas como que acepta nulos,  no te deja  juntarla con otras variables que no han sido marcadas como posibles nulables (vamos, que pueden ser null ).Bueno, realmente si te deja, pero te obliga a marcar el resultado como posible null.

El problema es que al final, si usas librerias Java  externas (como es lo normal), no tienes ningun control sobre si las funciones Java puras devuelven valores Null o no, con lo cual, lo mas probable es que tengas Null Pointers Exception como antes. Es decir, cuando solo uses Kotlin y todas las librerias externas sean Kotlin, pues igual a este tema  le sacas algun provecho pero actualmente se queda cojo.

2. **Kotlin tiene expresiones lambda.**

Vale, Java 8.0 (que ya tiene un tiempo) tambien lo tiene. Otro tema es que se utilize mas o menos, pero vamos que no es una ventaja sobre Java.

3. **Kotlin soporta variables genericas.**

En Kotlin, cuando defines una variable no tienes porque especificar de que tipo es.  O sea, puedes crear una variable tal que asi:

```var n=1
var s="hola"
```

Esto esta muy guay y evita tener que andar poniendo el tipo de variable, pero&#8230;. en Java 10, tambien lo tenemos disponible, usando, ademas, la misma sintaxis. (var n=1;). La unica diferencia es que en Kotlin, las lineas no tienen porque ir terminadas con el punto y coma.

4. **Extensiones a Funciones.**

Kotlin, sobre todo programando en Android, tiene ciertas extensiones, por ejemplo la  que hace que puedas acceder a las variables de tu entorno grafico directamente. Otra extension es que tu defines un POJO y no hace falta poner los setters y getters de siempre. Directamente Kotlin los crea por ti. Eso esta muy bien, pero&#8230; en Java tenemos el proyecto <a href="https://projectlombok.org/" target="_blank" rel="noopener">Lombok </a>que hace lo mismo, es decir crear _al vuelo,_ las sentencias setters , getters y alguna otra cosa más (echarle un vistazo, esta fenomenal).

Bueno, por no extenderme demasiado, Kotlin es un gran lenguaje y tiene cosas muy chulas, como por ejemplo, el hecho de definir en las funciones valores por defecto, de tal manera que en una funcion puede estar definida algo asi como esto:

<pre>fun multi(x:Int =0,y:Int=0)</pre>

De tal manera que a esta funcion la podremos llamar con un parametro, con dos o con ninguno, con lo cual te evitas el  crear 3 funciones como tendrias que crear en Java, y que una llame a otra (es decir, es un ahorro de código sin mas).

Mi opinión: Si vais a programar en Android, aprender Kotlin, la curva de aprendizaje es muy rapida y esta claro que Google le va  a dar mucho soporte y añadirle más cosas que ayudaran.Es obvio que Android no quiere depender de Java (entre otras cosas por los problemas de licencias que esta teniendo con Oracle) y por eso, al final, Kotlin sera el lenguaje que mandara en Android.

Si usais Java, pero programando, por ejemplo, servicios Web, yo no me molestaria demasiado en aprender Kotlin: las mejoras que te ofrece no son tan grandes y las que valen la pena, las tienes en las ultimas versiones de Java. Pero claro, esto es solo mi opinión 😉

&nbsp;