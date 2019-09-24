---
title: Aplicación CRUD con DJango-REST.
weight: 20
pre: "<b>o </b>"
chapter: false
draft: true
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

Partiendo del [artículo anterior](/7-django/djangoeneclipse/), y usando  el esqueleto del proyecto  Eclipse creado, vamos a ver como hacer una pequeña aplicación **CRUD** con **Django-Rest**.

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

 ![captura1](..\..\static\img\djangocrud\captura1.png)

El árbol de directorios del proyecto será el siguiente:

![captura2](..\..\static\img\djangocrud\captura2.png)

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

Con el modelo creado vamos a decirle a DJango que cree las tablas correspondientes en la base de datos sqlite . Esto lo podemos hacer desde el propio eclipse eligiendo la opción `Django|Make Migrations` que nos aparecerá al pulsar con el botón derecho encima del proyecto.

![captura3](..\..\static\img\djangocrud\captura3.png)

En la ventana que nos saldrá preguntando por la app que migrar, pondremos `library`

Si todo va bien en la consola aparecerá algo parecido a esto: 

```
Migrations for 'library':
  library\migrations\0001_initial.py
    - Create model Books
Finished "C:\Users\usuario.DESKTOP-HF5D20U\Documents\eclipse-learning\djangoresttest\manage.py makemigrations library" execution.
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
Finished "C:\Users\usuario.DESKTOP-HF5D20U\Documents\eclipse-learning\djangoresttest\manage.py migrate" execution.
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

