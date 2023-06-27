# Primera aplicación con Django

## Parte 1: Solicitudes y respuestas.

Para comprobar que Django está instalado y que versión ejecutando:

~~~
python -m django --version 
~~~

### Creando un proyecto.

~~~
django-admin startproject mysite
~~~

Esto creará el directorio/proyecto mysite en la ruta que nos encontremos.

### El servidor de desarrollo.

Verificamos que el proyecto Django funciona desde el directorio raíz:

~~~
python manage.py runserver
~~~

Visitamos http://127.0.0.1:8000 para comprobarlo.

### Creando la aplicación Encuestas.

Para crear la aplicación, nos aseguramos de estar en el mismo directorio que **manage.py** y ejecutamos:

~~~
python manage.py startapp polls
~~~

#### Nuestra primera vista.

Abrimos el archivo **polls/views.py** y colocamos el siguiente código:

~~~
from django.http import HttpResponse


def index(request):
    return HttpResponse("Hello, world. You're at the polls index.")
~~~

Debemos asignarla a una URL, y para esto necesitamos una URLconf. En el directorio polls, creamos un nuevo archivo llamado **urls.py** que incluirá el siguiente código:

~~~
from django.urls import path

from . import views

urlpatterns = [
    path("", views.index, name="index"),
]
~~~

El siguiente paso es apuntar la raíz URLconf al **polls/urls.py**. En **mysite/urls.py**, importamos include e insertamos include() en la urlpatterns, por lo que tenemos:

~~~
from django.contrib import admin
from django.urls import include, path

urlpatterns = [
    path("polls/", include("polls.urls")),
    path("admin/", admin.site.urls),
]
~~~

Iniciamos el servidor y accedemos a http://localhost:8000/polls. Deberíamos ver el texto ("Hola, mundo") que definimos en la vista anterior.