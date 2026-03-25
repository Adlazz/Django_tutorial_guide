# 📘 Guía Desarrollada – Django desde Cero con App de Recetas

Esta guía expande cada paso y concepto de la guía resumida, profundizando en el **por qué** y **para qué** de cada componente. Ideal para entender Django de verdad, no solo copiar y pegar comandos.

---

## 1. Entorno virtual

```bash
python -m venv venv
venv/Scripts/activate  # Windows
# source venv/bin/activate  # Linux / MacOS
python.exe -m pip install --upgrade pip
```

### ¿Qué es y por qué usarlo?

Un entorno virtual es una **copia aislada de Python** dentro de tu proyecto. Cada proyecto puede tener sus propias versiones de paquetes sin afectar a otros proyectos ni al Python global de tu sistema.

**Ejemplo real del problema que resuelve:** Tenés un proyecto viejo que usa Django 3.2 y uno nuevo que usa Django 5.0. Sin entorno virtual, instalar uno rompería el otro. Con entornos virtuales, cada proyecto tiene su propia versión de Django sin conflictos.

Cuando activás el entorno (`venv/Scripts/activate`), tu terminal "apunta" a ese Python aislado. Todo lo que instales con `pip` queda **solamente** dentro de esa carpeta `venv/`.

---

## 2. Instalar Django

```bash
pip install django
```

Esto instala Django **solo dentro del entorno virtual** activo. Si cerrás la terminal y no activás el entorno, el comando `django-admin` no va a funcionar porque el sistema no lo encuentra.

Django se instala como un paquete de Python que trae herramientas de línea de comandos (`django-admin`, `manage.py`) y todos los módulos que vas a usar en tu código.

---

## 3. Crear el proyecto

```bash
mkdir djangotutorial
cd djangotutorial
django-admin startproject mysite .
```

### ¿Qué es `django-admin`?

Es la **herramienta de línea de comandos** que viene con Django. Sirve para crear proyectos, crear apps, y ejecutar tareas administrativas.

### ¿Qué hace el `.` al final?

El punto le dice a Django: "creá los archivos del proyecto **acá mismo**, en el directorio actual". Sin el punto, Django crearía una carpeta extra anidada (`djangotutorial/mysite/mysite/`), lo cual es redundante e incómodo.

### Estructura generada

```
djangotutorial/
├── manage.py
└── mysite/
    ├── __init__.py
    ├── settings.py
    ├── urls.py
    ├── asgi.py
    └── wsgi.py
```

Veamos cada archivo:

### `manage.py`

Es el **asistente de línea de comandos** de tu proyecto. Hace lo mismo que `django-admin`, pero ya configurado para tu proyecto específico. A partir de ahora, vas a usar `python manage.py` en lugar de `django-admin` para casi todo.

### `mysite/__init__.py`

Un archivo vacío que le dice a Python: "esta carpeta es un **paquete** de Python, podés importar cosas desde acá". Sin este archivo, Python no reconocería `mysite` como un módulo.

### `mysite/settings.py`

El **archivo de configuración central** de tu proyecto. Acá se define todo: qué apps están instaladas, la base de datos, el idioma, la zona horaria, la seguridad, etc. Es el "tablero de control" de Django.

### `mysite/urls.py`

El **enrutador principal** del proyecto. Es la primera parada cuando llega una request HTTP. Define qué URLs existen y a dónde dirigir cada una.

### `mysite/wsgi.py` y `mysite/asgi.py`

Son los **puntos de entrada** para conectar tu aplicación con servidores web reales en producción.

**WSGI (Web Server Gateway Interface)** es el protocolo tradicional y sincrónico. Funciona como un molinete que deja pasar las requests de a una por vez. Es lo que usás para apps web normales (blogs, e-commerce, CRUDs) con servidores como **Gunicorn** o **uWSGI**.

**ASGI (Asynchronous Server Gateway Interface)** es el protocolo moderno y asincrónico. Puede manejar múltiples requests al mismo tiempo y además soporta **WebSockets** (conexiones en tiempo real). Es para apps que necesitan chat en vivo, notificaciones en tiempo real o streaming. Usa servidores como **Daphne** o **Uvicorn**.

**¿Cuándo elegir cuál?** Si tu app es un CRUD normal, WSGI alcanza y sobra. Si necesitás tiempo real, ASGI es el camino. Para desarrollo, ninguno de los dos importa porque usás el servidor de desarrollo de Django.

---

## 4. El servidor de desarrollo

```bash
python manage.py runserver
```

Django incluye un **servidor de desarrollo liviano** escrito en Python puro. Su único propósito es que puedas desarrollar rápido sin configurar nada complejo.

### ¿Por qué NO usarlo en producción?

- **Performance**: Es lento, maneja pocas requests simultáneas
- **Seguridad**: No está preparado para enfrentar ataques reales ni tráfico masivo
- **Estabilidad**: Puede caerse fácilmente ante errores
- **Limitaciones**: No sirve archivos estáticos eficientemente, no tiene SSL, etc.

### Analogía

El servidor de desarrollo es como cocinar en tu casa para vos y tus amigos. Un servidor de producción (Gunicorn + Nginx) es como tener un restaurante con cocina industrial que aguanta cientos de clientes.

### En desarrollo vs producción

```bash
# Lo que usás ahora (desarrollo):
python manage.py runserver

# Lo que usarías en producción:
gunicorn mysite.wsgi:application
```

---

## 5. Crear una app: `recipes`

```bash
python manage.py startapp recipes
```

### ¿Qué es una "app" en Django?

Una app es un **módulo independiente** que hace algo específico. No es una "aplicación" completa como la pensarías normalmente (como una app de celular). Es más bien un **componente reutilizable** dentro de tu proyecto.

Tu **proyecto** (`mysite`) es el contenedor principal. Dentro puede tener múltiples **apps** que hacen cosas distintas. Por ejemplo:

- `recipes` → gestiona recetas e ingredientes
- `users` → gestiona perfiles de usuario
- `blog` → gestiona artículos

Cada app puede ser desenchufada y usada en otro proyecto diferente. Esa es la filosofía **modular** de Django.

### Estructura generada por `startapp`

```
recipes/
├── __init__.py
├── admin.py       ← Registro de modelos en el panel de admin
├── apps.py        ← Configuración de la app
├── migrations/    ← Historial de cambios en la base de datos
│   └── __init__.py
├── models.py      ← Definición de modelos (tablas de la DB)
├── tests.py       ← Tests automatizados
└── views.py       ← Lógica de las vistas (qué devolver al usuario)
```

---

## 6. La primera vista

```python
# recipes/views.py
from django.http import HttpResponse

def index(request):
    return HttpResponse("Bienvenido a la app de recetas")
```

### ¿Qué es una vista?

Una vista es una **función Python** (o clase) que recibe una request HTTP y devuelve una response HTTP. Es donde vivie la lógica de "qué le muestro al usuario cuando visita tal URL".

### El parámetro `request`

Cada vista recibe obligatoriamente un objeto `request`. Este objeto contiene **toda la información** sobre el pedido que hizo el usuario: qué URL visitó, qué método HTTP usó (GET, POST), qué datos envió, las cookies, las cabeceras, etc.

### `HttpResponse`

Es la forma más básica de devolver contenido al navegador. Literalmente le mandás un string de texto que el navegador muestra. Más adelante vas a usar `render()` que permite devolver templates HTML completos con datos dinámicos:

```python
# Ahora (respuesta simple de texto):
return HttpResponse("Bienvenido a la app de recetas")

# Más adelante (respuesta con template HTML):
return render(request, 'recipes/index.html', {'recetas': recetas})
```

### ¿Y el `render` que viene por defecto?

Cuando creás la app, Django pone `from django.shortcuts import render` por defecto en `views.py`. No lo necesitás todavía, pero no molesta dejarlo. Lo vas a usar cuando trabajes con templates HTML.

---

## 7. Mapear la vista a una URL

```python
# recipes/urls.py (hay que crearlo manualmente)
from django.urls import path
from . import views

urlpatterns = [
    path("", views.index, name="index"),
]
```

### `django.urls`

Es un **módulo de Django** que contiene todo lo relacionado con el manejo de URLs. De ahí importamos `path`, que es la función que usamos para definir rutas.

### `from . import views`

El **punto** (`.`) significa "desde el directorio actual". Es decir: "en esta misma carpeta (`recipes/`), importá el archivo `views.py`". Esto se llama **importación relativa**.

### `urlpatterns`

Es una **lista** que Django busca automáticamente dentro de cada `urls.py`. Tiene que llamarse exactamente `urlpatterns` (no podés inventar otro nombre). Django la recorre para ver qué URLs existen en tu app.

### La función `path()` en detalle

`path()` es una función prefabricada de Django que recibe parámetros y los procesa internamente para hacer funcionar el sistema de URLs:

```python
path("", views.index, name="index")
#    ↓        ↓            ↓
# route    view         name
```

**Primer parámetro: `route` (la ruta)**
- `""` = ruta vacía = la raíz de la app
- Si fuera `"nueva/"` → la URL sería `/recipes/nueva/`
- Django compara esta ruta con la URL que pide el usuario

**Segundo parámetro: `view` (la vista)**
- `views.index` le dice a Django: "cuando alguien visite esta URL, ejecutá la función `index()` de `views.py`"
- Es una referencia a la función, no una llamada (sin paréntesis)

**Tercer parámetro: `name` (el nombre interno)**
- `name="index"` le asigna un nombre para referenciar esta URL desde otros lados del código
- En lugar de escribir la ruta literal en templates:

```html
<!-- Sin name (frágil, si cambiás la ruta se rompe): -->
<a href="/recipes/">Ver recetas</a>

<!-- Con name (robusto, Django resuelve la ruta por vos): -->
<a href="{% url 'index' %}">Ver recetas</a>
```

La ventaja es que si después cambiás la ruta de `""` a `"inicio/"`, no tenés que tocar nada más en tu código. Django busca por el nombre.

---

## 8. Incluir la URL de la app en el enrutador principal

```python
# mysite/urls.py
from django.contrib import admin
from django.urls import include, path

urlpatterns = [
    path("recipes/", include("recipes.urls")),
    path("admin/", admin.site.urls),
]
```

### ¿Cómo funciona `include()`?

`include()` es una función que permite **delegar** el manejo de URLs a otro archivo. La idea es "plug-and-play": cada app maneja sus propias URLs internamente.

Cuando Django recibe una request, hace lo siguiente:

```
Usuario visita: http://127.0.0.1:8000/recipes/

1. Django lee mysite/urls.py (el enrutador principal)
2. Encuentra path("recipes/", include("recipes.urls"))
3. "Corta" la parte "recipes/" que ya coincidió
4. Envía lo que sobra ("" en este caso) a recipes/urls.py
5. En recipes/urls.py busca path("", views.index)
6. ¡Coincide! Ejecuta views.index()
```

Esto tiene una ventaja enorme: podés cambiar `"recipes/"` por `"cocina/"` **en un solo lugar** (el archivo principal) y toda la app sigue funcionando perfectamente.

### `include("recipes.urls")` – El dotted path

`"recipes.urls"` es un **dotted path** (ruta con puntos). Es la forma que Python usa para referenciar módulos usando puntos en lugar de barras:

```
"recipes.urls"  →  recipes/urls.py
```

Se lee como: "desde el paquete `recipes`, importá el módulo `urls`". Django lo importa dinámicamente cuando arranca.

### `admin.site.urls`

`django.contrib.admin` no es solo una función, es una **app completa** (todo el panel de administración). `admin.site.urls` conecta todas las URLs internas del admin para que funcionen bajo `/admin/`.

---

## 9. Verificar que funcione

```bash
python manage.py runserver
```

Visitando `http://127.0.0.1:8000/recipes/` deberías ver el mensaje "Bienvenido a la app de recetas". Si funciona, significa que toda la cadena está conectada correctamente:

```
URL del navegador → mysite/urls.py → recipes/urls.py → recipes/views.py → HttpResponse
```

---

## 10. Configurar zona horaria y lenguaje

```python
# mysite/settings.py
LANGUAGE_CODE = 'es-ar'
TIME_ZONE = 'America/Argentina/Buenos_Aires'

USE_I18N = True
USE_L10N = True
USE_TZ = True
```

### ¿Qué hace cada configuración?

**`LANGUAGE_CODE = 'es-ar'`**
Define el idioma de la interfaz de Django (panel de admin, mensajes de error, formularios). `'es-ar'` = español de Argentina.

**`TIME_ZONE = 'America/Argentina/Buenos_Aires'`**
Define la zona horaria. Afecta cómo Django almacena y muestra fechas y horas. Por defecto viene `'UTC'`.

**`USE_I18N = True`** (Internationalization)
Activa el sistema de **traducciones**. La "I18N" viene de que entre la "I" y la "N" de "Internationalization" hay 18 letras. Con esto activado, Django traduce automáticamente textos de la interfaz al idioma configurado (ej: "Welcome" → "Bienvenido").

**`USE_L10N = True`** (Localization)
Activa el **formateo regional** de datos. La "L10N" viene de "Localization" (10 letras entre L y N). Formatea fechas, números y monedas según la región:

```
Con USE_L10N = True (Argentina):
  fecha  → "5 de febrero de 2026"
  número → "1.234,56"

Con USE_L10N = False (formato estándar):
  fecha  → "Feb. 5, 2026"
  número → "1,234.56"
```

**Nota sobre versiones:** En Django 4.0+, `USE_L10N` está **siempre activado** por defecto (deprecated). Podés dejar la línea sin problema, pero no es estrictamente necesaria en versiones modernas.

**`USE_TZ = True`**
Hace que Django maneje las fechas con **zona horaria** (timezone-aware). Esto evita problemas cuando tu app tiene usuarios en distintas zonas horarias.

### `USE_I18N` vs `USE_L10N`: ¿Son lo mismo?

No, son independientes. `I18N` traduce textos, `L10N` formatea datos numéricos y temporales. Podés tener uno sin el otro, aunque en la práctica conviene tener ambos activados.

---

## 11. Preparar la base de datos

```bash
python manage.py migrate
```

### ¿Hay que crear la base de datos manualmente?

**¡No!** Django ya creó automáticamente un archivo `db.sqlite3` cuando creaste el proyecto con `startproject`. Ese archivo **es tu base de datos** SQLite. Existe, pero está vacía (sin tablas).

### ¿Qué hace `migrate`?

Lee las apps que están en `INSTALLED_APPS` y crea las **tablas necesarias** para que funcionen:

```
db.sqlite3 (existe pero está vacía)
    ↓
python manage.py migrate
    ↓
db.sqlite3 (ahora tiene tablas: auth_user, django_session, etc.)
```

### ¿Es obligatorio?

Sí, si vas a usar las apps que vienen por defecto. Por ejemplo, si querés usar el panel de admin necesitás las tablas de usuarios. Si no corrés `migrate` y tratás de usar el admin, Django te tira error porque las tablas no existen.

### Las apps predefinidas de Django

Estas son las apps que vienen instaladas por defecto en `INSTALLED_APPS` y que `migrate` configura:

- **`django.contrib.admin`** → Panel de administración visual
- **`django.contrib.auth`** → Sistema de usuarios, login, logout y permisos
- **`django.contrib.contenttypes`** → Sistema interno que registra todos los modelos del proyecto. Otras apps (como auth y admin) lo necesitan para funcionar. Raramente interactuás directamente con esto
- **`django.contrib.sessions`** → Maneja sesiones de usuario: quién está logueado, datos del carrito de compras, etc. Guarda datos temporales del usuario entre requests
- **`django.contrib.messages`** → Sistema de notificaciones temporales para el usuario ("Guardado exitosamente", "Error al enviar"). Aparecen una vez y desaparecen
- **`django.contrib.staticfiles`** → Maneja archivos estáticos (CSS, JavaScript, imágenes). Los organiza durante desarrollo y los prepara para el deploy

---

## 12. Crear modelos

```python
# recipes/models.py
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

### ¿Qué es un modelo?

Un modelo es una **clase de Python** que define la estructura de una tabla en la base de datos. Cada atributo de la clase se convierte en una **columna** de la tabla. Django se encarga de traducir tu código Python a SQL automáticamente (esto es el **ORM**, que veremos más adelante).

```
Clase Python (Recipe)     →    Tabla en la DB (recipes_recipe)
Atributo (name)           →    Columna (name)
Instancia (Recipe(...))   →    Fila/registro en la tabla
```

### `models.Model`

Todas tus clases de modelo heredan de `models.Model`. Esta clase base le da a tu modelo toda la funcionalidad de Django: conexión con la base de datos, métodos para guardar, borrar, consultar, etc.

### Tipos de campos (Field types) más comunes

Django tiene muchos tipos de campos. Cada uno define qué tipo de dato almacena la columna:

**Campos de texto:**

```python
nombre = models.CharField(max_length=100)      # Texto corto (obligatorio max_length)
descripcion = models.TextField()                # Texto largo (sin límite)
email = models.EmailField()                     # Email (valida formato)
sitio_web = models.URLField()                   # URL (valida formato)
slug = models.SlugField(max_length=200)         # Para URLs amigables: "mi-receta-genial"
```

**Campos numéricos:**

```python
edad = models.IntegerField()                                      # Número entero
precio = models.DecimalField(max_digits=10, decimal_places=2)     # Decimal: 99999999.99
cantidad = models.PositiveIntegerField()                           # Entero positivo
temperatura = models.FloatField()                                  # Float
```

**Campos de fecha/hora:**

```python
nacimiento = models.DateField()                          # Solo fecha
hora_apertura = models.TimeField()                       # Solo hora
creado = models.DateTimeField()                          # Fecha y hora completa
creado = models.DateTimeField(auto_now_add=True)         # Se setea solo al crear
modificado = models.DateTimeField(auto_now=True)         # Se actualiza cada vez que guardás
```

**Campos booleanos y de archivos:**

```python
activo = models.BooleanField(default=True)               # True/False
foto = models.ImageField(upload_to='fotos/')             # Imagen
documento = models.FileField(upload_to='documentos/')    # Archivo cualquiera
```

### Argumentos opcionales comunes de los campos

Los campos aceptan argumentos opcionales que controlan su comportamiento:

```python
# Puede ser NULL en la DB
campo = models.CharField(max_length=100, null=True)

# Puede estar vacío en formularios
campo = models.CharField(max_length=100, blank=True)

# Valor por defecto
pais = models.CharField(max_length=50, default='Argentina')

# Opciones limitadas (choices)
DIFICULTAD_CHOICES = [
    ('F', 'Fácil'),
    ('M', 'Media'),
    ('D', 'Difícil'),
]
dificultad = models.CharField(max_length=1, choices=DIFICULTAD_CHOICES)

# Único (no puede repetirse)
dni = models.IntegerField(unique=True)

# Texto de ayuda (se muestra en formularios y admin)
email = models.EmailField(help_text="Ingresá tu email personal")
```

### Nombre legible para humanos (primer argumento opcional)

Cuando definís un campo, podés ponerle un **nombre descriptivo** como primer argumento. Este nombre se usa en el panel de admin y formularios:

```python
# SIN nombre legible (Django formatea el nombre de la variable automáticamente):
name = models.CharField(max_length=200)
# → En el admin muestra: "Name"

# CON nombre legible (primer argumento):
created_at = models.DateTimeField("fecha de creación")
# → En el admin muestra: "fecha de creación" en lugar de "Created at"
```

Es solo cosmético, para que la interfaz se vea más prolija. En tu código seguís usando el nombre de la variable (`created_at`).

### Relaciones entre modelos

Django soporta los tres tipos de relaciones de base de datos:

**ForeignKey (Muchos a Uno):**

```python
recipe = models.ForeignKey(Recipe, on_delete=models.CASCADE)
```

Esto dice: "cada Ingrediente pertenece a **una** Receta, pero una Receta puede tener **muchos** Ingredientes". Es la relación más común.

`on_delete=models.CASCADE` significa: "si borro una Receta, también borrar todos sus Ingredientes". Otras opciones son `PROTECT` (impide borrar), `SET_NULL` (pone null), etc.

**ManyToManyField (Muchos a Muchos):**

```python
tags = models.ManyToManyField('Tag')
```

Ejemplo: una receta puede tener muchos tags, y un tag puede estar en muchas recetas.

**OneToOneField (Uno a Uno):**

```python
perfil = models.OneToOneField(User, on_delete=models.CASCADE)
```

Ejemplo: cada usuario tiene exactamente un perfil, y cada perfil pertenece a exactamente un usuario.

### El método `__str__`

```python
def __str__(self):
    return self.name
```

Define cómo se muestra el objeto cuando lo imprimís o lo ves en el admin. Sin este método, Django muestra algo como `Recipe object (1)`, que no sirve para nada. Con `__str__`, muestra el nombre de la receta.

---

## 13. Registrar la app

```python
# mysite/settings.py
INSTALLED_APPS = [
    "recipes.apps.RecipesConfig",  # ← tu app
    "django.contrib.admin",
    "django.contrib.auth",
    "django.contrib.contenttypes",
    "django.contrib.sessions",
    "django.contrib.messages",
    "django.contrib.staticfiles",
]
```

### ¿Por qué hay que registrarla?

Django no sabe automáticamente que tu app existe. Tenés que agregarla a `INSTALLED_APPS` para que Django la tenga en cuenta al momento de buscar modelos, crear migraciones, servir archivos estáticos, etc.

### El dotted path `"recipes.apps.RecipesConfig"`

Es la forma de Python de referenciar algo dentro de un módulo usando puntos en lugar de barras:

```
"recipes.apps.RecipesConfig"
    ↓       ↓        ↓
 carpeta  archivo   clase
```

Se traduce a: "andá a `recipes/apps.py` y agarrá la clase `RecipesConfig`".

Es equivalente a hacer `from recipes.apps import RecipesConfig` en código Python, pero Django lo usa como string porque lo importa dinámicamente al arrancar el proyecto.

### ¿Se puede usar la forma corta?

Sí, en versiones modernas de Django podés poner solo `'recipes'` en lugar del dotted path completo. Django automáticamente busca `recipes/apps.py` y usa la clase de configuración que encuentre. Sin embargo, la documentación oficial recomienda el dotted path completo por claridad.

---

## 14. Crear y aplicar migraciones

```bash
python manage.py makemigrations recipes
python manage.py migrate
```

### ¿Qué son las migraciones?

Las migraciones son **archivos que describen los cambios** que hiciste en tus modelos para que Django pueda reflejarlos en la base de datos. Son como un "historial de cambios" de la estructura de tu DB.

### El flujo completo de migraciones

```
1. Modificás models.py (creás o cambiás modelos)
    ↓
2. makemigrations (crea el archivo de migración = "el plan")
    ↓
3. sqlmigrate (opcional, solo para ver qué SQL generaría)
    ↓
4. migrate (aplica los cambios a la base de datos)
```

### `makemigrations` – Crear el plan

```bash
python manage.py makemigrations recipes
```

Django lee tus modelos en `models.py`, los compara con el estado anterior, y genera un archivo de migración (como `0001_initial.py`) dentro de `recipes/migrations/`. Este archivo **no toca la base de datos**, solo describe qué cambios hacer.

### `migrate` – Ejecutar el plan

```bash
python manage.py migrate
```

Lee los archivos de migración pendientes y **ejecuta el SQL correspondiente** en la base de datos. Acá sí se crean/modifican las tablas reales.

### `sqlmigrate` – Ver qué va a hacer (opcional)

```bash
python manage.py sqlmigrate recipes 0001
```

**No ejecuta nada.** Solo muestra en pantalla el SQL que Django generaría al aplicar esa migración. Es útil para debugging, para revisar antes de aplicar cambios, o si tenés un DBA que necesita revisar los scripts SQL.

### Analogía

- **makemigrations** → Escribís la receta de cocina
- **sqlmigrate** → Leés la receta para revisarla antes de cocinar
- **migrate** → Cocinás el plato

### Nombres automáticos de tablas

Django genera nombres de tabla automáticamente combinando el nombre de la app con el nombre del modelo en minúsculas:

```
App: recipes + Modelo: Recipe  →  Tabla: recipes_recipe
App: recipes + Modelo: Ingredient  →  Tabla: recipes_ingredient
```

También agrega automáticamente un campo `id` como primary key (clave primaria) y el sufijo `_id` a los campos ForeignKey.

---

## 🧠 Conceptos clave ampliados

### El ORM de Django (Object-Relational Mapping)

El ORM es el sistema que te permite interactuar con la base de datos usando **código Python** en lugar de escribir SQL directamente. Es el corazón de Django y lo vas a usar constantemente.

**CRUD básico (Create, Read, Update, Delete):**

```python
# CREATE - Crear una receta
r = Recipe(name="Empanadas", created_at=timezone.now())
r.save()

# READ - Leer recetas
Recipe.objects.all()                        # Todas las recetas
Recipe.objects.get(id=1)                    # Una receta específica por ID
Recipe.objects.filter(name__startswith="E") # Varias con filtro

# UPDATE - Actualizar
r.name = "Empanadas salteñas"
r.save()

# DELETE - Borrar
r.delete()
```

**Acceder a relaciones:**

```python
# Desde la Receta → obtener sus Ingredientes
r.ingredient_set.all()
r.ingredient_set.create(name="Carne", quantity="500g")

# Desde el Ingrediente → obtener su Receta
i = Ingredient.objects.get(id=1)
i.recipe        # La receta a la que pertenece
i.recipe.name   # El nombre de esa receta
```

**Lookups (filtros avanzados):**

El doble guión bajo (`__`) es la forma de Django de acceder a filtros especiales y atravesar relaciones:

```python
.filter(id=1)                              # Igual a
.filter(name__startswith="Emp")            # Empieza con
.filter(created_at__year=2026)             # Año específico
.filter(quantity__contains="500")           # Contiene
Ingredient.objects.filter(recipe__name="Empanadas")
# "Dame ingredientes cuya receta se llame Empanadas"
```

**Resumen de los métodos más usados:**

- `objects.all()` → todos los registros
- `objects.get()` → un solo registro (tira error si no existe o hay más de uno)
- `objects.filter()` → varios registros con condición
- `objeto.save()` → guardar cambios
- `objeto.delete()` → borrar
- `modelo_relacionado_set` → acceder a relaciones inversas

No necesitás memorizarlo todo ahora, pero entender estos conceptos te va a servir para todo lo que viene.

---

## ✅ Siguientes pasos

- Registrar los modelos en el admin para gestionarlos visualmente
- Crear templates HTML para mostrar las recetas en el navegador
- Usar formularios para agregar recetas desde la web
- Conectar una base de datos real como PostgreSQL o MySQL
- Usar Django REST Framework para crear una API
