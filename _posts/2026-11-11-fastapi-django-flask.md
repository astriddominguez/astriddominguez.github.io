---
layout: post
title: "FastAPI, Django y Flask: cuál elegir para tu backend en Python"
date: 2026-11-11 00:00:00 +0000
category: "Backend y seguridad"
tags: [python, fastapi, django, flask, frameworks, backend, principiantes]
published: true
---

Ya sabemos [qué es una API]({% post_url 2026-11-09-que-es-una-api %}), cómo se [documenta con Swagger]({% post_url 2026-11-09-swagger %}) y cómo se [describe con OpenAPI]({% post_url 2026-11-10-openapi %}). El siguiente paso es programarla. Y si lo haces en Python, enseguida te encuentras con la misma pregunta: **¿FastAPI, Django o Flask?**

Los tres son los frameworks de backend más usados en Python, y los tres sirven para crear una API. Pero cada uno tiene una forma de pensar distinta, y elegir bien te ahorra muchos dolores de cabeza. En este post te cuento cómo es cada uno, sus ventajas y desventajas, y cuándo elegir uno u otro.

## ¿Qué es un framework?

Un **framework** es un conjunto de herramientas y reglas que te dan hecho el trabajo repetitivo. Sin framework, para crear una API tendrías que leer tú las peticiones HTTP que llegan, averiguar a qué ruta van, convertir los datos a JSON, devolver el código de estado correcto… Con un framework, todo eso ya está resuelto, y tú solo escribes **lo que hace tu API**.

Siguiendo con el ejemplo del restaurante: el framework es la cocina ya montada, con los fogones, las neveras y los utensilios. Tú solo tienes que cocinar.

## Los tres en una frase

- **Django:** el framework "con pilas incluidas". Trae de serie casi todo lo que necesita una aplicación web completa.
- **Flask:** el microframework. Trae lo mínimo, y el resto lo añades tú según lo necesites.
- **FastAPI:** el más moderno. Pensado específicamente para crear APIs rápidas, con validación y documentación automáticas.

Para compararlos mejor, vamos a programar con los tres el mismo endpoint que usamos en los posts anteriores: `GET /usuarios/{id}`.

## Django

[Django](https://www.djangoproject.com) nació en 2005 en la redacción de un periódico, donde necesitaban crear webs completas muy rápido. Por eso su filosofía es darte **todo lo necesario** desde el primer momento:

- Un **ORM** para trabajar con la base de datos usando clases de Python en lugar de escribir SQL.
- **Migraciones**, para que los cambios en la base de datos queden registrados y se apliquen solos.
- Un **panel de administración** generado automáticamente, para ver y editar los datos desde el navegador.
- Sistema de **usuarios, login y permisos**.
- **Plantillas** para generar páginas HTML.
- Protecciones de **seguridad** activadas por defecto (contra CSRF, inyección SQL, XSS…).

Un proyecto de Django no es un solo archivo, sino una estructura de carpetas que se crea con un comando:

```bash
pip install django
django-admin startproject miproyecto
cd miproyecto
python manage.py startapp usuarios
```

Después escribes la **vista** (la función que responde a la petición):

```python
# usuarios/views.py
from django.http import JsonResponse

def obtener_usuario(request, id):
    return JsonResponse({"id": id, "nombre": "Astrid"})
```

Y la conectas a una ruta:

```python
# miproyecto/urls.py
from django.urls import path
from usuarios import views

urlpatterns = [
    path("usuarios/<int:id>", views.obtener_usuario),
]
```

Lo arrancas con `python manage.py runserver` y lo pruebas en [http://127.0.0.1:8000/usuarios/1](http://127.0.0.1:8000/usuarios/1).

Para crear APIs "de verdad" con Django casi todo el mundo usa una extensión, [Django REST Framework](https://www.django-rest-framework.org) (DRF), que añade serialización, validación, autenticación por tokens y una interfaz web para probar la API.

**Ventajas:**

- Lo tienes **todo incluido** y todo encaja entre sí. No tienes que buscar ni elegir librerías.
- El **panel de administración** es una maravilla: te ahorra programar un backoffice.
- Muy **seguro** por defecto.
- **Muy maduro**: lleva casi 20 años en producción en sitios como Instagram, y tiene una comunidad enorme y una documentación excelente.
- La estructura está marcada, así que cualquier persona que sepa Django se orienta rápido en tu proyecto.

**Desventajas:**

- **Curva de aprendizaje** más alta: hay muchas piezas y muchas convenciones que aprender.
- Es **pesado** para proyectos pequeños: para una API de tres endpoints, es como alquilar un camión para llevar una mochila.
- Es menos flexible: si quieres hacer las cosas de otra forma, vas "contra" el framework.
- Para APIs necesitas añadir DRF, que es otra cosa más que aprender.
- Aunque ya soporta código asíncrono, no se diseñó con eso en mente y no todo el framework lo aprovecha.

📖 **Documentación oficial:** [docs.djangoproject.com](https://docs.djangoproject.com/es/) (está en español). Su [tutorial](https://docs.djangoproject.com/es/stable/intro/tutorial01/) es de los mejores para empezar. Y para APIs, la de [Django REST Framework](https://www.django-rest-framework.org).

## Flask

[Flask](https://flask.palletsprojects.com) apareció en 2010 con la filosofía contraria: un **microframework**. Te da lo básico (rutas, peticiones y respuestas) y nada más. No trae base de datos, ni login, ni panel de administración. Si los necesitas, eliges la librería que más te guste y la añades.

Toda una API con Flask puede caber en un archivo:

```python
# main.py
from flask import Flask

app = Flask(__name__)

@app.get("/usuarios/<int:id>")
def obtener_usuario(id):
    return {"id": id, "nombre": "Astrid"}
```

```bash
pip install flask
flask --app main run
```

Y ya puedes probarlo en [http://127.0.0.1:5000/usuarios/1](http://127.0.0.1:5000/usuarios/1). Si devuelves un diccionario, Flask lo convierte a JSON solo.

Lo que no trae, lo añades con **extensiones**. Las más habituales son Flask-SQLAlchemy (base de datos), Flask-Migrate (migraciones), Flask-Login (sesiones de usuario) o Flask-JWT-Extended (tokens [JWT]({% post_url 2026-11-09-jwt %})).

**Ventajas:**

- **Muy fácil de empezar**: con cinco líneas tienes una API funcionando.
- **Flexible**: tú decides la estructura del proyecto y qué librerías usar.
- **Ligero**, ideal para proyectos pequeños, prototipos y microservicios.
- Perfecto para **aprender**, porque al no esconder casi nada, entiendes bien qué está pasando.
- También es muy maduro, con una comunidad grande y muchas extensiones.

**Desventajas:**

- Esa libertad tiene un precio: **tienes que tomar tú todas las decisiones**. En proyectos grandes, si no pones orden, cada uno acaba organizado de una forma distinta.
- **No valida los datos** que llegan: tienes que comprobarlos tú o añadir una librería.
- **No genera documentación** automáticamente: para tener Swagger necesitas una extensión.
- Depender de muchas extensiones de terceros significa más piezas que mantener y actualizar.
- Como Django, se diseñó para código síncrono. Permite funciones `async`, pero no es su punto fuerte.

📖 **Documentación oficial:** [flask.palletsprojects.com](https://flask.palletsprojects.com) (en inglés). Empieza por el [Quickstart](https://flask.palletsprojects.com/en/stable/quickstart/) y el [tutorial](https://flask.palletsprojects.com/en/stable/tutorial/).

## FastAPI

[FastAPI](https://fastapi.tiangolo.com) es el más joven de los tres (2018), y ha crecido muchísimo en pocos años. Como su nombre indica, está pensado **solo para APIs**, y se apoya en dos ideas:

- **Los tipos de Python** (`id: int`, `nombre: str`…). A partir de ellos, FastAPI valida los datos que llegan, los convierte al tipo correcto y genera la documentación.
- **Código asíncrono** (`async`/`await`), que permite atender muchas peticiones a la vez sin quedarse esperando, por ejemplo, a que responda la base de datos.

Es el que ya vimos en el post de Swagger:

```python
# main.py
from fastapi import FastAPI

app = FastAPI()

@app.get("/usuarios/{id}")
def obtener_usuario(id: int):
    return {"id": id, "nombre": "Astrid"}
```

```bash
pip install "fastapi[standard]"
fastapi dev main.py
```

Se parece mucho a Flask, pero fíjate en el `id: int`. Gracias a eso:

- Si pides `/usuarios/abc`, FastAPI responde automáticamente con un **422** explicando que `id` tiene que ser un número. No has escrito ni una línea para validarlo.
- En [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs) tienes ya tu **Swagger UI**, y en `/openapi.json` el archivo [OpenAPI]({% post_url 2026-11-10-openapi %}) generado a partir del código.

Para los datos que se envían en el cuerpo de la petición se usan modelos de **Pydantic**, que son clases donde describes los campos y sus tipos:

```python
from pydantic import BaseModel

class Usuario(BaseModel):
    nombre: str
    email: str
    edad: int | None = None

@app.post("/usuarios", status_code=201)
def crear_usuario(usuario: Usuario):
    return usuario
```

Si falta el `nombre` o la `edad` no es un número, la petición se rechaza sola con un error claro. Y el esquema `Usuario` aparece en la documentación igual que el que escribimos a mano en el post de OpenAPI.

**Ventajas:**

- **Validación y documentación automáticas** a partir de los tipos. Es su gran punto fuerte: menos código y la documentación siempre al día.
- **Muy rápido**, de los frameworks de Python con mejor rendimiento, gracias a que es asíncrono.
- **Ayuda del editor**: como todo tiene tipos, el editor te autocompleta y te avisa de errores antes de ejecutar.
- Muy fácil de empezar, casi tanto como Flask.
- **Documentación excelente** y con traducción al español.
- Es la opción más habitual para servir **modelos de IA y machine learning** como APIs.

**Desventajas:**

- Como Flask, **no trae base de datos, ni panel de administración, ni sistema de usuarios**. Tienes que añadirlos tú (por ejemplo, SQLAlchemy o SQLModel para la base de datos).
- Solo tiene sentido para **APIs**. Si quieres una web con páginas HTML, no es la herramienta pensada para eso.
- Es **más joven**: su comunidad y su ecosistema de librerías son más pequeños que los de Django o Flask.
- Para sacarle partido tienes que entender los **tipos** y la **programación asíncrona**, y lo asíncrono mal usado puede hacer que tu API vaya más lenta en lugar de más rápida.

📖 **Documentación oficial:** [fastapi.tiangolo.com/es](https://fastapi.tiangolo.com/es/) (con traducción al español). Su [tutorial](https://fastapi.tiangolo.com/es/tutorial/) va paso a paso y es muy fácil de seguir.

## Comparativa rápida

| | Django | Flask | FastAPI |
|---|---|---|---|
| **Tipo** | Completo ("pilas incluidas") | Microframework | Framework para APIs |
| **Año** | 2005 | 2010 | 2018 |
| **Ideal para** | Aplicaciones web completas | Proyectos pequeños y flexibles | APIs modernas |
| **Base de datos** | Incluida (ORM y migraciones) | Con extensiones | Con librerías externas |
| **Panel de administración** | Incluido | No | No |
| **Validación de datos** | Formularios / DRF | Tienes que añadirla | Automática (Pydantic) |
| **Documentación Swagger** | Con extensiones | Con extensiones | Automática |
| **Asíncrono** | Parcial | Parcial | Sí, de base |
| **Curva de aprendizaje** | Alta | Baja | Baja-media |
| **Documentación oficial** | [Django](https://docs.djangoproject.com/es/) | [Flask](https://flask.palletsprojects.com) | [FastAPI](https://fastapi.tiangolo.com/es/) |

## ¿Cuál elijo?

No hay un framework mejor que los demás. Hay uno **más adecuado para cada proyecto**. Estas son algunas pistas:

### Elige Django si…

- Vas a crear una **aplicación web completa**: con usuarios, base de datos, páginas y un panel para gestionar los datos.
- Necesitas un **backoffice** rápido: una tienda online, un blog con editores, una intranet, un CRM…
- El proyecto va a ser **grande** y lo va a mantener un equipo durante años. La estructura marcada ayuda a que todo el mundo trabaje igual.
- Prefieres que el framework tome las decisiones por ti y centrarte en tu aplicación.

### Elige Flask si…

- Estás **aprendiendo** cómo funciona un backend y quieres entender cada pieza.
- Es un proyecto **pequeño**: un prototipo, una herramienta interna, un microservicio sencillo o una web con pocas páginas.
- Quieres **libertad** total para elegir tus librerías y organizar el proyecto a tu manera.
- Trabajas en un proyecto que ya usa Flask (hay muchísimos).

### Elige FastAPI si…

- Lo que vas a construir es **una API** (por ejemplo, el backend de una app móvil o de un frontend en React, Angular o Vue).
- Quieres **validación y documentación automáticas** sin escribirlas a mano.
- El **rendimiento** importa o tu API va a recibir muchas peticiones a la vez.
- Vas a crear **microservicios** o a servir **modelos de IA o machine learning**.
- Empiezas un proyecto **nuevo** y quieres usar Python moderno con tipos.

### Resumiendo

| Si tu proyecto es… | Mi recomendación |
|---|---|
| Una web completa con usuarios y panel de administración | **Django** |
| Un proyecto grande con un equipo grande | **Django** |
| Una API para una app móvil o un frontend | **FastAPI** |
| Microservicios o una API con mucho tráfico | **FastAPI** |
| Servir un modelo de IA | **FastAPI** |
| Un prototipo o una herramienta pequeña | **Flask** o **FastAPI** |
| Tu primer backend, para aprender | **Flask** o **FastAPI** |

> Si estás empezando y no sabes cuál aprender primero, mi consejo es **FastAPI**: es fácil, te obliga a usar tipos (una buena costumbre) y lo que aprendes de APIs, validación y OpenAPI te sirve para cualquier otro framework.

## Para terminar

Django, Flask y FastAPI resuelven el mismo problema con filosofías distintas. **Django** te lo da todo hecho para construir aplicaciones web completas. **Flask** te da lo mínimo y libertad para construir a tu manera. **FastAPI** está pensado para APIs, y te regala la validación y la documentación a partir de los tipos de Python.

Lo más importante es que los conceptos son los mismos en los tres: rutas, métodos HTTP, peticiones, respuestas y códigos de estado. Una vez que domines uno, pasar a otro es cuestión de días.

Si quieres practicar, prueba a crear el endpoint `GET /usuarios/{id}` con los tres frameworks y compara cómo te sientes con cada uno. Y después, intenta añadir el `POST /usuarios` del post de OpenAPI: ahí es donde de verdad se notan las diferencias.
