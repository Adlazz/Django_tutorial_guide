# Django Tutorial - Parte 2: Admin

## Crear superusuario
```bash
python manage.py createsuperuser
```
Pedirá usuario, email y contraseña. Luego acceder en `http://127.0.0.1:8000/admin/`.

## Registrar un modelo en el Admin
Por defecto solo aparecen *Groups* y *Users*. Para agregar un modelo propio, editar `polls/admin.py`:
```python
from django.contrib import admin
from .models import Question

admin.site.register(Question)
```

## Qué ofrece el Admin automáticamente
- Formulario generado automáticamente desde el modelo.
- Widgets adecuados según el tipo de campo (`CharField` → input de texto, `DateTimeField` → calendario/hora).
- Opciones: *Save*, *Save and continue editing*, *Save and add another*, *Delete*.
- Historial de cambios por objeto (quién cambió qué y cuándo).

## Nota
Si la fecha no coincide, revisar `TIME_ZONE` en `settings.py`.
