# 🚀 Guía Rápida para Empezar con Django – App de Recetas

Este tutorial crea una aplicación web con Django donde se listan recetas y sus ingredientes asociados.

---

## 1. Crear y activar entorno virtual

```bash
python -m venv venv
source venv/Scripts/activate  # Windows
# source venv/bin/activate    # Linux / MacOS
pip install --upgrade pip
```

---

## 2. Instalar Django

```bash
pip install django
```

---

## 3. Crear el proyecto

```bash
mkdir djangotutorial
cd djangotutorial
django-admin startproject mysite .
```

> El `.` al final indica que el proyecto se crea dentro del directorio actual.

---

## 4. Ejecutar el servidor por primera vez

```bash
python manage.py runserver
```

Verifica que funcione ingresando a [http://127.0.0.1:8000](http://127.0.0.1:8000)

---

## 5. Crear una app llamada `recipes`

```bash
python manage.py startapp recipes
```

---

## 6. Crear la primera vista

Archivo: `recipes/views.py`

```python
from django.http import HttpResponse

def index(request):
    return HttpResponse("Bienvenido a la app de recetas")
```

---

## 7. Mapear la vista a una URL

Crear archivo: `recipes/urls.py`

```python
from django.urls import path
from . import views

urlpatterns = [
    path("", views.index, name="index"),
]
```

---

## 8. Incluir la URL de la app en el enrutador principal

Editar `mysite/urls.py`:

```python
from django.contrib import admin
from django.urls import include, path

urlpatterns = [
    path("recipes/", include("recipes.urls")),  # ← Nueva línea
    path("admin/", admin.site.urls),
]
```

---

## 9. Verificar que funcione

```bash
python manage.py runserver
```

Ir a [http://127.0.0.1:8000/recipes](http://127.0.0.1:8000/recipes) → debería mostrar: `Bienvenido a la app de recetas`

---

## 10. Configurar zona horaria y lenguaje

Editar `mysite/settings.py`:

```python
LANGUAGE_CODE = 'es-ar'
TIME_ZONE = 'America/Argentina/Buenos_Aires'

USE_I18N = True
USE_L10N = True
USE_TZ = True
```

---

## 11. Preparar la base de datos

```bash
python manage.py migrate
```

---

## 12. Crear modelos

Archivo: `recipes/models.py`

```python
from django.db import models

class Recipe(models.Model):
    name = models.CharField(max_length=200)
    created_at = models.DateTimeField("fecha de creación")

    def __str__(self):
        return self.name

class Ingredient(models.Model):
    recipe = models.ForeignKey(Recipe, on_delete=models.CASCADE)
    name = models.CharField(max_length=100)
    quantity = models.CharField(max_length=100)

    def __str__(self):
        return f"{self.name} ({self.quantity})"
```

---

## 13. Registrar la app

Editar `mysite/settings.py`, agregar `"recipes.apps.RecipesConfig"` en `INSTALLED_APPS`:

```python
INSTALLED_APPS = [
    "recipes.apps.RecipesConfig",  # ← nueva línea
    "django.contrib.admin",
    "django.contrib.auth",
    "django.contrib.contenttypes",
    "django.contrib.sessions",
    "django.contrib.messages",
    "django.contrib.staticfiles",
]
```

---

## 14. Crear y aplicar migraciones

```bash
python manage.py makemigrations recipes
python manage.py migrate
```

---

## 🧠 Conceptos clave

### ¿Qué es una app en Django?
Una app es un módulo independiente que hace algo específico. En este caso, la app `recipes` gestiona recetas e ingredientes.

### ¿Qué son migraciones?
Son los archivos que describen los cambios en los modelos para reflejarlos en la base de datos.

---

## ✅ Siguientes pasos

- Registrar los modelos en el admin para gestionarlos visualmente.
- Crear un formulario para agregar recetas.
- Conectar una base de datos real como PostgreSQL o MySQL.
- Usar Django REST Framework para una API.
