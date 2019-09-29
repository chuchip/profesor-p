---
title: Aplicación CRUD con DJango-REST.
weight: 20
pre: "<b>o </b>"
chapter: false
draft: false
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

Partiendo del [artículo anterior](/7-django/djangoeneclipse/), y usando  el esqueleto del proyecto  Eclipse creado, vamos a ver como hacer una pequeña aplicación **CRUD** con **Django-Rest**. El código fuente esta disponible en [https://github.com/chuchip/djangorestcrud](https://github.com/chuchip/djangorestcrud)

Esta aplicación nos servirá para consultar, modificar, borrar o añadir los libros disponibles en una biblioteca.

El modelo de datos con el vamos a trabajar será el siguiente:

- title. Tipo: String. Titulo del libro.
- type. TIpo: String. Tipo libro (aventura, científico,comic, etc.)
- author -> String
- creation_date -> Date
- number_of_pages -> int 
- user. Tipo: String. Usuario al que se le presto el libro.
- borrow_date: Tipo Date. Fecha en la que se presto el libro. 

Para plasmar este modelo de datos en nuestro proyecto, primero debemos crear una aplicación. Para crearlo desde la línea de comandos, estando dentro del entorno de **djangoenv** y situándonos en el directorio donde creamos el proyecto, ejecutaremos este comando:

```bash
c:\> cd \python # Este directorio es donde creamos el entorno de trabajo
C:\python>djangoenv\Scripts\activate # Activamos el entorno de trabajo
C:\python>djangoenv\Scripts\activate > cd \DIR_PROYECTO # Moverse donde este el proyeco.
(djangoenv) \DIR_PROYECTO > python manage.py startapp library # Creamos la app
```

También lo podemos hacer de una manera más cómoda desde eclipse, pulsando con el botón derecho del ratón y eligiendo el menú `Django/Create Application`

 ![captura1](/img/djangocrud/captura1.png)

El árbol de directorios del proyecto será el siguiente:

![captura2](/img/djangocrud/captura2.png)

Ahora debemos especificar en el fichero de configuración de  DJango que tenemos la app `library`, aprovecharemos para indicar que también vamos a utilizar la app `rest_framework`. Para ello editaremos el fichero `settings.py `y en la variable `INSTALLED_APPS` añadiremos la app.

```
INSTALLED_APPS = [
    ...
    'rest_framework',
    'library.apps.LibraryConfig',
]
```

Observar que la línea añadida para definir la *app* se compone del nombre de la *app* más el nombre del fichero  (sin la extensión ".py") y la clase donde definimos la configuración de la aplicación.

De acuerdo, vamos a crear el modelo de datos. Para ello editaremos el fichero `library\models.py`, y escribiremos el siguiente código:

```
from django.db import models

class Books(models.Model):
    title = models.CharField('Titulo',max_length=100, blank=False)
    type = models.CharField('Tipo', max_length=40,blank=False)
    author = models.CharField('Autor', max_length=100,blank=False)
    creation_date = models.DateField('Fecha alta',null=False)
    number_of_pages=models.IntegerField(null=True)
    user=models.CharField(max_length=30,null=False)
    borrow_date= models.DateField('Fecha de prestamo',null=False)   
```

Como se puede ver, simplemente creamos una clase a la que llamamos `Books` y que extiende de `models.Model`. Dentro de la clase definimos variables  de los tipos que necesitamos, especificando además ciertas propiedades para cada campo. 

[Aquí](https://docs.djangoproject.com/en/2.2/topics/db/models/) tienes más información sobre los modelos de datos en Django

Con el modelo creado es hora de decirle a DJango que cree las tablas correspondientes en la base de datos sqlite . Esto lo podemos hacer desde el propio eclipse eligiendo la opción `Django|Make Migrations` que nos aparecerá al pulsar con el botón derecho encima del proyecto.

![captura3](/img/djangocrud/captura3.png)

En la ventana que nos saldrá preguntando por la app que migrar, pondremos `library`

Si todo va bien en la consola aparecerá algo parecido a esto: 

```
Migrations for 'library':
  library\migrations\0001_initial.py
    - Create model Books
Finished "....\manage.py makemigrations library" execution.
```

{{% notice note %}}

Esto seria el equivalente a ejecutar el comando  `python manage.py makemigrations library`

{{% /notice %}}

Lo que este comando ha creado ha sido un fichero en el directorio `library\migrations` con los comandos necesarios para crear una representación de nuestro modelo en la base de datos.

Para crear la estructura de tablas  en la base de datos volveremos a pulsar con el botón derecho del ratón sobre el proyecto y elegiremos la opción `Django|Migrate`

Lo cual nos deberá mostrar la siguiente salida:

```
Operations to perform:
  Apply all migrations: admin, auth, contenttypes, library, sessions
Running migrations:
  Applying library.0001_initial... OK
Finished "..\manage.py migrate" execution.
```

{{% notice note %}}

El usar esta opción en Eclipse seria el equivalente a ejecutar el comando  `python manage.py migrate` desde la línea de comandos.

{{% /notice %}}

Ahora que ya tendremos la estructura en la base de datos, vamos a crear una clase para serializar el modelo de datos, en el fichero `library/serializers.py`

```
from rest_framework import serializers
from library.models import Books

class LibrarySerializer(serializers.ModelSerializer):
    class Meta:
        model = Books
        fields = ['id', 'title', 'type', 'author', 'creation_date', 'number_of_pages','user','borrow_date']

```

Con esta simple clase definiremos como **DJango** tiene que convertir el modelo de datos a JSON y viceversa. Al heredar de `serializers.ModelSerializer` simplemente tendremos que especificar el modelo de datos a usar y los campos a presentar.  Si queremos personalizar como presentar los datos en la salida podríamos usar heredar de la clase **serializers.Serializer** pero en este tutorial vamos a intentar hacerlo sencillo.

Hasta ahora ya hemos definido los datos y como representarlos, ahora nos falta poder comunicarnos con el exterior a través de una API.  Eso es tan fácil como crear un fichero llamado `library/views.py` con el siguiente contenido.

```
from rest_framework import status
from rest_framework.decorators import api_view
from rest_framework.response import Response
from library.models import Books
from library.serializers import BooksSerializer

@api_view(['GET', 'POST'])
def book_list(request):
    """
    List all code Books, or create a new Book.
    """
    if request.method == 'GET':
        books = Books.objects.all()
        serializer = BooksSerializer(books, many=True)
        return Response(serializer.data)

    elif request.method == 'POST':
        serializer = BooksSerializer(data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data, status=status.HTTP_201_CREATED)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

```

Con la función `def book_list(request)`  será la utilizada para devolver todos los libros en la biblioteca o para insertar un nuevo libro.

Como se puede ver esta función aceptara peticiones GET y POST. Si la petición es GET buscara todos los libros dentro del modelo `Books`  llamando a la función `Books.objects.all()`, el cual devolverá una lista de instancias de la clase `Book`. Esa lista será pasada al constructor de la clase `BookSerializer`, añadiendo el parámetro ` many=True` por ser una lista de modelos y no un único modelo . Para terminar se devolverá un objeto Response, al que se le pasara los datos del objeto `serializer`.

 En caso de que la petición sea tipo POST, creara un *serializer* con los datos de la petición. Para ello simplemente pasara al crear el objeto tipo `BookSerializer` le pasara en el constructor los datos enviados en la petición (**request.data**).

Si los datos son validos, se guardaran en la base datos y se devolverán los datos insertados con el código HTTP 201. En el caso de que los datos no fueran validos, se devolverán los errores detectados y el codigo HTTP 400 (Bad_Request).

Antes de poder realizar ninguna prueba, en el fichero `library/urls.py` vamos a indicar la ruta que ejecutara la función:

```` 
from django.urls import path
from library import views

urlpatterns = [
    path('books/', views.book_list),   
]
````

Con esto indicamos a **DJango** que las peticiones a http://localhost:8000/books  deben ser tratadas en la función `book_list` de la clase `views`. Sin embargo todavía nos falta un paso y es definir en el fichero `djangorestcrud/urls.py` que tipo de peticiones deben ser dirigidas a la aplicación **library** . 

```
from django.contrib import admin
from django.urls import path,include

urlpatterns = [  
     path('', include('library.urls')),
]
```

Recordemos que **Django** funciona añadiendo aplicaciones al modulo principal y  cada aplicación tendrá su propia URL. En este caso estamos especificando que cualquier petición sea dirigida al modulo `library`. Si quisiéramos que solo se mandaran al modulo `library` las peticiones que contuvieran en el path la cadena **library** pondríamos la siguiente linea:  `path('library/', include('library.urls'))` 

Para poder hacer nuestra primera prueba, sin depender de ninguna herramienta externa a Python, vamos a usar la libreria **httpie** de Python. La instalaremos con el comando: 

```
pip install httpie
```

Para insertar nuestro primer libro, ejecutaremos la siguiente sentencia:

```
> http POST http://localhost:8000/books/ title="mi primer libro" type="Aventuras"    author="yo mismo"  "creation_date"="2019-09-27"    "number_of_pages"=15
HTTP/1.1 201 Created
Allow: POST, OPTIONS, GET
Content-Length: 155
Content-Type: application/json
Date: Sun, 29 Sep 2019 18:41:10 GMT
Server: WSGIServer/0.2 CPython/3.7.4
Vary: Accept, Cookie
X-Frame-Options: SAMEORIGIN

{
    "author": "yo mismo",
    "borrow_date": null,
    "creation_date": "2019-09-27",
    "id": 1,
    "number_of_pages": 15,
    "title": "mi primer libro",
    "type": "Aventuras",
    "user": null
}
```

Como se ve  la petición POST nos devuelve el registro insertado, incluyendo su id, y el código HTTP devuelto es un 201 (Created)

Esta sería la misma sentencia desde postman:

![Captura4](/img/djangocrud/captura4.png)



Podemos comprobar que el registro se ha insertado correctamente con la sentencia



```
> http GET http://localhost:8000/books/
HTTP/1.1 200 OK
Allow: POST, OPTIONS, GET
Content-Length: 1704
Content-Type: application/json
Date: Sun, 29 Sep 2019 18:42:56 GMT
Server: WSGIServer/0.2 CPython/3.7.4
Vary: Accept, Cookie
X-Frame-Options: SAMEORIGIN

{
        "author": "yo mismo",
        "borrow_date": null,
        "creation_date": "2019-09-27",
        "id": 1,
        "number_of_pages": 15,
        "title": "mi primer libro",
        "type": "Aventuras",
        "user": null
    },
```

Terminar diciendo que **DJango-Rest** dispone de una bonita interfaz para realizar peticiones desde el navegador para ello iremos a la misma URL de antes: http://localhost:8000/books/

![Captura5](/img/djangocrud/captura5.png)

En próximas entradas seguiré profundizando en esta aplicación, añadiéndole más funcionalidades.



