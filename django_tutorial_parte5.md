# Django Tutorial 4 — Formularios y Vistas Genéricas

> Basado en la documentación oficial de Django 6.0 — [Tutorial Part 4](https://docs.djangoproject.com/en/6.0/intro/tutorial04/)

---

## Índice

1. [Crear el formulario de votación](#1-crear-el-formulario-de-votación)
   - [El template `detail.html`](#el-template-detailhtml)
   - [La vista `vote()` en `views.py`](#la-vista-vote-en-viewspy)
   - [La vista `results()` y su template](#la-vista-results-y-su-template)
2. [Vistas Genéricas — Menos código, más convención](#2-vistas-genéricas--menos-código-más-convención)
   - [Paso 1: Actualizar `urls.py`](#paso-1-actualizar-urlspy)
   - [Paso 2: Reescribir `views.py` con CBV](#paso-2-reescribir-viewspy-con-cbv)
3. [PD — Cómo funcionan las Generic Views por dentro](#3-pd--cómo-funcionan-las-generic-views-por-dentro)
   - [Todas las vistas genéricas disponibles](#todas-las-vistas-genéricas-disponibles)
   - [Qué pasa por dentro cuando Django ejecuta una CBV](#qué-pasa-por-dentro-cuando-django-ejecuta-una-cbv)
   - [Métodos útiles para sobreescribir](#métodos-útiles-para-sobreescribir)
   - [¿Cuándo usar CBV y cuándo una función?](#cuándo-usar-cbv-y-cuándo-una-función)
4. [Buenas prácticas y comparación con la doc oficial](#4-buenas-prácticas-y-comparación-con-la-doc-oficial)

---

## 1. Crear el formulario de votación

### El template `detail.html`

**Archivo:** `polls/templates/polls/detail.html`

```html
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
```

#### Desglose línea por línea

**`<form action="{% url 'polls:vote' question.id %}" method="post">`**

- `action` define a dónde se manda el formulario cuando el usuario hace clic en "Vote". En vez de hardcodear `/polls/3/vote/`, usamos la tag `{% url %}` que genera la URL dinámicamente desde el nombre de la ruta (`polls:vote`) y el ID de la pregunta.
- Si algún día cambiás la URL en `urls.py`, no tenés que tocar los templates — esto es una **buena práctica fundamental**.
- `method="post"` es crucial: **toda operación que modifica datos en el servidor debe usar POST, nunca GET**. GET es para consultas (leer datos), POST es para acciones (escribir, modificar, eliminar). Si usaras GET, la votación quedaría expuesta en la URL, podría guardarse en el historial del navegador y sería fácil de repetir o falsificar.

**`{% csrf_token %}`**

- CSRF significa *Cross-Site Request Forgery* (Falsificación de Petición entre Sitios). El ataque consiste en que una página maliciosa engaña a tu navegador para que envíe un formulario a tu sitio sin que el usuario lo sepa.
- Django lo previene generando un token secreto único por sesión. Este tag lo inserta como campo oculto en el formulario. Cuando llega el POST, Django verifica que el token esté presente y sea válido. Si no está → rechaza la petición con un error 403.
- **Regla de oro:** todo formulario POST en Django debe tener `{% csrf_token %}`.

**`{% for choice in question.choice_set.all %}`**

- `question.choice_set` es el *related manager* que Django crea automáticamente para acceder a las `Choice` relacionadas con esa `Question`. Es equivalente a `Choice.objects.filter(question=question)`, pero más conveniente.
- `.all` trae todos los registros.

**`<input type="radio" name="choice" id="choice{{ forloop.counter }}" value="{{ choice.id }}">`**

- `name="choice"`: todos los radio buttons tienen el mismo `name`. El navegador garantiza que solo uno puede estar seleccionado a la vez. Cuando se envía el form, se manda un solo par `choice=<id>` en el POST.
- `value="{{ choice.id }}"`: lo que Django va a recibir en `request.POST['choice']` es el ID de la opción seleccionada.
- `forloop.counter`: variable especial del template que Django provee dentro de un `{% for %}`. Empieza en 1 y se incrementa cada iteración. La usamos para generar IDs únicos como `choice1`, `choice2`, etc., necesarios para que los `<label>` funcionen correctamente.

---

### La vista `vote()` en `views.py`

**Archivo:** `polls/views.py`

```python
from django.db.models import F
from django.http import HttpResponse, HttpResponseRedirect
from django.shortcuts import get_object_or_404, render
from django.urls import reverse

from .models import Choice, Question


def vote(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    try:
        selected_choice = question.choice_set.get(pk=request.POST["choice"])
    except (KeyError, Choice.DoesNotExist):
        # Vuelve a mostrar el formulario con mensaje de error
        return render(
            request,
            "polls/detail.html",
            {
                "question": question,
                "error_message": "You didn't select a choice.",
            },
        )
    else:
        selected_choice.votes = F("votes") + 1
        selected_choice.save()
        # Siempre redirigir después de procesar un POST exitoso
        return HttpResponseRedirect(reverse("polls:results", args=(question.id,)))
```

#### Desglose

**`get_object_or_404(Question, pk=question_id)`**

Busca la pregunta por su clave primaria. Si no existe, devuelve automáticamente un error 404. Siempre preferir esto sobre un `try/except` manual con `Question.objects.get()`.

**`request.POST["choice"]`**

- `request.POST` es un diccionario (técnicamente un `QueryDict`) con los datos enviados en el formulario.
- Los valores de POST **siempre son strings**, incluso si el valor original era un número.
- También existe `request.GET` para parámetros de URL (`?page=2`), pero acá usamos POST explícitamente para garantizar que los votos solo se registren desde el formulario, no desde una URL directa.

**`try / except (KeyError, Choice.DoesNotExist)`**

Se manejan dos casos de error:
- `KeyError`: el usuario envió el formulario sin seleccionar ninguna opción (la clave `"choice"` no está en `request.POST`).
- `Choice.DoesNotExist`: alguien manipuló el formulario y mandó un ID de choice que no existe.

En ambos casos, volvemos a mostrar el formulario con un mensaje de error que el template ya sabe cómo mostrar gracias a `{% if error_message %}`.

**`selected_choice.votes = F("votes") + 1`** ⭐ Race condition

Esta es una de las partes más importantes. `F("votes")` es una **expresión F** de Django.

```python
# ❌ MAL — tiene race condition
selected_choice.votes += 1
selected_choice.save()

# ✅ BIEN — operación atómica en la base de datos
selected_choice.votes = F("votes") + 1
selected_choice.save()
```

**¿Por qué importa?** Con `+= 1`, Django primero lee el valor actual de la base de datos, lo suma en Python, y luego lo escribe. Si dos usuarios votan al mismo tiempo, ambos leen el mismo valor (digamos `5`), ambos calculan `6`, y ambos escriben `6`. El resultado es que se registra solo 1 voto en vez de 2. Esto se llama **race condition**.

Con `F("votes") + 1`, Django genera SQL del tipo `UPDATE SET votes = votes + 1`, que la base de datos ejecuta atómicamente. No hay problema de concurrencia.

**`return HttpResponseRedirect(reverse("polls:results", args=(question.id,)))`**

Dos cosas importantes:

1. **Siempre redirigir después de un POST exitoso.** Si el usuario aprieta F5 (recargar) después de votar, el navegador le va a preguntar si quiere reenviar el formulario. Si ya redirigiste a una página GET, recargar solo re-ejecuta esa consulta GET, no el voto. Este patrón se llama **Post/Redirect/Get (PRG)** y es un estándar del desarrollo web.

2. **`reverse()`** genera la URL a partir del nombre de la vista en vez de hardcodearla. `reverse("polls:results", args=(question.id,))` produce algo como `"/polls/3/results/"`. Si cambiás la URL en `urls.py`, `reverse()` sigue funcionando sin tocar las vistas.

---

### La vista `results()` y su template

**Archivo:** `polls/views.py`

```python
def results(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    return render(request, "polls/results.html", {"question": question})
```

**Archivo:** `polls/templates/polls/results.html`

```html
<h1>{{ question.question_text }}</h1>

<ul>
{% for choice in question.choice_set.all %}
    <li>{{ choice.choice_text }} -- {{ choice.votes }} vote{{ choice.votes|pluralize }}</li>
{% endfor %}
</ul>

<a href="{% url 'polls:detail' question.id %}">Vote again?</a>
```

El filtro `|pluralize` es un helper del sistema de templates de Django: si `choice.votes` es 1, no agrega nada; si es cualquier otro número, agrega la letra `s`. Entonces muestra `"1 vote"` o `"3 votes"` automáticamente.

---

## 2. Vistas Genéricas — Menos código, más convención

### ¿Por qué existen?

Django notó que muchas vistas siguen exactamente el mismo patrón:

1. Recibir un parámetro de la URL (como un ID)
2. Buscar un objeto en la base de datos
3. Pasarlo a un template
4. Devolver el HTML renderizado

Escribir eso una y otra vez es repetitivo. Django lo resuelve con las **Class-Based Views (CBV)** genéricas: clases prehechas que implementan ese patrón, y vos solo sobreescribís lo que necesitás personalizar.

> **Nota:** El tutorial enseñó primero las *function-based views* (vistas como funciones) a propósito, para que entiendas qué está pasando por dentro. Las vistas genéricas hacen lo mismo pero automáticamente. Conocer la base te permite entender y debuggear las genéricas.

---

### Paso 1: Actualizar `urls.py`

**Archivo:** `polls/urls.py`

```python
from django.urls import path
from . import views

app_name = "polls"
urlpatterns = [
    path("", views.IndexView.as_view(), name="index"),
    path("<int:pk>/", views.DetailView.as_view(), name="detail"),
    path("<int:pk>/results/", views.ResultsView.as_view(), name="results"),
    path("<int:question_id>/vote/", views.vote, name="vote"),
]
```

**Cambios clave:**

- `<question_id>` cambió a `<pk>` en las rutas de `detail` y `results`. Esto es **obligatorio** para `DetailView`: esa clase genérica espera que el parámetro de la URL se llame `pk` (primary key). Si lo dejás como `question_id`, no va a encontrar el objeto y va a fallar con un 404 silencioso.
- `.as_view()`: las CBV no son funciones sino clases. `.as_view()` es un método de clase que devuelve una función callable que Django puede usar como view. Es el "adaptador" entre el sistema de URLs de Django (que espera funciones) y las clases.

---

### Paso 2: Reescribir `views.py` con CBV

**Archivo:** `polls/views.py`

```python
from django.db.models import F
from django.http import HttpResponseRedirect
from django.shortcuts import get_object_or_404, render
from django.urls import reverse
from django.views import generic

from .models import Choice, Question


class IndexView(generic.ListView):
    template_name = "polls/index.html"
    context_object_name = "latest_question_list"

    def get_queryset(self):
        """Devuelve las últimas 5 preguntas publicadas."""
        return Question.objects.order_by("-pub_date")[:5]


class DetailView(generic.DetailView):
    model = Question
    template_name = "polls/detail.html"


class ResultsView(generic.DetailView):
    model = Question
    template_name = "polls/results.html"


def vote(request, question_id):
    # Sin cambios respecto a la versión anterior
    ...
```

#### Desglose de cada clase

**`IndexView(generic.ListView)`**

`ListView` está diseñada para mostrar una lista de objetos.

- Por defecto buscaría el template `polls/question_list.html`, pero sobreescribimos `template_name` para usar el que ya teníamos.
- `context_object_name`: por defecto la variable en el template se llamaría `question_list`, pero nuestro template usa `latest_question_list`. Lo sobreescribimos para mantener la compatibilidad.
- `get_queryset()` sobreescribe el método que define qué objetos mostrar. Acá filtramos y limitamos a 5.

**`DetailView(generic.DetailView)`**

`DetailView` muestra el detalle de un objeto individual.

- Necesita saber el modelo (`model = Question`) y usa `pk` de la URL para buscarlo con `get_object_or_404` automáticamente.
- Sin `template_name`, buscaría `polls/question_detail.html`. Lo sobreescribimos para apuntar a `polls/detail.html`.
- El objeto se pasa al template automáticamente como `question` (Django infiere el nombre en minúsculas del modelo).

**`ResultsView(generic.DetailView)`**

Idéntica a `DetailView` en comportamiento. La única diferencia es el template que usa. Aunque parezca raro reutilizar `DetailView` para esto, tiene sentido: ambas muestran el detalle de una `Question`, solo que con presentaciones distintas.

---

## 3. PD — Cómo funcionan las Generic Views por dentro

### Todas las vistas genéricas disponibles

Las vistas genéricas de Django se dividen en tres grupos:

#### Grupo 1 — Utilidades simples (no necesitan modelo)

| Clase | Para qué sirve | Ejemplo de uso |
|---|---|---|
| `TemplateView` | Solo renderiza un template | Páginas estáticas: "About", "Home", "FAQ" |
| `RedirectView` | Solo redirige a otra URL | URLs viejas que migraron, aliases |

```python
# Ejemplo TemplateView
class AboutView(generic.TemplateView):
    template_name = "about.html"

# Ejemplo RedirectView
class OldPollsView(generic.RedirectView):
    url = "/polls/"  # o pattern_name = "polls:index"
```

#### Grupo 2 — Display (solo muestran datos)

| Clase | Para qué sirve | Ejemplo de uso |
|---|---|---|
| `ListView` | Lista de objetos del mismo tipo | Lista de preguntas, lista de artículos |
| `DetailView` | Detalle de un objeto individual por `pk` | Ver una pregunta, ver un artículo |

#### Grupo 3 — Edición (trabajan con formularios y modelos)

| Clase | Para qué sirve | Ejemplo de uso |
|---|---|---|
| `CreateView` | Muestra formulario para crear un objeto nuevo | Crear una pregunta nueva |
| `UpdateView` | Formulario pre-poblado para editar un objeto | Editar el texto de una pregunta |
| `DeleteView` | Página de confirmación para eliminar | Borrar una pregunta |
| `FormView` | Formulario sin modelo específico | Formulario de contacto, búsqueda |

```python
# Ejemplo de las vistas de edición
from django.urls import reverse_lazy

class QuestionCreateView(generic.CreateView):
    model = Question
    fields = ["question_text", "pub_date"]
    template_name = "polls/question_form.html"
    success_url = reverse_lazy("polls:index")  # a dónde redirigir al guardar

class QuestionUpdateView(generic.UpdateView):
    model = Question
    fields = ["question_text"]
    template_name = "polls/question_form.html"
    success_url = reverse_lazy("polls:index")

class QuestionDeleteView(generic.DeleteView):
    model = Question
    template_name = "polls/question_confirm_delete.html"
    success_url = reverse_lazy("polls:index")
```

---

### Qué pasa por dentro cuando Django ejecuta una CBV

Cuando Django recibe un request a una URL que apunta a una CBV, el flujo interno es:

```
URL llega al router
       ↓
MiVista.as_view()     ← devuelve una función callable
       ↓
dispatch(request)     ← decide si llamar get() o post() según el verbo HTTP
       ↓
   get() o post()     ← la lógica principal de la vista
       ↓
render_to_response()  ← arma y devuelve la respuesta HTTP
```

El método central que no se ve en el tutorial es `dispatch()`. Internamente hace algo equivalente a:

```python
def dispatch(self, request, *args, **kwargs):
    if request.method.lower() in self.http_method_names:
        # Busca dinámicamente el método: self.get(), self.post(), etc.
        handler = getattr(self, request.method.lower())
        return handler(request, *args, **kwargs)
```

Es decir, `dispatch` es quien "despacha" el request al método correcto según el verbo HTTP. Cuando sobreescribís `get_queryset()` o `get_context_data()`, estás interceptando el flujo en un punto específico sin tener que reescribir todo.

---

### Métodos útiles para sobreescribir

Estos son los puntos de extensión más comunes:

```python
class MiVista(generic.ListView):

    def get_queryset(self):
        """Cambiar QUÉ objetos se traen de la base de datos."""
        return Question.objects.filter(pub_date__lte=timezone.now())

    def get_context_data(self, **kwargs):
        """Agregar variables extra al template."""
        context = super().get_context_data(**kwargs)
        context["categorias"] = Categoria.objects.all()
        context["usuario_activo"] = self.request.user
        return context

    def get_template_names(self):
        """Cambiar el template dinámicamente según alguna condición."""
        if self.request.user.is_staff:
            return ["polls/index_admin.html"]
        return ["polls/index.html"]
```

> **Nota sobre `get_context_data`:** siempre llamá a `super().get_context_data(**kwargs)` primero para no perder el contexto que la clase padre ya agregó (como el propio objeto o la lista de objetos).

Para explorar el código fuente de Django directamente en tu entorno:

```bash
# Con el venv activado, te muestra la ruta del archivo
python -c "import django.views.generic; print(django.views.generic.__file__)"
```

---

### ¿Cuándo usar CBV y cuándo una función?

No hay una respuesta única, pero la guía práctica es:

| Situación | Recomendación |
|---|---|
| Mostrar lista de objetos | `ListView` |
| Mostrar detalle de un objeto | `DetailView` |
| Crear, editar, eliminar con formulario | `CreateView`, `UpdateView`, `DeleteView` |
| Lógica de votación, procesamiento complejo | Vista como función (`def vote`) |
| Formulario que no corresponde a un modelo | `FormView` |
| Página estática sin datos | `TemplateView` |

**Regla de oro:** si la vista hace exactamente lo que la CBV genérica hace, usá la CBV. Si necesitás más de dos o tres sobreescrituras de métodos para que funcione como querés, quizás una función es más clara.

---

## 4. Buenas prácticas y comparación con la doc oficial

| Tema | Lo que muestra el tutorial | Práctica actual recomendada |
|---|---|---|
| Formularios HTML | Manual con `<form>` | Usar `django.forms.Form` o `ModelForm` para validación robusta y menos código |
| Race condition en votos | `F("votes") + 1` ✅ | Correcto, es la forma estándar |
| Redirect tras POST | `HttpResponseRedirect` + `reverse()` ✅ | Correcto, sigue siendo la práctica estándar (patrón PRG) |
| CBV vs FBV | Muestra ambas | En proyectos reales se usan ambas según el caso; las CBV genéricas son ideales para CRUD estándar |
| `template_name` en CBV | Lo define siempre | Es buena práctica definirlo explícitamente para mayor claridad |
| `get_queryset()` | Solo las 5 últimas | En producción también filtrarías por `pub_date__lte=timezone.now()` para no mostrar preguntas futuras (cubierto en Parte 5) |

---

## Próximo paso

El **Tutorial 5** agrega tests automatizados y filtros de fecha a `get_queryset()`, que es exactamente el tipo de personalización más común en `ListView`. Con lo visto acá, cuando llegues a ese punto ya vas a entender qué método estás sobreescribiendo y por qué.
