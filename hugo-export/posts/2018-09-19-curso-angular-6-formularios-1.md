---
title: Curso Angular 6 – Formularios (1)
author: El Profe
type: post
date: 2018-09-19T08:29:58+00:00
url: /2018/09/19/curso-angular-6-formularios-1/
categories:
  - angular
tags:
  - angular
  - formularios

---
En  la aplicación que estamos desarrollando, de la cual, os recuerdo tenéis el código fuente en <a href="https://github.com/chuchip/yagesclient-angular" target="_blank" rel="noopener">https://github.com/chuchip/yagesclient-angular</a>, tenemos que poder solicitar al usuario el ejercicio sobre el que vamos a realizar la consulta.

Para introducir ese dato, vamos a utilizar un formulario muy simple, pero que nos servirá para entender algunos conceptos básicos de Angular.

Trabajando con el componente **AppComponent**, definido en el fichero **src/app/app.component.ts** nos centramos primero en  el código HTML  el cual,  es el siguiente:

<span style="background-color: #ccffff;">app.component.html</span>

<pre>&lt;form  (ngSubmit)="buscar()"&gt;
       &lt;label value="Introduzca año"&gt;Introduzca Año &lt;/label&gt;
      &lt;input   [(ngModel)]="ejercicio" /&gt;
      &lt;span *ngIf="msgError !=''"  &gt;
                     {{msgError}}
      &lt;/span&gt;
      &lt;div&gt;&lt;button &gt;Consultar&lt;/button&gt;&lt;/div&gt;
&lt;/form&gt;</pre>

En la etiqueta **form** con el modificador  **(ngSubmit)=&#8221;buscar()&#8221; **indicaremos que llame a la función **buscar** del componente **AppComponent **cuando se envié el formulario con el botón correspondiente.

En el input, vemos la etiqueta **[(ngModel)] ,** a la que le asignamos el valor **&#8220;ejercicio&#8221;**. Esta etiqueta sirve para unir el valor de la variable **ejercicio** con la vista. Esa unión es bidireccional; si se modifica el valor de la variable en **AppComponent  ** se cambiara el valor en la vista y viceversa. Estos cambios son inmediatos, es decir en cuanto introduzcamos una letra (o la borremos) en la pagina web, el valor de la variable **ejercicio**  cambiara inmediatamente.

La  variable, esta definida como otra cualquiera, sin ningún tipo de decorador o etiqueta especial.

A continuación, en  la etiqueta **span** vemos la directiva ***ngIf .** Con ella indicamos que la etiqueta **span **y todo lo que haya dentro de ella solo se debe procesar (y por lo tanto mostrar) si se cumple la condición indicada.

En este caso, si la variable **msgError** es diferente de &#8220;&#8221; (o sea si no esta vacía)  se ejecutara la etiqueta **span**. En caso contrario, simplemente saltara hasta donde se cierre esa etiqueta con **</span>**

Dentro de **span** encontramos el texto  **{{msgError}}**. Con las dobles llaves le indicamos a Angular que lo que hay en su interior es una variable de nuestro _componente _que debe mostrar.  Digamos que es un poco como la directiva **ngModel **pero de solo salida. De tal modo que si cambia el valor de la variable, cambiara el valor mostrado en nuestro navegador.

La variable entre las dobles llaves puede ser de cualquier tipo. Una cadena de texto, un objeto, un número, etc.

<span style="background-color: #00ffff;">app.component.ts</span>

<pre>export class AppComponent  {
       <strong> ejercicio=2018</strong>; 
        ejercicioActual=0;
        <strong>msgError="";</strong>
        <strong>constructor</strong>( private router: Router) {    
        }
	<strong>buscar</strong>(): void 
	{
		let num:number=+this.ejercicio;
		if (num == 0 )
		{
		  this.msgError="Ejercicio no ES valido";		 
		  return;
		}
		this.msgError="";
		this.ejercicioActual=this.ejercicio;
		this.router.navigate([''+this.ejercicioActual])
	}
}</pre>

De este modo, cuando muestre el formulario en la página web, el input aparecerá con el valor 2018.

Observemos ahora la función **buscar. **Esa función, como hemos dicho,  sera llamada  cuando pulsemos el botón de nuestro formulario.

Lo primero que hacemos es asignar a una variable local, a la que hemos llamado **num** el valor de la variable global **ejercicio.** Obsérvese la palabra **this** antes de ejercicio, para indicar que esa variable es global. Si pondríamos** ejercicio** sin la palabra reservada **this**, Angular daría un error, avisando de que esa variable no existe.

Si nos fijamos vemos que se pone un más (+) antes de **this.ejercicio ** esto es una facilidad que nos da JavaScript para transformar una variable a tipo numérico. De ese modo, si el usuario ha introducido en el campo de texto algo que no sea un numero, por ejemplo &#8220;pepe&#8221;, como no se podrá transformar **this.ejercicio **a un número, el valor de **num **sera igual a cero.

En el caso de que **num** sea cero, ponemos la variable global **this.msgError **con el texto de error y saldrá de la función.

En caso de que **num** sea diferente a cero ponemos la variable **this.msgError **a &#8220;&#8221; (cadena vacía) , y la variable **ejercicioActual** la hacemos igual a **ejercicio,** para después navegar a la ruta **ejercicioActual**, llamando a la función navigate del objeto **this.router **que Angular habrá inyectado en nuestra clase al haber puesto en el constructor que recibimos la variable **router** del tipo  **Router.**

Entiéndase que _navegar _es a todos efectos como si el usuario pusiera en la barra de navegación lo que nosotros hemos programado. En este caso, si se ha introducido el valor **2018** en el formulario**,** iríamos a la dirección http://localhost:8080/2018

Resumiendo, cuando se pulse el botón **Consultar**, si el usuario introduce un número valido en el formulario, navegaremos al ejercicio tecleado. En caso contrario aparecerá el mensaje **Ejercicio no ES valido.**

¡¡ Nos vemos en la próxima clase 😉 !!

&nbsp;