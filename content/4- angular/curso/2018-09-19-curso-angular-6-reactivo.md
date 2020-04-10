---
title: Curso Angular 6 – Reactivo!
author: El Profe
chapter: false
pre: "<b>o </b>"
type: post
date: 2018-09-19T10:43:09+00:00
url: /2018/09/19/curso-angular-6-reactivo/
enclosure:
  - |
    |
        http://www.profesor-p.com/img/2018/09/prueba4.mp4
        86909
        video/mp4
        
categories:
  - angular
tags:
  - angular
  - angular6
  - reactivo

---
Antes de continuar con el curso quiero dejar claro que Angular es **reactivo**.

¿ Que significa eso ?.

Pues básicamente que cualquier cambio que se haga en el modelo sera transmitido a la vista y viceversa. El modelo, entiéndase que son nuestros _Componentes. _Es decir, nuestras clases definidas en los ficheros TypeScript. La vista es el código HTML que se visualizara en nuestro navegador.

Así, si cambiamos el valor de un campo **INPUT **del HTML, que este unido con la directiva **[(ngModel)] **a una variable. Esa variable se modificara en tiempo real.

Obsérvese el siguiente código:

```
import { Component} from '@angular/core';

@Component({
  selector: 'app-datos-mes',
  template: '<p>Teclea aquí: <input [(ngModel)]="variable"/> <br>lo que estas tecleando: {{variable}} </p>',
})
export class DatosMesComponent{
  variable:string="";
} 
```
  
Esto nos mostraría una pantalla como esta:

!()[/img/2018/09/Captura1.png)

Al modificar el valor del input, cambiara el valor mostrado.


En cuanto tecleemos una letra, el valor de <strong>variable </strong>cambiara y, por lo tanto, el valor mostrado con <strong>{{variable}}  </strong>también cambiara.


Esto funciona a todos los niveles. Incluyendo las condiciones.

```
import { Component} from '@angular/core';

@Component({
  selector: 'app-datos-mes',
  template: `<p>Teclea aquí: <input [(ngModel)]="variable"/>
     <br>lo que estas tecleando: {{variable}} </p>
     <div *ngIf="variable=='0'">Variable es cero</div> `, 
})
export class DatosMesComponent{
  variable:string="";
}
```

Así, en el momento que el valor de <strong>variable </strong>sea igual a &#8220;0&#8221; Nos mostrara el texto <strong>Variable es cero. </strong>Como se puede ver en el siguiente vídeo .


<div style="width: 856px;" class="wp-video">
  <!--[if lt IE 9]><![endif]--><video class="wp-video-shortcode" id="video-308-1" width="856" height="480" preload="metadata" controls="controls"><source type="video/mp4" src="/img/2018/09/prueba4.mp4?_=1" />
  
  <a href="/img/2018/09/prueba4.mp4">/img/2018/09/prueba4.mp4</a></video>
</div>