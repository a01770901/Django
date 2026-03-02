# Proyecto Django con Bootstrap y jQuery  
## Validación de Formularios en Cliente y Servidor


# Objetivo del Proyecto

Desarrollar una aplicación web usando **Django** que integre:

- Diseño moderno con Bootstrap
- Validación en el cliente usando jQuery
- Validación robusta en el servidor con Django Forms
- Mensajes de error personalizados
- Almacenamiento en base de datos

Tecnologías utilizadas:

- Django
- Bootstrap 5
- jQuery
- base de datos


# Concepto Clave: Doble Validación

En aplicaciones web reales es obligatorio validar datos en dos niveles:

```
mi_proyecto/
│
├── mi_proyecto/
├── usuarios/
│   ├── templates/
│   │   └── usuarios/
│   │       ├── base.html
│   │       ├── registro.html
│   │       └── exito.html
│   ├── models.py
│   ├── forms.py
│   ├── views.py
│   └── urls.py
```

---

# 1. Creación del Proyecto

## Crear entorno virtual

```bash
python -m venv env
```

Activar entorno:

Windows:
```bash
env\Scripts\activate
```

Instalar Django:

```bash
pip install django
```

Crear proyecto:

```bash
django-admin startproject mi_proyecto
cd mi_proyecto
```

Crear aplicación:

```bash
python manage.py startapp usuarios
```

Registrar app en `settings.py`:

```python
INSTALLED_APPS = [
    ...
    'usuarios',
]
```

---

# 2. Modelo de Base de Datos

Archivo: `models.py`

```python
from django.db import models

class Usuario(models.Model):
    nombre = models.CharField(max_length=100)
    email = models.EmailField()
    edad = models.IntegerField()

    def __str__(self):
        return self.nombre
```

Migraciones:

```bash
python manage.py makemigrations
python manage.py migrate
```

---

# 3. Formulario con Validaciones Backend

Archivo: `forms.py`

```python
from django import forms
from .models import Usuario

class UsuarioForm(forms.ModelForm):
    class Meta:
        model = Usuario
        fields = ['nombre', 'email', 'edad']

    def clean_nombre(self):
        nombre = self.cleaned_data.get('nombre')
        if len(nombre) < 3:
            raise forms.ValidationError(
                "El nombre debe tener al menos 3 caracteres."
            )
        return nombre

    def clean_edad(self):
        edad = self.cleaned_data.get('edad')
        if edad < 18:
            raise forms.ValidationError(
                "Debes ser mayor de edad."
            )
        return edad
```

---

# ¿En qué se basaron las validaciones?

Las validaciones fueron diseñadas bajo estos criterios:

## 🔹 Validación de nombre (mínimo 3 caracteres)

**Justificación:**

- Evita registros inválidos como "A" o "Jo".
- Garantiza que el dato tenga significado.
- Es una validación común en sistemas reales.

Se implementó usando el método:

```
clean_nombre()
```

Porque Django permite validar campos específicos usando el patrón:

```
clean_<nombre_del_campo>()
```

---

## 🔹 Validación de edad (mayor o igual a 18)

**Justificación:**

- Simula una regla de negocio real.
- Ejemplo: plataformas que requieren mayoría de edad.
- Representa validación lógica de dominio.

Se valida con:

```
clean_edad()
```

---

## 🔹 Validación de email

No fue necesario crear validación manual porque:

- `EmailField()` ya incluye validación automática de formato.
- Django verifica que el email tenga estructura válida.

Esto demuestra uso correcto de validaciones integradas del framework.

---

#  4. Vista

Archivo: `views.py`

```python
from django.shortcuts import render, redirect
from .forms import UsuarioForm

def registro(request):
    if request.method == 'POST':
        form = UsuarioForm(request.POST)
        if form.is_valid():
            form.save()
            return redirect('exito')
    else:
        form = UsuarioForm()

    return render(request, 'usuarios/registro.html', {'form': form})

def exito(request):
    return render(request, 'usuarios/exito.html')
```

---

#  5. URLs

`usuarios/urls.py`

```python
from django.urls import path
from . import views

urlpatterns = [
    path('', views.registro, name='registro'),
    path('exito/', views.exito, name='exito'),
]
```

`urls.py` principal:

```python
from django.urls import path, include

urlpatterns = [
    path('', include('usuarios.urls')),
]
```

---

#  6. Integración de Bootstrap y jQuery

Archivo: `base.html`

```html
<!DOCTYPE html>
<html>
<head>
    <title>{% block title %}{% endblock %}</title>

    <!-- Bootstrap -->
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/css/bootstrap.min.css" rel="stylesheet">

    <!-- jQuery -->
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body class="container mt-5">

{% block content %}
{% endblock %}

</body>
</html>
```

---

# 7. Formulario con Validación Frontend

Archivo: `registro.html`

```html
{% extends 'usuarios/base.html' %}

{% block content %}

<h2>Registro de Usuario</h2>

<form method="POST" id="formRegistro" novalidate>
    {% csrf_token %}

    <div class="mb-3">
        <label>Nombre</label>
        {{ form.nombre }}
        <div class="text-danger">
            {{ form.nombre.errors }}
        </div>
    </div>

    <div class="mb-3">
        <label>Email</label>
        {{ form.email }}
        <div class="text-danger">
            {{ form.email.errors }}
        </div>
    </div>

    <div class="mb-3">
        <label>Edad</label>
        {{ form.edad }}
        <div class="text-danger">
            {{ form.edad.errors }}
        </div>
    </div>

    <button type="submit" class="btn btn-primary">Registrarse</button>
</form>

<script>
$(document).ready(function(){
    $("#formRegistro").submit(function(){
        if($("#id_nombre").val() == ""){
            alert("El nombre es obligatorio");
            return false;
        }
    });
});
</script>

{% endblock %}
```

---

# Flujo Completo de Validación

1. Usuario llena formulario.
2. jQuery verifica nombre vacío.
3. Si pasa, se envía al servidor.
4. Django ejecuta `is_valid()`.
5. Se ejecutan `clean_nombre()` y `clean_edad()`.
6. Si todo es válido → se guarda en base de datos.
7. Se redirige a página de éxito.

---

# Ejecutar Proyecto

```bash
python manage.py runserver
```

Abrir:

```
http://127.0.0.1:8000/
```

---
