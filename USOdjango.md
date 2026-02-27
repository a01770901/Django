# 📘 Guía Completa para Trabajar con Django usando MySQL

Autor: Tabata  
Proyecto: clasedjango  

---

# Creación de ambiente virtual y librerías necesarias

## Paso 1: Crear entorno virtual

```bash
python -m venv env
```

## Paso 2: Activar entorno virtual

### Windows
```bash
env\Scripts\activate
```

### Mac/Linux
```bash
source env/Scripts/activate
```

## 🔹 Paso 3: Instalar dependencias necesarias

```bash
pip install django
pip install mysqlclient
```

Verificar instalación:

```bash
django-admin --version
```

---

#  Crear base de datos manualmente (MySQL / Laragon)

En este proyecto **NO usamos SQLite**.  
La base de datos se crea manualmente en MySQL.

## 🔹 Paso 1: Abrir Laragon o MySQL

Entrar a:

- HeidiSQL
- phpMyAdmin
- MySQL Workbench

## 🔹 Paso 2: Crear base de datos

```sql
CREATE DATABASE clasedjango_db;
```

---

# Crear proyecto Django

```bash
django-admin startproject clasedjango
cd clasedjango
```

---

# Configurar conexión a MySQL

Editar:

```
clasedjango/settings.py
```

Modificar la sección `DATABASES`:

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'clasedjango_db',
        'USER': 'root',
        'PASSWORD': '',
        'HOST': '127.0.0.1',
        'PORT': '3306',
    }
}
```

---

# Crear tablas en la base de datos

```bash
python manage.py migrate
```

Esto crea automáticamente las tablas necesarias en MySQL.

---

# Crear una app

```bash
python manage.py startapp articles
```

Registrar en `settings.py`:

```python
INSTALLED_APPS = [
    ...
    'articles',
]
```

---

# Estructura del Proyecto

```
clasedjango/
│
├── manage.py
│
├── clasedjango/
│   ├── settings.py
│   ├── urls.py
│
├── articles/
│   ├── migrations/
│   ├── models.py
│   ├── views.py
│   ├── forms.py
│   ├── urls.py
│   ├── templates/
│   │    ├── article_template.html
│   │    ├── article_list.html
│   │    ├── new_article_form.html
│   │    └── article_detail.html
```

---

# Modelo (`models.py`)

```python
from django.db import models

class Article(models.Model):
    name = models.CharField(max_length=100)
    content = models.TextField()

    def __str__(self):
        return self.name
```

Migrar modelo:

```bash
python manage.py makemigrations
python manage.py migrate
```

---

# Formulario (`forms.py`)

```python
from .models import Article
from django.forms import ModelForm

class ArticleForm(ModelForm):
    class Meta:
        model = Article
        fields = "__all__"
```

---

# Vistas (`views.py`)

```python
from django.views.generic import ListView, DetailView
from django.shortcuts import render, redirect
from django.views import View
from .models import Article
from .forms import ArticleForm


class ArticleView(View):
    def get(self, request):
        return render(request, "article_template.html", {'data': 'hey data'})


class ArticleListView(ListView):
    model = Article
    context_object_name = "article_list"


class ArticleDetailView(DetailView):
    model = Article
    context_object_name = "article"


class NewArticleForm(View):

    def get(self, request):
        form = ArticleForm()
        return render(request, 'new_article_form.html', {'form': form})
    
    def post(self, request):
        form = ArticleForm(request.POST)

        if form.is_valid():
            form.save()
            return redirect("all_articles")
        
        return render(request, 'new_article_form.html', {"form": form})
```

---

# URLs de la App (`articles/urls.py`)

```python
from django.urls import path
from . import views

urlpatterns = [
    path('', views.ArticleView.as_view(), name="article_home"),
    path('all', views.ArticleListView.as_view(), name="all_articles"),
    path('new', views.NewArticleForm.as_view(), name="new_article"),
    path('<pk>', views.ArticleDetailView.as_view(), name="article_detail"),
]
```

---

# URL Principal (`clasedjango/urls.py`)

```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('articles/', include('articles.urls')),
]
```

---

# Templates

## article_list.html

```html
{% block content %}

<h2>Articles</h2>

<ul>
    {% for article in article_list %}
        <li>{{ article.name }}</li>
    {% endfor %}
</ul>

{% endblock %}
```

---

## new_article_form.html

```html
<h2>Nuevo Artículo</h2>

<form method="post">
    {% csrf_token %}
    {{ form.as_p }}
    <button type="submit">Guardar</button>
</form>
```

---

# Probar el proyecto

Ejecutar servidor:

```bash
python manage.py runserver
```

Abrir en navegador:

Home:
```
http://127.0.0.1:8000/articles/
```

Lista:
```
http://127.0.0.1:8000/articles/all
```

Crear:
```
http://127.0.0.1:8000/articles/new
```

Detalle:
```
http://127.0.0.1:8000/articles/1
```

---

# Referencias

* Documentación oficial de Django: https://docs.djangoproject.com/
* Práctica realizada en clase