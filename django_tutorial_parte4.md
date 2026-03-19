# Django — Notas de aprendizaje (Tutorial partes 1–3)

---

## Formateo de strings en Python

El tutorial de Django usa el operador `%` (estilo legacy). En código moderno se recomienda **f-strings**.

| Estilo | Ejemplo | ¿Usar? |
|--------|---------|--------|
| `%` operator (Python 2 / legacy) | `"Hola %s" % nombre` | ✗ No recomendado |
| `.format()` (Python 3 clásico) | `"Hola {}".format(nombre)` | ~ Aceptable |
| f-string (Python 3.6+ moderno) | `f"Hola {nombre}"` | ✓ Recomendado |

```python
# Con % (confuso con varios valores)
"Hola %s, tenés %d mensajes." % (nombre, cantidad)

# Con f-string (legible y moderno)
f"Hola {nombre}, tenés {cantidad} mensajes."
```

---

## Vistas (Views)

Una vista es una función Python que recibe un `request` y devuelve un `HttpResponse` (o lanza una excepción como `Http404`). Cada vista corresponde a una URL.

```python
def detail(request, question_id):
    return HttpResponse(f"You're looking at question {question_id}.")

def results(request, question_id):
    return HttpResponse(f"You're looking at the results of question {question_id}.")

def vote(request, question_id):
    return HttpResponse(f"You're voting on question {question_id}.")
```

### Evolución de la vista — simplificación progresiva

**1. Hardcodeada (sin template)**

```python
def index(request):
    latest_question_list = Question.objects.order_by("-pub_date")[:5]
    output = ", ".join([q.question_text for q in latest_question_list])
    return HttpResponse(output)
```

Problema: el HTML está mezclado con Python. Difícil de mantener.

**2. Manual con template (verboso)**

```python
from django.http import HttpResponse
from django.template import loader
from .models import Question

def index(request):
    latest_question_list = Question.objects.order_by("-pub_date")[:5]
    template = loader.get_template("polls/index.html")
    context = {"latest_question_list": latest_question_list}
    return HttpResponse(template.render(context, request))
```

**3. Con `render()` — shortcut recomendado**

```python
from django.shortcuts import render
from .models import Question

def index(request):
    latest_question_list = Question.objects.order_by("-pub_date")[:5]
    context = {"latest_question_list": latest_question_list}
    return render(request, "polls/index.html", context)
```

`render()` combina el request + template + context y devuelve el `HttpResponse` listo.

**4. Con `get_object_or_404()` — para objetos individuales**

```python
from django.shortcuts import get_object_or_404, render
from .models import Question

def detail(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    return render(request, "polls/detail.html", {"question": question})
```

Alternativa manual (más verbose, evitada con el atajo):

```python
try:
    question = Question.objects.get(pk=question_id)
except Question.DoesNotExist:
    raise Http404("Question does not exist")
```

- Si el objeto existe → lo devuelve
- Si no existe → lanza `Http404` automáticamente

**¿Por qué `get_object_or_404()` y no manejar la excepción a mano?**

"Loose coupling": el modelo (`Question`) no debería saber que existe HTTP ni códigos 404. Esa responsabilidad le corresponde a la vista. El shortcut hace ese trabajo en la capa correcta.

---

## El Context

Es el puente entre Python y el template. Un diccionario donde la **clave** es el nombre de variable que se usa en el HTML y el **valor** es el objeto Python real.

```python
# Forma con variable previa (más legible para varios datos)
context = {"latest_question_list": latest_question_list}
return render(request, "polls/index.html", context)

# Forma inline (más concisa para uno o dos datos)
return render(request, "polls/detail.html", {"question": question})
```

Son exactamente equivalentes. En el template se accede así:

```html
{{ question.question_text }}
{{ latest_question_list }}
```

---

## URLs — `urls.py`

### Dominio vs. ruta

- **Dominio**: `misitio.com` — no lo define Django
- **Ruta**: `/polls/5/results/` — esto sí lo define Django en `urls.py`

```
https://misitio.com/polls/5/results/
└─ dominio ──┘└─ ruta (Django) ──────┘
```

### La función `path()`

```python
path("<int:question_id>/results/", views.results, name="results")
#    └── 1. patrón ──────────────┘  └── 2. vista ┘  └── 3. nombre ┘
```

| Parámetro | Qué es |
|-----------|--------|
| Patrón | La forma que tiene la URL |
| Vista | Función a ejecutar cuando la URL coincide |
| Nombre | Alias para referenciar la URL desde templates o código |

### Angle brackets `< >` — variables en la URL

Se usan para capturar partes variables de la ruta:

```
<int : question_id>
 └─┘   └─────────┘
  │         │
  │         └── nombre del argumento que se pasa a la vista
  └── converter: solo acepta enteros
```

Cuando llega `/polls/34/results/`, Django extrae `34`, lo convierte a `int` y llama:

```python
results(request, question_id=34)
```

Si alguien pone `/polls/abc/results/` → Django devuelve **404 automáticamente**.

### Converters disponibles

| Converter | Acepta | Ejemplo |
|-----------|--------|---------|
| `int` | números enteros positivos | `34` |
| `str` | texto sin `/` | `hola-mundo` |
| `slug` | letras, números, guiones | `mi-articulo-2024` |
| `uuid` | formato UUID | `550e8400-e29b-41d4` |
| `path` | texto incluyendo `/` | `carpeta/archivo` |

### `polls/urls.py` — estado final de la parte 3

```python
from django.urls import path
from . import views

app_name = "polls"  # namespace

urlpatterns = [
    path("", views.index, name="index"),
    path("<int:question_id>/", views.detail, name="detail"),
    path("<int:question_id>/results/", views.results, name="results"),
    path("<int:question_id>/vote/", views.vote, name="vote"),
]
```

---

## Templates

### Estructura de carpetas — namespacing

```
polls/
└── templates/
    └── polls/           ← subcarpeta con el nombre de la app
        └── index.html   ← se referencia como "polls/index.html"
        └── detail.html
```

**¿Por qué la subcarpeta `polls/` dentro de `templates/`?**

Django busca templates en la carpeta `templates/` de *todas* las apps instaladas al mismo tiempo. Sin la subcarpeta, puede haber colisión:

```
# SIN namespace — colisión posible
polls/templates/index.html   ← ¿cuál usa Django?
blog/templates/index.html    ← ¿cuál usa Django?

# CON namespace — siempre único
polls/templates/polls/index.html   → "polls/index.html"
blog/templates/blog/index.html     → "blog/index.html"
```

Es el mismo problema que con las URLs — misma solución.

### Sintaxis de templates

| Sintaxis | Para qué | Ejemplo |
|----------|----------|---------|
| `{{ }}` | Mostrar un valor | `{{ question.question_text }}` |
| `{% %}` | Lógica / instrucciones | `{% if lista %}`, `{% for x in lista %}` |

- `{{ }}` = *"mostrá este dato"*
- `{% %}` = *"hacé esta operación"*

### Template de ejemplo — `index.html`

```html
{% if latest_question_list %}
    <ul>
    {% for question in latest_question_list %}
        <li>
            <a href="{% url 'polls:detail' question.id %}">
                {{ question.question_text }}
            </a>
        </li>
    {% endfor %}
    </ul>
{% else %}
    <p>No polls are available.</p>
{% endif %}
```

Los bloques `{% for %}`, `{% if %}` siempre necesitan su cierre (`{% endfor %}`, `{% endif %}`).

### URLs en templates — evitar hardcodeo

```html
<!-- Hardcodeado — frágil, hay que cambiar en cada template si cambia la ruta -->
<a href="/polls/{{ question.id }}/">

<!-- Con {% url %} — usa el name= de urls.py, se actualiza solo -->
<a href="{% url 'polls:detail' question.id %}">
```

---

## Namespacing de URLs

### El problema

Si dos apps tienen una vista con el mismo `name`:

```python
# polls/urls.py
path("<int:question_id>/", views.detail, name="detail")

# blog/urls.py
path("<int:post_id>/", views.detail, name="detail")
```

`{% url 'detail' %}` en un template es ambiguo — Django no sabe a cuál referirse.

### La solución — `app_name`

```python
# polls/urls.py
app_name = "polls"  # una sola línea

urlpatterns = [...]
```

```html
<!-- En el template, sintaxis namespace:nombre -->
{% url 'polls:detail' question.id %}
{% url 'blog:detail' post.id %}
```

### Resumen del patrón `namespace:nombre`

```
polls  :  detail
  │           │
  │           └── name= definido en path()
  └── app_name definido en urls.py
```

### Analogía templates vs. URLs

| Problema | Sin namespace | Con namespace |
|----------|--------------|---------------|
| Templates | `templates/index.html` → colisión | `templates/polls/index.html` → único |
| URLs | `{% url 'detail' %}` → ambiguo | `{% url 'polls:detail' %}` → único |

---

## Flujo completo de un request

```
Usuario entra a /polls/34/
        ↓
mysite/urls.py → detecta "polls/" → deriva a polls/urls.py
        ↓
polls/urls.py → matchea "<int:question_id>/" → llama a detail(request, question_id=34)
        ↓
views.py → busca Question con pk=34 → arma context → llama a render()
        ↓
render() → combina context con detail.html → devuelve HttpResponse con HTML final
        ↓
El navegador muestra la página
```

---

## Resumen de buenas prácticas

- Usar **f-strings** en lugar de `%` para formatear strings
- Usar **`render()`** en lugar de `loader.get_template()` + `HttpResponse` manual
- Usar **`get_object_or_404()`** para buscar objetos individuales
- Siempre aplicar **namespace** con `app_name` desde el principio
- Siempre usar **`{% url %}`** en templates, nunca hardcodear rutas
- Siempre crear la subcarpeta con el nombre de la app dentro de `templates/`
