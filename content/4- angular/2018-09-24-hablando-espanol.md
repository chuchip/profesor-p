---
title: Angular – Hablando español
author: El Profe
pre: "<b>o </b>"
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

Angular si no se le dice lo contrario, es ingles del bueno. Las fechas las pondrá en su formato de mes/dia/año . Los números serán con las comas con separadores de miles y los decimales con puntos. Vamos, muy ingles todo.

El problema es que no todo el mundo es ingles (aunque les pene a los ingleses ;- )) . Así que en esta entrada voy a explicar como hacer que Angular se nos vuelva español.

Lo primero es editar el fichero <strong>app.module.ts</strong>, para importar nuestros ficheros de locale (lease localización 😉 ) .

```
import localeEs from '@angular/common/locales/es';

registerLocaleData(localeEs, 'es');
```

Ahora tendremos que incluir un _provider_ para que cuando mostremos un valor por pantalla  y queramos formatearlo lo formatee con nuestro locale español.

De hecho, si no ponemos este provider, Angular cascara con gran alegría, si intentamos utilizar un pipe.
```
@NgModule({
.....
  providers: [ { provide: LOCALE_ID, useValue: 'es' } ],
.....
})
```

Ahora si en nuestra aplicación tenemos este código:
```
import { Component } from '@angular/core';
@Component({
  selector: 'app-prueba',
  template: `
  <p>
  Numero: 
  {{numero |  number: '1.2-2'}}<br>
  Fecha: {{fecha | date: 'shortDate'}}
</p>
  `
})

export class PruebaComponent {
  numero=1234.5;
  fecha=new Date();  
}
```
   
Donde, como podemos ver utilizamos pipes, para formatear un numero: <strong>{{numero | number: &#8216;1.2-2&#8217;}} </strong>y una fecha: <strong>{{fecha | date: &#8216;shortDate&#8217;}} </strong>

El resultado sera el siguiente, que como se ve, es la salida en nuestro querido lenguaje español.

![](/img/2018/09/Captura-1.png)

Ahora ya solo falta que hagáis vuestra aplicación multi-lenguaje.  O sea, que se pueda elegir el idioma en que se van a presentar tanto los textos como los lenguajes.

Pero eso, chavales, es para nota.. y si me apetece ya lo explicare otro día.
