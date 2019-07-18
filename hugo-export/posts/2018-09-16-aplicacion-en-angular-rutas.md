---
title: Aplicación en Angular – Rutas
author: El Profe
type: post
date: 2018-09-16T14:42:39+00:00
url: /2018/09/16/aplicacion-en-angular-rutas/
categories:
  - angular
tags:
  - angular
  - angular6
  - curso
  - routes

---
Una vez he explicado [en la anterior entrada][1]  como inicializa Angular la aplicación, voy a explicar como hacer para que esta pueda aceptar parámetros a través de la URL introducida en el navegador. Básicamente, lo que deseo hacer es que, suponiendo que nuestra aplicación este corriendo en http://localhost:4200/ (es la dirección por defecto en la que escucha Node.js cuando lo lanzamos con el comando ng serve  ), si vamos a la dirección http://localhost:4200/2018  nos muestre las ventas del ejercicio 2018. A su vez, si vamos a la dirección http://localhost:4200/2018/3 nos deberá mostrara los datos del ejercicio 2018 y en el detalle los datos del mes 3. Es decir, el primer número sera el ejercicio y el segundo el mes. Si no se pone el segundo parámetro iremos al mes 1 del ejercicio mandado. En el caso de no poner ningún parámetro, el programa nos preguntara el ejercicio con el correspondiente formulario.

En Angular, las rutas se definen creando un fichero TypeScript,  al que las buenas normas, nos aconsejan llamar **app-routing.module.ts**. Este fichero se dejara en el directorio src/app/.

Veamos el fichero creado en nuestra aplicación:

<div>
  <pre>import { NgModule } from '@angular/core';
import { CommonModule } from '@angular/common';
import { RouterModule, Routes } from '@angular/router';
import {DatosEjercicioComponent} from './datos-ejercicio/datos-ejercicio.component';

const routes: Routes = [    
  {path: ':year', component: DatosEjercicioComponent },
  {path: ':year/:month', component: DatosEjercicioComponent }
];

@NgModule({
  imports: [
    CommonModule,
    RouterModule.forRoot(routes) 
  ],
  exports: [ RouterModule ],
  declarations: []
})

export class AppRoutingModule { }</pre>
</div>

Como se ve, lo primero tenemos una serie de _imports_ que necesitaremos.  A continuación declaramos una variable, a la que llamamos **routes ** (le podriamos haber llamado mis\_rutitas\_chulis pero es por ser serios 😛 ) y que es un array de objetos tipo **Routes** .  En esa variable es donde vamos a definir nuestras rutas.

Así, el primer elemento del array tiene el elemento _path_ cuyo valor es &#8220;year&#8221;, y el elemento _component_ cuyo valor es: _DatosEjercicioComponent_. Esto, Angular lo traducirá por: cuando la URL tenga un único parámetro (year) carga el componente DatosEjercicioComponent.  El especificar que es un parámetro lo hacemos con los dos puntos (&#8220;:&#8221;) antes de la palabra :year. Y el nombre de esa variable podremos capturarla y utilizarla en nuestro componente _DatosEjercicioComponent._

En el caso de que la linea fuera:

<pre>{path: '/paco', component: DatosEjercicioComponent },</pre>

Angular cargaría el componente indicado cuando fuéramos a la ruta http://localhost:4200/paco . Ojo, si la dirección fuera, otra, como http://localhost:4200/paco/el_guapo no funcionaria.

Aclarar, ademas, que DatosEjercicioComponent es el componente que hemos definido en  la linea **import {DatosEjercicioComponent} from &#8216;./datos-ejercicio/datos-ejercicio.component&#8217;**. Si no tuvieramos ese import Angular nos daría un error.

La carga de la variable route, en el  modulo de rutas de Angular se hace con la linea: **RouterModule.forRoot(routes)**

Hay maneras de cargar diferentes rutas para una misma aplicación, pero no vamos a hablar de ello en este curso.

Para especificar que debe cargar el componente marcado por la ruta, debemos incluir en el código HTML la siguiente etiqueta: **<router-outlet>.** Esa etiqueta incrustara el correspondiente componente de la ruta, como hacia la etiqueta **<app-root>** con el componente **AppComponent** en la anterior entrada.

Ahora veamos parte de nuestra clase <span style="background-color: #ccffff;">datos-ejercicio.component.ts</span>

<pre>import { ActivatedRoute, Router} from '@angular/router';
@Component({
  selector: 'app-datos-ejercicio',
  templateUrl: './datos-ejercicio.component.html',
  styleUrls: ['./datos-ejercicio.component.css']
})
export class DatosEjercicioComponent  {
constructor( private route: ActivatedRoute,
    private router: Router, private _datosserver: DatosserverService)  
 { 
  }

  ngOnInit() {       
    this.year = +this.route.snapshot.paramMap.get('year');

....</pre>

Como vemos, lo primero es importar los componentes para tratar rutas. Después especificamos que nuestra clase es del tipo  componente con la etiqueta _@Component_ para después definirla.

Lo primero que encontramos es la palabra **constructor**, con esa palabra definimos un tipo de función especial dentro de nuestra clase que sera llamada cuando el objeto sea creado. En nuestro ejemplo, cuando Angular encuentre la etiqueta **<router-outlet>** y deba cargar nuestra clase, lo primero que hará sera ejecutar el código existente en esa función.

En este caso, como veis, no hay nada dentro de la función, sin embargo si que recibe una serie de parámetros. Esas variables estarán disponibles dentro de nuestra clase, a nivel global. Eso es una característica de **TypeScript**, las variables recibidas en un constructor estarán disponibles para toda la clase y no solo dentro del constructor como ocurre en todos los demás tipos de funciones.

En la función **ngOnInit**, se le asigna a la variable global **year** el valor recibido por el componente de la ruta. Observése el símbolo más (+) . Esto convertirá el texto recibido en un número.

La función **ngOnInit** es una función especial de Angular y sera llamada una vez el componente a sido construido (o sea, después de llamar al constructor), pero antes de incrustarlo (por así decirlo) en la pantalla dentro del código HTML. Hay otra serie de funciones especiales que podéis consultar en la <a href="https://angular.io/guide/lifecycle-hooks" target="_blank" rel="noopener">pagina oficial de Angular</a>.

Y con esto termino esta entrada hablando sobre las rutas en Angular. Podéis profundizar más en este tema en la <a href="https://angular.io/guide/router" target="_blank" rel="noopener">documentación oficial de Angular </a>o visitando este otro excelente manual <a href="https://academia-binaria.com/paginas-y-rutas-angular-spa/" target="_blank" rel="noopener">https://academia-binaria.com/paginas-y-rutas-angular-spa/</a>

 [1]: http://www.profesor-p.com/2018/09/14/aplicacion-en-angular-inicializando/