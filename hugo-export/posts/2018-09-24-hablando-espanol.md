---
title: Angular – Hablando español
author: El Profe
type: post
date: 2018-09-24T10:45:43+00:00
url: /2018/09/24/hablando-espanol/
categories:
  - angular
  - i18n
tags:
  - angular
  - i18n
  - internacionalizacion

---
<div>
  Angular si no se le dice lo contrario, es ingles del bueno. Las fechas las pondrá en su formato de mes/dia/año . Los números serán con las comas con separadores de miles y los decimales con puntos. Vamos, muy ingles todo.
</div>

<div>
  El problema es que no todo el mundo es ingles (aunque les pene a los ingleses ;- )) . Así que en esta entrada voy a explicar como hacer que Angular se nos vuelva español.
</div>

<div>
</div>

<div>
  Lo primero es editar el fichero <strong>app.module.ts</strong>, para importar nuestros ficheros de locale (lease localización 😉 ) .
</div>

<div>
</div>

<pre>import localeEs from '@angular/common/locales/es';

registerLocaleData(localeEs, 'es');</pre>

Ahora tendremos que incluir un _provider_ para que cuando mostremos un valor por pantalla  y queramos formatearlo lo formatee con nuestro locale español.

De hecho, si no ponemos este provider, Angular cascara con gran alegría, si intentamos utilizar un pipe.

<div>
  <div>
    <pre>@NgModule({
.....
  providers: [ { provide: LOCALE_ID, useValue: 'es' } ],
.....
})</pre>
    
    <p>
      Ahora si en nuestra aplicación tenemos este código:
    </p>
    
    <div>
      <div>
        <pre>import { Component } from '@angular/core';
@Component({
  selector: 'app-prueba',
  template: `
  &lt;p&gt;
  Numero: 
  {{numero |  number: '1.2-2'}}&lt;br&gt;
  Fecha: {{fecha | date: 'shortDate'}}
&lt;/p&gt;
  `
})

export class PruebaComponent {
  numero=1234.5;
  fecha=new Date();  
}</pre>
      </div>
    </div>
    
    <p>
      Donde, como podemos ver utilizamos pipes, para formatear un numero: <strong>{{numero | number: &#8216;1.2-2&#8217;}} </strong>y una fecha: <strong>{{fecha | date: &#8216;shortDate&#8217;}} </strong>
    </p>
    
    <p>
      El resultado sera el siguiente, que como se ve, es la salida en nuestro querido lenguaje español.
    </p>
    
    <p>
      <img class="alignnone size-full wp-image-336" src="http://www.profesor-p.com/wp-content/uploads/2018/09/Captura-1.png" alt="" width="312" height="125" srcset="http://www.profesor-p.com/wp-content/uploads/2018/09/Captura-1.png 312w, http://www.profesor-p.com/wp-content/uploads/2018/09/Captura-1-300x120.png 300w" sizes="(max-width: 312px) 100vw, 312px" />
    </p>
    
    <p>
      Ahora ya solo falta que hagáis vuestra aplicación multi-lenguaje.  O sea, que se pueda elegir el idioma en que se van a presentar tanto los textos como los lenguajes.
    </p>
    
    <p>
      Pero eso, chavales, es para nota.. y si me apetece ya lo explicare otro día.
    </p>
  </div>
</div>