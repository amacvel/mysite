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

## Parte 2: Modelos y el sitio de administración.

### Configuración de la base de datos.

Abrimos **mysite/settings.py**. De forma predeterminada, la configuración utiliza SQLite. Establecemos nuestra zona horaria a:

~~~
TIME_ZONE = 'Atlantic/Canary'
~~~

**INSTALLED_APPS** contiene los nombres de todas las aplicaciones que están activadas en esta instancia de Django.

Algunas de estas aplicaciones utilizan al menos una tabla de la base de datos, por lo que debemos crearlas antes de poder usarlas. Ejecutamos el siguiente comando:

~~~
python manage.py migrate
~~~

### Creando modelos.

En nuestra aplicación de encuentas, crearemos dos modelos: **Question** y **Choice**. Editamos el **polls/models.py**:

~~~
from django.db import models


class Question(models.Model):
    question_text = models.CharField(max_length=200)
    pub_date = models.DateTimeField("date published")


class Choice(models.Model):
    question = models.ForeignKey(Question, on_delete=models.CASCADE)
    choice_text = models.CharField(max_length=200)
    votes = models.IntegerField(default=0)
~~~

### Activando modelos.

Para incluir la aplicación 'polls' a nuestro proyecto debemos agregar una referencia. Editamos **mysite/settings.py** y añadimos la ruta punteada en **INSTALLED_APPS**. Se verá tal que así:

~~~
INSTALLED_APPS = [
    "polls.apps.PollsConfig",
    "django.contrib.admin",
    "django.contrib.auth",
    "django.contrib.contenttypes",
    "django.contrib.sessions",
    "django.contrib.messages",
    "django.contrib.staticfiles",
]
~~~

Ejecutamos otro comando:

~~~
python manage.py makemigrations polls
~~~

Deberíamos ver algo similar a lo siguiente:

~~~
Migrations for 'polls':
  polls/migrations/0001_initial.py
    - Create model Question
    - Create model Choice
~~~

Al ejecutar makemigrations, le estamos diciendo a Django que hemos realizado algunos cambios en sus modelos (en este caso, hemos realizado otros nuevos) y que desea que los cambios se almacenen como una migración.

El comando sqlmigrate toma nombres de migración y devuelve su SQL:

~~~
python manage.py sqlmigrate polls 0001
~~~

Deberíamos ver algo similar a lo siguiente:

~~~
BEGIN;
--
-- Create model Question
--
CREATE TABLE "polls_question" (
    "id" bigint NOT NULL PRIMARY KEY GENERATED BY DEFAULT AS IDENTITY,
    "question_text" varchar(200) NOT NULL,
    "pub_date" timestamp with time zone NOT NULL
);
--
-- Create model Choice
--
CREATE TABLE "polls_choice" (
    "id" bigint NOT NULL PRIMARY KEY GENERATED BY DEFAULT AS IDENTITY,
    "choice_text" varchar(200) NOT NULL,
    "votes" integer NOT NULL,
    "question_id" bigint NOT NULL
);
ALTER TABLE "polls_choice"
  ADD CONSTRAINT "polls_choice_question_id_c5b4b260_fk_polls_question_id"
    FOREIGN KEY ("question_id")
    REFERENCES "polls_question" ("id")
    DEFERRABLE INITIALLY DEFERRED;
CREATE INDEX "polls_choice_question_id_c5b4b260" ON "polls_choice" ("question_id");

COMMIT;
~~~

Ahora, volvemos a ejecutar migrate para crear las tablas modelo en la base de datos:

~~~
python manage.py migrate
Operations to perform:
  Apply all migrations: admin, auth, contenttypes, polls, sessions
Running migrations:
  Rendering model states... DONE
  Applying polls.0001_initial... OK
~~~

Resumen de los tres pasos para realizar cambios en el modelo:
- Cambios en los modelos (en **models.py**).
- Ejecutar para crear migraciones para esos cambios **python manage.py makemigrations**
- Ejecutar para aplicar esos cambios a la base de datos **python manage.py migrate**

### Jugando con la API.

Invocamos el shell de Python:

~~~
python manage.py shell
~~~

~~~
>>> from polls.models import Choice, Question  # Import the model classes we just wrote.

# No questions are in the system yet.
>>> Question.objects.all()
<QuerySet []>

# Create a new Question.
# Support for time zones is enabled in the default settings file, so
# Django expects a datetime with tzinfo for pub_date. Use timezone.now()
# instead of datetime.datetime.now() and it will do the right thing.
>>> from django.utils import timezone
>>> q = Question(question_text="What's new?", pub_date=timezone.now())

# Save the object into the database. You have to call save() explicitly.
>>> q.save()

# Now it has an ID.
>>> q.id
1

# Access model field values via Python attributes.
>>> q.question_text
"What's new?"
>>> q.pub_date
datetime.datetime(2012, 2, 26, 13, 0, 0, 775217, tzinfo=datetime.timezone.utc)

# Change values by changing the attributes, then calling save().
>>> q.question_text = "What's up?"
>>> q.save()

# objects.all() displays all the questions in the database.
>>> Question.objects.all()
<QuerySet [<Question: Question object (1)>]>
~~~

No es una representación útil de este objeto. Editaremos el modelo y agregaremos un método a ambos en **polls/models.py**:

~~~
from django.db import models


class Question(models.Model):
    # ...
    def __str__(self):
        return self.question_text


class Choice(models.Model):
    # ...
    def __str__(self):
        return self.choice_text
~~~

Agregamos también un método personalizado a este modelo:

~~~
import datetime

from django.db import models
from django.utils import timezone


class Question(models.Model):
    # ...
    def was_published_recently(self):
        return self.pub_date >= timezone.now() - datetime.timedelta(days=1)
~~~

Guardamos los cambios e iniciamos un nuevo shell:

~~~
>>> from polls.models import Choice, Question

# Make sure our __str__() addition worked.
>>> Question.objects.all()
<QuerySet [<Question: What's up?>]>

# Django provides a rich database lookup API that's entirely driven by
# keyword arguments.
>>> Question.objects.filter(id=1)
<QuerySet [<Question: What's up?>]>
>>> Question.objects.filter(question_text__startswith="What")
<QuerySet [<Question: What's up?>]>

# Get the question that was published this year.
>>> from django.utils import timezone
>>> current_year = timezone.now().year
>>> Question.objects.get(pub_date__year=current_year)
<Question: What's up?>

# Request an ID that doesn't exist, this will raise an exception.
>>> Question.objects.get(id=2)
Traceback (most recent call last):
    ...
DoesNotExist: Question matching query does not exist.

# Lookup by a primary key is the most common case, so Django provides a
# shortcut for primary-key exact lookups.
# The following is identical to Question.objects.get(id=1).
>>> Question.objects.get(pk=1)
<Question: What's up?>

# Make sure our custom method worked.
>>> q = Question.objects.get(pk=1)
>>> q.was_published_recently()
True

# Give the Question a couple of Choices. The create call constructs a new
# Choice object, does the INSERT statement, adds the choice to the set
# of available choices and returns the new Choice object. Django creates
# a set to hold the "other side" of a ForeignKey relation
# (e.g. a question's choice) which can be accessed via the API.
>>> q = Question.objects.get(pk=1)

# Display any choices from the related object set -- none so far.
>>> q.choice_set.all()
<QuerySet []>

# Create three choices.
>>> q.choice_set.create(choice_text="Not much", votes=0)
<Choice: Not much>
>>> q.choice_set.create(choice_text="The sky", votes=0)
<Choice: The sky>
>>> c = q.choice_set.create(choice_text="Just hacking again", votes=0)

# Choice objects have API access to their related Question objects.
>>> c.question
<Question: What's up?>

# And vice versa: Question objects get access to Choice objects.
>>> q.choice_set.all()
<QuerySet [<Choice: Not much>, <Choice: The sky>, <Choice: Just hacking again>]>
>>> q.choice_set.count()
3

# The API automatically follows relationships as far as you need.
# Use double underscores to separate relationships.
# This works as many levels deep as you want; there's no limit.
# Find all Choices for any question whose pub_date is in this year
# (reusing the 'current_year' variable we created above).
>>> Choice.objects.filter(question__pub_date__year=current_year)
<QuerySet [<Choice: Not much>, <Choice: The sky>, <Choice: Just hacking again>]>

# Let's delete one of the choices. Use delete() for that.
>>> c = q.choice_set.filter(choice_text__startswith="Just hacking")
>>> c.delete()
~~~

### Presentando Django Admin.

Primero necesitamos crear un usuario para el sitio de administración. Ejecutamos:

~~~
python manage.py createsuperuser
~~~

Iniciamos el servidor de desarrollo:

~~~
python manage.py runserver
~~~

Abrimos un navegador web y vamos a http://127.0.0.1:8000/admin. Deberíamos ver la pantalla de inicio de sesión del administrador.

Iniciamos sesión con la cuenta de superusuario que creamos anteriormente.

Debemos hacer que nuestra aplicación de encuentas sea modificable en el administrador, para ello vamos a **polls/admin.py** y editamos lo siguiente:

~~~
from django.contrib import admin

from .models import Question

admin.site.register(Question)
~~~

Ahora que lo hemos registrado, Django sabe que debe mostrarse en la página de índice de la administración.

## Parte 3: Vistas y plantillas.

### Escribiendo más vistas.

Agregamos algunas vistas más en **polls/views.py**:

~~~
def detail(request, question_id):
    return HttpResponse("You're looking at question %s." % question_id)


def results(request, question_id):
    response = "You're looking at the results of question %s."
    return HttpResponse(response % question_id)


def vote(request, question_id):
    return HttpResponse("You're voting on question %s." % question_id)
~~~

Conectamos las vistas al **polls.urls** agregando las siguientes path():

~~~
from django.urls import path

from . import views

urlpatterns = [
    # ex: /polls/
    path("", views.index, name="index"),
    # ex: /polls/5/
    path("<int:question_id>/", views.detail, name="detail"),
    # ex: /polls/5/results/
    path("<int:question_id>/results/", views.results, name="results"),
    # ex: /polls/5/vote/
    path("<int:question_id>/vote/", views.vote, name="vote"),
]
~~~

### Escribir vistas que realmente hagan algo.

Todo lo que Django quiere es un HttpResponse, o una excepción.

Mostramos las últimas 5 preguntas de la encuesta en el sistema, separadas por comas, según la fecha de publicación:

~~~
from django.http import HttpResponse

from .models import Question


def index(request):
    latest_question_list = Question.objects.order_by("-pub_date")[:5]
    output = ", ".join([q.question_text for q in latest_question_list])
    return HttpResponse(output)


# Leave the rest of the views (detail, results, vote) unchanged
~~~

Crearemos un directorio llamado templates dentro del directorio polls. Dentro de templates, otro directorio llamado polls y dentro un archivo llamado index.html con el siguiente código:

~~~
{% if latest_question_list %}
    <ul>
    {% for question in latest_question_list %}
        <li><a href="/polls/{{ question.id }}/">{{ question.question_text }}</a></li>
    {% endfor %}
    </ul>
{% else %}
    <p>No polls are available.</p>
{% endif %}
~~~

Actualizamos nuestra vista index **polls/views.py** para usar la plantilla:

~~~
from django.http import HttpResponse
from django.template import loader

from .models import Question


def index(request):
    latest_question_list = Question.objects.order_by("-pub_date")[:5]
    template = loader.get_template("polls/index.html")
    context = {
        "latest_question_list": latest_question_list,
    }
    return HttpResponse(template.render(context, request))
~~~

### El atajo render().

Es un modismo muy común cargar una plantilla, llenar un contexto y devolver un HttpResponse con el resultado de la plantilla renderizada. Django proporciona un atajo. Aquí está la vista index() completa, reescrita:

~~~
from django.shortcuts import render

from .models import Question


def index(request):
    latest_question_list = Question.objects.order_by("-pub_date")[:5]
    context = {"latest_question_list": latest_question_list}
    return render(request, "polls/index.html", context)
~~~

### Generando un error 404.

Abordaremos ahora la vista details de la pregunta. Aquí la vista:

~~~
from django.http import Http404
from django.shortcuts import render

from .models import Question


# ...
def detail(request, question_id):
    try:
        question = Question.objects.get(pk=question_id)
    except Question.DoesNotExist:
        raise Http404("Question does not exist")
    return render(request, "polls/detail.html", {"question": question})
~~~

Si deseamos que el ejemplo anterior funcione rápidamente, crearemos el archivo **detail.html** que contendrá sólo:

~~~
{{ question }}
~~~

### El atajo get_object_or_404().

Es un modismo muy común para usar get() y plantear Http404 si el objeto no existe. Django proporciona un atajo. Aquí está la vista detail(), reescrita:

~~~
from django.shortcuts import get_object_or_404, render

from .models import Question


# ...
def detail(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    return render(request, "polls/detail.html", {"question": question})
~~~

### Usar el sistema de plantillas.

Así se vería la plantilla **detail.html**:

~~~
<h1>{{ question.question_text }}</h1>
<ul>
{% for choice in question.choice_set.all %}
    <li>{{ choice.choice_text }}</li>
{% endfor %}
</ul>
~~~

### Eliminación de URL codificadas en plantillas.

Cuando escribimos el enlace a una pregunta en la plantilla **polls/index.html**, el enlace estaba parcialmente codificado de esta manera:

~~~
<li><a href="/polls/{{ question.id }}/">{{ question.question_text }}</a></li>
~~~

Sin embargo, dado que definimos el argumento de nombre en las funciones path() del módulo polls.urls, podemos eliminar la dependencia de rutas de URL específicas definidas en sus configuraciones de URL utilizando la etiqueta de plantilla:{% url %}

~~~
<li><a href="{% url 'detail' question.id %}">{{ question.question_text }}</a></li>
~~~

### Nombres de URL de espacio de nombres.

En el archivo **polls/urls.py**, agregamos un app_name para configurar el espacio de nombres de la aplicación:

~~~
from django.urls import path

from . import views

app_name = "polls"
urlpatterns = [
    path("", views.index, name="index"),
    path("<int:question_id>/", views.detail, name="detail"),
    path("<int:question_id>/results/", views.results, name="results"),
    path("<int:question_id>/vote/", views.vote, name="vote"),
]
~~~

Ahora editamos la plantilla **polls/index.html** a:

~~~
<li><a href="{% url 'polls:detail' question.id %}">{{ question.question_text }}</a></li>
~~~

## Parte 4: Formularios y vistas genéricas.

### Escribe un formulario.

Actualizamos nuestra plantilla **polls/detail.html** para que contenga el siguiente formulario:

~~~
<form action="{% url 'polls:vote' question.id %}" method="post">
{% csrf_token %}
<fieldset>
    <legend><h1>{{ question.question_text }}</h1></legend>
    {% if error_message %}<p><strong>{{ error_message }}</strong></p>{% endif %}
    {% for choice in question.choice_set.all %}
        <input type="radio" name="choice" id="choice{{ forloop.counter }}" value="{{ choice.id }}">
        <label for="choice{{ forloop.counter }}">{{ choice.choice_text }}</label><br>
    {% endfor %}
</fieldset>
<input type="submit" value="Vote">
</form>
~~~

Ahora, crearemos una vista de Django que maneje los datos enviados y haga algo con ellos. También crearemos la función vote(). Agregamos lo siguiente a **polls/views.py**:

~~~
from django.http import HttpResponse, HttpResponseRedirect
from django.shortcuts import get_object_or_404, render
from django.urls import reverse

from .models import Choice, Question


# ...
def vote(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    try:
        selected_choice = question.choice_set.get(pk=request.POST["choice"])
    except (KeyError, Choice.DoesNotExist):
        # Redisplay the question voting form.
        return render(
            request,
            "polls/detail.html",
            {
                "question": question,
                "error_message": "You didn't select a choice.",
            },
        )
    else:
        selected_choice.votes += 1
        selected_choice.save()
        # Always return an HttpResponseRedirect after successfully dealing
        # with POST data. This prevents data from being posted twice if a
        # user hits the Back button.
        return HttpResponseRedirect(reverse("polls:results", args=(question.id,)))
~~~

Después de que alguien vota en una pregunta, vote() redirige a la página de resultados de la pregunta. Escribimos esa vista:

~~~
from django.shortcuts import get_object_or_404, render


def results(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    return render(request, "polls/results.html", {"question": question})
~~~

Ahora, creamos la plantilla **polls/results.html**:

~~~
<h1>{{ question.question_text }}</h1>

<ul>
{% for choice in question.choice_set.all %}
    <li>{{ choice.choice_text }} -- {{ choice.votes }} vote{{ choice.votes|pluralize }}</li>
{% endfor %}
</ul>

<a href="{% url 'polls:detail' question.id %}">Vote again?</a>
~~~

Ahora, vamos a /polls/1/ en el navegador y votamos en la pregunta. Deberíamos ver una página de resultados que se actualiza cada vez que vota. Si enviamos el formulario sin haber elegido una opción, deberíamos ver el mensaje de error.

### Usar vistas genéricas: Menos código es mejor.

Convertiremos nuestra aplicación de encuestas para usar el sistema de vistas genéricas, de modo que podamos eliminar un montón de nuestro propio código. Tendremos que dar algunos pasos para hacer la conversión. Haremos lo siguiente:

- Convertimos la URLconf.
- Eliminamos algunas de las vistas antiguas e innecesarias.
- Introducimos nuevas vistas basadas en las vistas genéricas de Django.

### Modificar URLconf.

Abrimos **polls/urls.py y modificamos:

~~~
from django.urls import path

from . import views

app_name = "polls"
urlpatterns = [
    path("", views.IndexView.as_view(), name="index"),
    path("<int:pk>/", views.DetailView.as_view(), name="detail"),
    path("<int:pk>/results/", views.ResultsView.as_view(), name="results"),
    path("<int:question_id>/vote/", views.vote, name="vote"),
]
~~~

### Modificar vistas.

A continuación, eliminamos nuestras antiguas vistas index, detail y results. Usaremos las vistas genéricas de Django en su lugar. Para hacerlo, abrimos el **polls/views.py**  y cambiamos lo siguiente:

~~~
from django.http import HttpResponseRedirect
from django.shortcuts import get_object_or_404, render
from django.urls import reverse
from django.views import generic

from .models import Choice, Question


class IndexView(generic.ListView):
    template_name = "polls/index.html"
    context_object_name = "latest_question_list"

    def get_queryset(self):
        """Return the last five published questions."""
        return Question.objects.order_by("-pub_date")[:5]


class DetailView(generic.DetailView):
    model = Question
    template_name = "polls/detail.html"


class ResultsView(generic.DetailView):
    model = Question
    template_name = "polls/results.html"


def vote(request, question_id):
    ...  # same as above, no changes needed.
~~~

Estamos usando dos vistas genéricas: ListViewy DetailView. Respectivamente, esas dos vistas resumen los conceptos de "mostrar una lista de objetos" y "mostrar una página de detalles para un tipo particular de objeto".