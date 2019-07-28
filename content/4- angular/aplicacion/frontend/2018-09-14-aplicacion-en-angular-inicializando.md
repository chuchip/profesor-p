---
title:  Angular - Inicializando
weight: 20
author: El Profe
pre: "<b>o </b>"
type: post
date: 2018-09-14T14:53:10+00:00
url: /2018/09/14/aplicacion-en-angular-inicializando/
categories:
  - angular
tags:
  - angular
  - angular 6
  - frontend
  - html

---
Continuando con [la entrada donde instalaba Angular][1], seguimos desarrollando la aplicación que detallo en [esta página][2].

Una vez tenemos creado el esqueleto de nuestro programa, con el comando &#8220;**ng new&#8221; , ** entraremos al directorio **src. **En este directorio es donde realmente vamos a trabajar.

Los demás directorios son donde están las librerías y utilidades que nuestra aplicación usara pero que son propias de Angular y nosotros no las tocaremos (al menos en este ejemplo).

Primero vamos a explicar los pasos que sigue Angular para ejecutar nuestra recién creada aplicación.

Cuando ejecutamos el comando &#8220;**ng serve &#8211;open&#8221;** desde nuestro directorio **yagesclient**,Angular, una vez que ha compilado, comprobado dependencias y otra serie de tareas, en las que no voy a entrar, lee el fichero **src/main.ts**, el cual detallo a continuación.

**<span style="background-color: #00ffff;">src/main.ts</span>**

<pre>import { enableProdMode } from '@angular/core';
import { platformBrowserDynamic } from '@angular/platform-browser-dynamic';

import { AppModule } from './app/app.module';
import { environment } from './environments/environment';

if (environment.production) {
  enableProdMode();
}

platformBrowserDynamic().bootstrapModule(AppModule)
  .catch(err =&gt; console.log(err));</pre>

En este fichero se importan (**import**)  es decir, se cargan  una serie de ficheros para después arrancar nuestra aplicación. con la ultima linea:

<pre>platformBrowserDynamic().bootstrapModule(<strong>AppModule</strong>)
.catch(err =&gt; console.log(err));</pre>

Como se puede intuir este comando ejecuta una función, a la que se le pasa el parámetro **AppModule**. No voy a entrar a explicar los detalles , pues se escapa al ámbito de este curso, lo que voy a  explicar es donde definimos ese parámetro **AppModule** .

La clave esta en la linea:

<pre><strong>import { AppModule } from './app/app.module';</strong></pre>

Ahí importamos ese parámetro, especificando  que lo debe cargar del fichero &#8220;**./app/app.module.ts&#8221;**. La extensión **ts**  (**T**ype**S**cript) la pone Angular pues busca un fichero de ese tipo, pero nosotros no debemos ponerlo.

En este  fichero están definidos los   _módulos_ de nuestra aplicación.

El contenido del fichero es el siguiente:

<span style="background-color: #00ffff;"><strong>src/app/app.module.ts</strong></span>

<pre>import { BrowserModule } from '@angular/platform-browser';
import { NgModule } from '@angular/core';

import { AppComponent } from './app.component';

@NgModule({
  declarations: [
    AppComponent
  ],
  imports: [
    BrowserModule,
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }</pre>

Este fichero define un _modulo,_ porque así se lo especificamos a Angular  con la etiqueta **@NgModule.** El _modulo_ se llamara **AppModule,** que no deja de ser una clase u objeto, que exportamos a la vez que creamos,  con la sentencia:

<pre>export class AppModule { }</pre>

Esta clase **AppModule **es el objeto que importábamos en nuestro anterior fichero con la linea <span style="background-color: #d9d4d4;"><strong>import { AppModule } from &#8216;./app/app.module&#8217; </strong></span>

El nombre de la clase podría haber sido perfectamente **miMaravillosaClase  **con lo cual habríamos puesto:

<pre>export class <strong>miMaravillosaClase  </strong>{ }</pre>

Y en el import seria **<span style="background-color: #d9d4d4;">import { miMaravillosaClase  } from &#8216;./app/app.module&#8217;</span>
  
** 

Lo que quiero resaltar es que el nombre de la clase **no **debe coincidir con el del fichero ni tiene porque acabar en **Module**, sin embargo, por claridad en el código y buenas practicas los módulos siempre se terminan con Module y coincide el nombre del modulo con la del fichero.

Volviendo al fichero <span style="background-color: #00ffff;"><strong>/app/app.module.ts</strong></span>, centrémonos en el siguiente  **import.**

<pre>import { AppComponent } from './app.component';</pre>

Donde importamos el componente **AppComponent . **Este _componente esta definido en el fichero _&#8216;./app.component.ts&#8217;

<span style="background-color: #00ffff;"><strong>src/app/app.component.ts</strong></span>

<pre>import { Component } from '@angular/core';

@Component({
<strong>  selector: 'app-root',</strong>
 <strong> templateUrl: './app.component.html',</strong>
  <strong>styleUrls: ['./app.component.css']</strong>
})
export class AppComponent {
   title = '<em>Cliente de Aplicacion de prueba</em>';
}</pre>

Aquí definimos un _componente_ (obsérvese la etiqueta **@Component) **que sera una clase llamada **AppComponent** anteriormente importada.

Con la etiqueta **selector** indicamos como llamaremos a este componente en nuestros fichero HTML (en la parte de la vista, vamos). Es decir, se llamara **app-root**.

Con **templateUrl** indicaremos el fichero html a cargar cuando pongamos la etiqueta  **app-root** en algun otro fichero html que Angular ya haya cargado (index.html en nuestro ejemplo). Con **styleUrls** indicaremos la hoja de estilo a cargar para el anterior fichero html.

Un _componente_ en angular no es sino una objeto que nosotros definimos. Ese objeto tendrá un código HTML, su hoja de estilos (CSS) y por supuesto JavaScript.

<div style="background-color: #00ffff; border: 1px solid blue;">
  ¿ JavaScript ?. Sí, porque aunque en angular nosotros usamos TypeScript,  a  la hora de compilar el programa, ese código se  convertirá  en JavaScript, que es el lenguaje que entienden los navegadores (Chrome, Firefox, Opera, IE, etc).
</div>

Nuestra aplicación, como vemos,  ya tiene su primer _componente _creado, el objeto **AppComponent**, y este componente es usado por el fichero **index.html**, del cual enseñamos una versión simplificada a continuación.

<span style="background-color: #00ffff;">/src/index.html</span>

<pre>&lt;!doctype html&gt;
&lt;html lang="en"&gt;
&lt;head&gt;
   
  &lt;meta charset="utf-8"&gt;
  &lt;title&gt;Yagesclient&lt;/title&gt;
  &lt;base href="/"&gt;

  &lt;meta name="viewport" content="width=device-width, initial-scale=1"&gt;
  &lt;link rel="icon" type="image/x-icon" href="favicon.ico"&gt;
&lt;/head&gt;
&lt;body&gt;
  <strong>&lt;app-root&gt;&lt;/app-root&gt;</strong>
&lt;/body&gt;
&lt;/html&gt;</pre>

Como se puede ver es un fichero HTML _casi_ normal. Lo único que nos debe llamar la atención es  la etiqueta:

<span style="background-color: #d9d4d4;"><strong> <app-root></app-root></strong></span>. ¿ Os suenan ?. Exacto, son las que hemos definido con el parámetro **selector: &#8216;app-root&#8217;.**

Resumiendo, cuando Angular ejecute nuestra aplicación cargara el fichero _index.html_ y cuando encuentre las etiqueta **<app-root>** sabrá que debe incluir el componente definido en el fichero **src/app/app.component.ts**.

Lo que incrustara sera el contenido del fichero html definido en la anteriormente citada etiqueta **templateUrl**, ademas de instanciar (crear) el objeto **AppComponent.**

Sin entrar en detalles, para ver la relación entre la clase y el fichero HTML, o dicho de otra manera, la relación _vista / controlador,_ obsérvese que dentro de la clase **AppComponent** se crea la variable  **title** asignándole  el valor &#8221;_Cliente de Aplicacion de prueba_&#8216;. Como en el fichero **src/app.component.html** tenemos solo la siguiente linea

<pre>Esto es el titulo {{ title }}</pre>

El código  **<app-root></app-root>** del fichero index.html sera sustituido por:

<pre>Esto es el titulo <em>Cliente de Aplicacion de prueba</em></pre>

Y con esto termino esta entrada, en la próxima hablare de las rutas. Espero haberme explicado bien. En caso contrario no dudéis en realizar todas las preguntas necesarias.

&nbsp;

 [1]: http://www.profesor-p.com/2018/09/13/aplicacion-en-angular-instalacion-y-configuracion-basica/
 [2]: http://www.profesor-p.com/aplicacion-usando-java-y-angular/