[django_tutorial_parte3.md](https://github.com/user-attachments/files/26143934/django_tutorial_parte3.md)
# Django Tutorial - Parte 3: Vistas y Templates

## Qué es una Vista
Una vista es una función Python que recibe un `request` y devuelve un `HttpResponse` (o lanza una excepción como `Http404`). Cada vista corresponde a una URL.

---

## Evolución de las Vistas (de simple a completo)

### 1. Vista hardcodeada (sin template)
```python
def index(request):
    latest_question_list = Question.objects.order_by("-pub_date")[:5]
    output = ", ".join([q.question_text for q in latest_question_list])
    return HttpResponse(output)
```
Problema: el HTML está mezclado con Python. Difícil de mantener.

### 2. Vista con Template (forma correcta)
```python
def index(request):
    latest_question_list = Question.objects.order_by("-pub_date")[:5]
    context = {"latest_question_list": latest_question_list}
    return render(request, "polls/index.html", context)
```
`render()` es un atajo de Django que combina el request + template + contexto y devuelve el HttpResponse listo.

---

## Templates

### Estructura de carpetas requerida
```
polls/
    templates/
        polls/
            index.html
            detail.html
```
> La carpeta anidada `polls/` dentro de `templates/` es convención para evitar colisiones de nombres entre apps.

### Ejemplo de template `index.html`
```html
{% if latest_question_list %}
    <ul>
    {% for question in latest_question_list %}
        <li><a href="{% url 'polls:detail' question.id %}">{{ question.question_text }}</a></li>
    {% endfor %}
    </ul>
{% else %}
    <p>No polls are available.</p>
{% endif %}
```
- `{{ variable }}` → imprime el valor de una variable
- `{% tag %}` → lógica: loops, condicionales, urls, etc.

---

## El Context

El **context** es un diccionario que le pasa datos de Python al template.

```python
# Forma con variable previa (más legible para varios datos)
context = {"latest_question_list": latest_question_list}
return render(request, "polls/index.html", context)

# Forma inline (más concisa para uno o dos datos)
return render(request, "polls/detail.html", {"question": question})
```
Son exactamente equivalentes. La clave del diccionario (`"question"`, `"latest_question_list"`) es el nombre con el que accedés a la variable **dentro del template**.

```html
<!-- En el template accedés así: -->
{{ question.question_text }}
{{ latest_question_list }}
```

---

## get_object_or_404

```python
from django.shortcuts import get_object_or_404, render

def detail(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    return render(request, "polls/detail.html", {"question": question})
```
- Si existe → devuelve el objeto
- Si no existe → lanza `Http404` automáticamente (sin necesidad de try/except manual)

Alternativa manual (más verbose, evitada con el atajo):
```python
try:
    question = Question.objects.get(pk=question_id)
except Question.DoesNotExist:
    raise Http404("Question does not exist")
```

---

## Namespacing de URLs (`app_name`)

Para evitar conflictos entre apps, se agrega en `polls/urls.py`:
```python
app_name = "polls"  # <-- esto habilita el namespace
urlpatterns = [...]
```

Luego en los templates se referencian las URLs así:
```html
<!-- Sin namespace (puede romper si hay otra app con "detail") -->
{% url 'detail' question.id %}

<!-- Con namespace (recomendado) -->
{% url 'polls:detail' question.id %}
```
> Esto probablemente fue la causa del error que tuviste al correr `/polls`. El tutorial lo menciona al final pero es necesario antes.

---

## URLs de la app (`polls/urls.py`) - Estado final de la parte 3

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
