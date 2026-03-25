# 📚 Guía de Django - Tutorial Oficial Expandido

Guía completa basada en el tutorial oficial de Django, con explicaciones detalladas y conceptos ampliados para un aprendizaje profundo del framework.

## 📖 Contenido

### Tutorial 01 - Setup e Instalación Inicial

- **[tutorial01_setup_initial_app.md](tutorial01_setup_initial_app.md)** - Guía rápida de inicio
  - Configuración del entorno virtual
  - Instalación de Django
  - Creación del proyecto y primera app
  - Configuración básica de modelos y migraciones

- **[tutorial01_setup_initial_app_detailed.md](tutorial01_setup_initial_app_detailed.md)** - Versión detallada
  - Explicación profunda de cada componente
  - ¿Qué es y por qué usar un entorno virtual?
  - Estructura del proyecto Django
  - Conceptos clave: apps, modelos, ORM, migraciones
  - WSGI vs ASGI

### Tutorial 02 - Panel de Administración

- **[tutorial02_admin_panel.md](tutorial02_admin_panel.md)**
  - Creación de superusuario
  - Registro de modelos en el admin
  - Características automáticas del panel de administración

### Tutorial 03 - Vistas, Templates y URLs

- **[tutorial03_views_templates_urls.md](tutorial03_views_templates_urls.md)**
  - Evolución de vistas (hardcodeada → template → render)
  - Sistema de templates de Django
  - Context y paso de datos
  - `get_object_or_404`
  - Namespacing de URLs con `app_name`
  - Flujo completo de un request

### Tutorial 04 - Notas de Aprendizaje

- **[tutorial04_notes_views_urls_templates.md](tutorial04_notes_views_urls_templates.md)**
  - Formateo de strings en Python (f-strings vs %)
  - Comparación de diferentes enfoques en vistas
  - Shortcuts de Django: `render()`, `get_object_or_404()`
  - Anatomía del sistema de URLs
  - Converters disponibles (`int`, `str`, `slug`, etc.)
  - Buenas prácticas y resumen de conceptos

### Tutorial 05 - Formularios y Vistas Genéricas

- **[tutorial05_forms_generic_views.md](tutorial05_forms_generic_views.md)**
  - Creación de formularios HTML
  - Protección CSRF
  - Vista `vote()` con manejo de POST
  - Race conditions y expresiones F
  - Patrón Post/Redirect/Get (PRG)
  - Class-Based Views (CBV)
  - Vistas genéricas: `ListView`, `DetailView`
  - `CreateView`, `UpdateView`, `DeleteView`
  - Personalización de CBV

## 🎯 Características de esta Guía

- ✅ **Dos niveles de profundidad**: versión rápida y versión detallada
- ✅ **Explicaciones del "por qué"**: no solo el "qué" y "cómo"
- ✅ **Buenas prácticas**: comparación entre enfoques legacy y modernos
- ✅ **Conceptos ampliados**: ORM, migraciones, seguridad, race conditions
- ✅ **Ejemplos prácticos**: código real con anotaciones
- ✅ **Flujos completos**: seguimiento paso a paso de requests HTTP

## 🚀 Cómo Usar Esta Guía

### Para principiantes
1. Comenzá con **tutorial01_setup_initial_app.md** (versión rápida)
2. Seguí el orden numérico de los tutoriales
3. Leé la versión **detailed** de la parte 1 cuando quieras profundizar

### Para quienes ya conocen Django
1. Usá esta guía como referencia rápida
2. Consultá las versiones detalladas para refrescar conceptos específicos
3. Revisá **tutorial04** para comparar enfoques y buenas prácticas

## 📦 Proyecto de Ejemplo

Los tutoriales construyen una aplicación web de **recetas e ingredientes** (la versión detallada usa este ejemplo) y una aplicación de **encuestas/polls** (tutorial oficial).

### Stack tecnológico
- Python 3.6+
- Django 5.0+
- SQLite (base de datos por defecto)

### Setup rápido

```bash
# Crear entorno virtual
python -m venv venv
venv/Scripts/activate  # Windows
# source venv/bin/activate  # Linux/macOS

# Instalar Django
pip install django

# Crear proyecto
django-admin startproject mysite .

# Ejecutar servidor
python manage.py runserver
```

## 📝 Temas Cubiertos

- Entornos virtuales y gestión de dependencias
- Estructura de proyectos y apps en Django
- Sistema de modelos y ORM
- Migraciones de base de datos
- Vistas: funciones vs clases
- Sistema de templates y context
- Enrutamiento de URLs y namespacing
- Panel de administración
- Formularios y validación
- Protección CSRF
- Vistas genéricas (CBV)
- Race conditions y operaciones atómicas
- Patrón Post/Redirect/Get

## 🔗 Recursos Adicionales

- [Documentación Oficial de Django](https://docs.djangoproject.com/)
- [Django Tutorial Oficial](https://docs.djangoproject.com/en/stable/intro/tutorial01/)

## 📄 Licencia

Esta guía está basada en el tutorial oficial de Django, expandida con explicaciones adicionales para fines educativos.

---

**Nota**: Esta guía está en desarrollo continuo. Faltan agregar las partes 6 y 7 del tutorial oficial (testing y archivos estáticos).
