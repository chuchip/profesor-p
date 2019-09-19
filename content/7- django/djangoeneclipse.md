---
title: Creando una aplicacion DJango con Python3 en Eclipse
weight: 10
pre: "<b>o </b>"
chapter: false
author: El Profe
type: post
date: 2019-08-18
categories:
  - python
  - eclipse
  - django
tags:
 - python
  - eclipse
  - django
  - django-rest

---

Realmente para crear un proyecto con  [DJango](https://www.djangoproject.com/) no necesitamos ningún IDE sin embargo siempre es más cómodo usar uno que nos facilite el poder realizar **debug** , además de ofrecernos otras ventajas.

Podríamos usar [PyCharm](https://www.jetbrains.com/pycharm/download/) el cual crea un entorno automáticamente pero en este tutorial vamos a explicar como usar  Eclipse con el plugin [PyDev](https://www.pydev.org/).

[En esta página](https://www.django-rest-framework.org/tutorial/quickstart/) tienes los primeros pasos para crear un entorno para DJango pero yo voy a explicarlo para poder usarlo después con **PyDev**. Aclarar que este este articulo el proyecto fue creado  en una máquina con Windows 10.

Hay dos opciones para crear el proyecto. Usando un entorno de trabajo (*enviroment*) o sin usarlo. Si prefiere no usarlo pase al [punto dos](./#creando-el-proyecto-django). En caso de querer usarlo  sigue leyendo.

### **1. Usando  un entorno de trabajo.**

El usar un entorno de trabajo en Python es muy recomendable ya que nos evita problemas de dependencias. Podremos tener diferentes versiones de Python y de las librerías usadas para cada entorno, de tal manera que si, por ejemplo, actualizamos la versión de Python en la máquina, los proyectos creados bajo un entorno dado, seguirán usando la versión de Python con las que originalmente se  realizaron.

En nuestro caso vamos a crear un entorno de trabajo que incluya [DJango](https://www.djangoproject.com/) y [Django-Rest](https://www.django-rest-framework.org) , para ello desde la línea de comandos (cmd) ejecutaremos lo siguiente:

```bash
c:\python> python -m vvenv djangoenv
```

**djangoenv** será el nombre del directorio donde se creara el entorno de trabajo. Para cambiar a este entorno ejecutaremos el comando `djangoenv\Scripts\activate`. Observaremos que ahora el prompt empieza por **(djangoenv)**. Es decir deberemos ver algo parecido a esto: `(djangoenv) C:\Users\...` 

Ahora instalaremos los paquetes de **DJango** y **DJangoRest**.

```bash
(djangoenv) c:\python> pip install django djangorestframework
```

 Listo, ya tenemos nuestro entorno de Python con los paquetes instalados. Lo podemos comprobar con el siguiente comando:

```bash
(djangoenv) C:\python> pip list
Package             Version
------------------- -------
Django              2.2.5
djangorestframework 3.10.3
pip                 19.0.3
pytz                2019.2
setuptools          40.8.0
sqlparse            0.3.0
(djangoenv) C:\python>python --version
Python 3.7.4
```



En el menú `Windows` de Eclipse, escogeremos la opción `Preferences` para crear un nuevo perfil de Django. Para ello bajaremos a la rama `Python Interpreter`

![captura8](/img/djangoeclipse/captura8.png)

 En la nueva ventana que nos aparece erigiremos la opción `Browse for python/pypy exe` 



![captura9](/img/djangoeclipse/captura9.png)

Iremos al directorio donde hemos creado el entorno **djangoenv** , entrando a la carpeta **Scripts**  para seleccionar el ejecutable **python**.

Como nombre del interprete pondremos **djangoenv**.

![captura2](/img/djangoeclipse/captura2.png)

En la siguiente pantalla dejaremos los valores como aparecen por defecto.

![captura3](/img/djangoeclipse/captura3.png)



### 2. Creando  el proyecto DJango

Cerraremos la ventana  de preferencias y crearemos un nuevo proyecto del tipo "*PyDev DDJango*"

![Ficheros creados](/img/djangoeclipse/captura1.png)

Elegiremos el nuevo *interprete* creado, pondremos un nombre al proyecto y daremos a **Next**:

![Captura4](/img/djangoeclipse/captura4.png)

Volveremos a dar a **Next**  ye n la ultima pantalla  dejaremos como base de datos a usar sqlite3 y como versión de Django la 1.4 o superior. Es decir dejaremos los parámetros como aparecen por defecto para después  pulsar el botón **Finish**.

![Captura5](/img/djangoeclipse/captura5.png)

Ahora ya tenemos una instalación básica de **DJango**. Para probar si todo funciona bien vamos a ejecutarla.

Para ello pulsaremos sobre el botón  derecho del ratón encima del proyecto y dentro del menú *DJango* elegiremos la opción *Custom Command*

![Captura6](/img/djangoeclipse/captura6.png)

En la ventana que nos aparecerá solicitando el comando a ejecutar escribiremos **runserver**.

En la consola podremos ver que se nos ha levantado un servidor web, escuchando en el puerto 8000 de nuestra máquina (en localhost).

```
Django version 2.2.5, using settings 'djangoresttest.settings'
Starting development server at http://127.0.0.1:8000/
Quit the server with CTRL-BREAK.
```

Podemos ver que efectivamente el servidor esta funcionando si usando un navegador vamos a la dirección:

http://127.0.0.1:8000/ veremos la siguiente pantalla:

![Captura7](/img/djangoeclipse/captura7.png)



Las siguientes veces que deseemos ejecutar el servidor de **DJango** simplemente  iremos al menú `Run` de eclipse y elegiremos la nueva configuración que se nos habrá creado.

Y esto es todo por ahora. En un próximo articulo seguiré creando el servidor Rest usando Django.