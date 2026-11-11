---
layout: post
title: "Ejercicio: una agenda de contactos con OpenAPI, Postman, un mock y FastAPI"
date: 2026-11-11 01:00:00 +0000
category: "Ejercicios prácticos"
tags: [ejercicio, api, openapi, swagger, postman, mock, fastapi, python, git, buenas-practicas]
published: true
---

En los últimos posts hemos visto [qué es una API]({% post_url 2026-11-09-que-es-una-api %}), cómo se [prueba con Swagger]({% post_url 2026-11-09-swagger %}), cómo se [describe con OpenAPI]({% post_url 2026-11-10-openapi %}) y [qué framework elegir]({% post_url 2026-11-11-fastapi-django-flask %}) para programarla. Toca juntarlo todo en un proyecto.

En este ejercicio vas a crear la API de una **agenda de contactos**, y la vas a construir **de dos maneras**:

- **Design-first (primero el diseño):** escribes a mano el contrato OpenAPI, en YAML y en JSON, levantas un **servidor falso (mock)** que responde siguiendo ese contrato y pruebas todas las peticiones con **Postman**. Todo ello sin haber escrito ni una línea de código de la API.
- **Code-first (primero el código):** programas la misma API con **FastAPI** y dejas que genere el Swagger por ti. Después comparas el contrato generado con el que escribiste a mano, y lanzas la misma colección de Postman contra la API de verdad.

Y como siempre, lo haremos con buenas prácticas: una rama para cada fase, commits con [Conventional Commits]({% post_url 2026-11-05-git-conventional-commits %}), `main` protegida y una [CI]({% post_url 2026-11-10-ci-cd %}) que lo comprueba todo. En cada paso te recordaré qué buena práctica estás aplicando y por qué.

## Antes de empezar

**Necesitas:**

- [Git](https://git-scm.com) instalado y configurado, y una cuenta de [GitHub](https://github.com).
- Python 3.12 o superior.
- [Node.js](https://nodejs.org) (la versión LTS), para el servidor mock. Compruébalo con `node --version`.
- [Postman](https://www.postman.com/downloads/), la aplicación de escritorio. La versión web también sirve, pero para hacer peticiones a `localhost` necesitarías instalar además su agente.

**Conviene haber leído antes:**

- [OpenAPI: cómo describir una API en un archivo]({% post_url 2026-11-10-openapi %}). Es la base de toda la primera parte.
- [Ejercicio: una calculadora en Python con Git, tests y CI/CD]({% post_url 2026-11-10-ejercicio-calculadora-buenas-practicas %}). Aquí repasaremos la preparación del proyecto más rápido, porque allí está explicada paso a paso.

**Cómo funciona el ejercicio:** cada fase tiene un objetivo y unas pistas. Intenta hacerlo por tu cuenta y, si te atascas, despliega la **solución**.

### La API que vamos a construir

| Método | Ruta | Qué hace | Respuestas |
|---|---|---|---|
| GET | `/contactos` | Lista los contactos. Con `?nombre=ana` filtra por nombre | 200 |
| POST | `/contactos` | Crea un contacto | 201, 422 |
| GET | `/contactos/{id}` | Devuelve un contacto | 200, 404 |
| PUT | `/contactos/{id}` | Actualiza un contacto | 200, 404, 422 |
| DELETE | `/contactos/{id}` | Borra un contacto | 204, 404 |

Un contacto tiene un `id` (lo pone el servidor), un `nombre` y un `telefono` (obligatorios) y un `email` (opcional).

### El proyecto final

```
agenda-api/
├── .github/
│   └── workflows/
│       └── ci.yml                     # CI: linter, contrato y tests
├── app/
│   ├── __init__.py
│   └── main.py                        # fase 4: la API con FastAPI
├── contrato/
│   ├── openapi.yaml                   # fase 1: escrito a mano
│   ├── openapi.json                   # fase 1: el mismo, en JSON
│   └── openapi-generado.json          # fase 5: el que genera FastAPI
├── postman/
│   ├── agenda.postman_collection.json # fase 3
│   ├── mock.postman_environment.json
│   └── local.postman_environment.json
├── scripts/
│   ├── yaml_a_json.py
│   └── exportar_openapi.py
├── tests/
│   └── test_contactos.py
├── .gitignore
├── pyproject.toml
├── README.md
├── requirements.txt
└── requirements-dev.txt
```

Y este es el plan, con la rama de cada fase:

| Fase | Qué haces | Rama | Versión |
|---|---|---|---|
| 0 | Preparar el proyecto | `main` (solo el primer commit) | |
| 1 | Escribir el contrato en YAML y JSON | `feat/contrato-openapi` | `v0.1.0` |
| 2 | Levantar el servidor mock | `docs/servidor-mock` | |
| 3 | Probar todo con Postman | `test/coleccion-postman` | |
| 4 | Programar la API con FastAPI | `feat/api-fastapi` | |
| 5 | Comparar los contratos y probar la API real | `docs/contrato-generado` | `v1.0.0` |

## Fase 0: preparar el proyecto

Esta fase es igual que en la calculadora, así que vamos a ir rápido. Si algo no te suena, repasa la [fase 0 del ejercicio de la calculadora]({% post_url 2026-11-10-ejercicio-calculadora-buenas-practicas %}#fase-0-preparar-el-proyecto).

1. Crea la carpeta `agenda-api`, inicia Git con la rama `main` y crea y activa un entorno virtual.
2. Crea `requirements.txt` (de momento vacío, con un comentario) y `requirements-dev.txt` con las herramientas de desarrollo que usaremos al principio:
   - **ruff**, el linter.
   - **pyyaml**, para leer YAML desde Python.
   - **openapi-spec-validator**, que comprueba que un archivo OpenAPI es correcto.
3. Crea el `pyproject.toml`, el `.gitignore` y un `README.md` corto.
4. Haz el primer commit, crea el repositorio **público** y vacío `agenda-api` en GitHub y sube el código.

> 💡 **Buena práctica:** las dependencias de la aplicación y las de desarrollo van separadas, y con las versiones fijadas. Así tu ordenador y la CI usan exactamente lo mismo.

<details markdown="1">
<summary>Ver solución</summary>

```bash
mkdir agenda-api
cd agenda-api
git init -b main
python3 -m venv .venv
source .venv/bin/activate      # en Windows: .venv\Scripts\activate
```

```
# requirements.txt
# Dependencias de la aplicación. Las añadiremos en la fase 4.
```

```
# requirements-dev.txt
-r requirements.txt
ruff
pyyaml
openapi-spec-validator
```

```bash
pip install -r requirements-dev.txt
pip freeze    # apunta las versiones en requirements-dev.txt (ruff==..., etc.)
```

```toml
# pyproject.toml
[tool.pytest.ini_options]
pythonpath = ["."]
testpaths = ["tests"]

[tool.ruff]
line-length = 100
```

```gitignore
# .gitignore
.venv/
__pycache__/
*.pyc
.pytest_cache/
.ruff_cache/
.coverage
htmlcov/
node_modules/
.vscode/
.idea/
.DS_Store
```

He añadido `node_modules/` por si instalas Prism en el proyecto en lugar de usarlo con `npx`.

```bash
git add .
git commit -m "chore: prepara la estructura del proyecto"
git remote add origin https://github.com/TU_USUARIO/agenda-api.git
git push -u origin main
```

</details>

A partir de aquí, **no vuelvas a trabajar directamente en `main`**. Cada fase va en su propia rama y entra en `main` con un pull request.

---

# Parte A: design-first

En design-first, lo primero es **ponerse de acuerdo en el contrato**. Imagina que en tu equipo hay una persona haciendo el frontend y otra el backend: si las dos tienen el archivo OpenAPI, pueden empezar a trabajar a la vez, sin esperarse.

## Fase 1: el contrato en YAML

Crea la rama `feat/contrato-openapi` y, dentro de la carpeta `contrato/`, el archivo `openapi.yaml`. Te recomiendo escribirlo en [Swagger Editor](https://editor.swagger.io), que te va marcando los errores y te enseña cómo queda en Swagger UI mientras escribes, y copiarlo después al archivo.

> 💡 **Buena práctica:** el nombre de la rama dice qué tipo de cambio es (`feat`) y de qué trata (`contrato-openapi`). Cualquiera que vea la lista de ramas sabe en qué estás trabajando.

### 1.1. La cabecera

Empieza con `openapi`, `info` y `servers`. Pon **dos** servidores: el mock, que estará en `http://127.0.0.1:4010`, y la API real, que estará en `http://127.0.0.1:8000`.

<details markdown="1">
<summary>Ver solución</summary>

```yaml
openapi: 3.1.0
info:
  title: Agenda de contactos
  description: API para gestionar una agenda de contactos.
  version: 1.0.0
servers:
  - url: http://127.0.0.1:4010
    description: Servidor mock (Prism)
  - url: http://127.0.0.1:8000
    description: API real (FastAPI)
```

En Swagger UI aparecerá un desplegable para elegir a qué servidor mandar las peticiones.

</details>

### 1.2. Los esquemas

Define en `components/schemas` los modelos de datos. Esta vez vamos a usar **dos esquemas** en lugar de uno con `readOnly` como en el post de OpenAPI:

- **`ContactoNuevo`**: lo que se envía para crear o actualizar un contacto. `nombre` y `telefono` son obligatorios, y `email` es opcional, con `format: email`.
- **`Contacto`**: lo que devuelve la API. Es lo mismo que `ContactoNuevo` **más** el `id`.

Añade también un esquema **`Error`** con un único campo, `detail`, de tipo texto. Será lo que devuelva la API cuando algo vaya mal.

Pon un `example` en cada campo: el servidor mock los usará para responder.

**Pista:** para decir "lo mismo que otro esquema, más algo", OpenAPI tiene `allOf`, que combina varios esquemas en uno.

<details markdown="1">
<summary>Ver solución</summary>

```yaml
components:
  schemas:
    ContactoNuevo:
      type: object
      required: [nombre, telefono]
      properties:
        nombre:
          type: string
          minLength: 1
          example: Ana García
        telefono:
          type: string
          example: "600123456"
        email:
          type: string
          format: email
          example: ana@ejemplo.com

    Contacto:
      allOf:
        - $ref: '#/components/schemas/ContactoNuevo'
        - type: object
          required: [id]
          properties:
            id:
              type: integer
              example: 1

    Error:
      type: object
      properties:
        detail:
          type: string
          example: Contacto no encontrado
```

Algunos detalles:

- El teléfono es un `string`, no un `integer`. Los teléfonos no se suman ni se restan, pueden empezar por `0` o llevar un `+`. Y el ejemplo va entre comillas para que YAML no lo lea como un número.
- `allOf` significa "tiene que cumplir **todos** estos esquemas a la vez": los campos de `ContactoNuevo` y además el `id`.
- ¿Por qué el error usa el campo `detail`? Porque es el formato que usa FastAPI para sus errores. Como ya sabemos que la API la programaremos con FastAPI, diseñamos el contrato para que encaje. En un proyecto real, esta decisión se toma en el equipo.

</details>

### 1.3. Piezas reutilizables: parámetros y respuestas

En `components` no solo se guardan esquemas. También puedes guardar **parámetros** y **respuestas** que se repiten:

- El parámetro `id` de la ruta aparece en tres endpoints (GET, PUT y DELETE de `/contactos/{id}`).
- La respuesta 404 aparece en esos mismos tres.
- La respuesta 422 (datos incorrectos) aparece en el POST y en el PUT.

Defínelos una vez en `components/parameters` y `components/responses`.

<details markdown="1">
<summary>Ver solución</summary>

Esto va dentro de `components`, al mismo nivel que `schemas`:

```yaml
  parameters:
    ContactoId:
      name: id
      in: path
      required: true
      description: El id del contacto
      schema:
        type: integer

  responses:
    NoEncontrado:
      description: No hay ningún contacto con ese id
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/Error'
    DatosIncorrectos:
      description: Faltan datos o son incorrectos
```

En el post de OpenAPI usamos el 400 para los datos incorrectos. Aquí usamos el **422** (*Unprocessable Content*: "he entendido la petición, pero los datos no son válidos") porque es el que usa FastAPI. Los dos se ven mucho en APIs reales.

</details>

### 1.4. Los endpoints

Ahora escribe los `paths` con los cinco endpoints de la tabla del principio. Usa `$ref` para todo lo que hayas definido en `components`.

Dos pistas:

- El parámetro `nombre` del listado va en la *query* (`in: query`) y **no** es obligatorio.
- Si todos los métodos de una ruta comparten un parámetro, puedes ponerlo **una sola vez** en la ruta, en lugar de repetirlo en cada método.

<details markdown="1">
<summary>Ver solución</summary>

```yaml
paths:
  /contactos:
    get:
      summary: Listar contactos
      operationId: listarContactos
      parameters:
        - name: nombre
          in: query
          required: false
          description: Devuelve solo los contactos cuyo nombre contenga este texto
          schema:
            type: string
      responses:
        '200':
          description: La lista de contactos
          content:
            application/json:
              schema:
                type: array
                items:
                  $ref: '#/components/schemas/Contacto'
    post:
      summary: Crear un contacto
      operationId: crearContacto
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/ContactoNuevo'
      responses:
        '201':
          description: Contacto creado
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Contacto'
        '422':
          $ref: '#/components/responses/DatosIncorrectos'

  /contactos/{id}:
    parameters:
      - $ref: '#/components/parameters/ContactoId'
    get:
      summary: Obtener un contacto
      operationId: obtenerContacto
      responses:
        '200':
          description: El contacto
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Contacto'
        '404':
          $ref: '#/components/responses/NoEncontrado'
    put:
      summary: Actualizar un contacto
      operationId: actualizarContacto
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/ContactoNuevo'
      responses:
        '200':
          description: Contacto actualizado
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Contacto'
        '404':
          $ref: '#/components/responses/NoEncontrado'
        '422':
          $ref: '#/components/responses/DatosIncorrectos'
    delete:
      summary: Borrar un contacto
      operationId: borrarContacto
      responses:
        '204':
          description: Contacto borrado
        '404':
          $ref: '#/components/responses/NoEncontrado'
```

`operationId` es un nombre único para cada operación. Es opcional, pero muchas herramientas lo usan: Postman, por ejemplo, para nombrar las peticiones, y los generadores de código para nombrar las funciones.

El archivo completo es la unión de los tres bloques: la cabecera, `paths` y `components`.

</details>

### 1.5. Valídalo y haz commit

Pégalo entero en Swagger Editor y comprueba que no marca ningún error. Después compruébalo también desde la terminal:

```bash
openapi-spec-validator contrato/openapi.yaml
```

Si todo está bien, dirá `OK`. Haz commit del contrato.

> 💡 **Buena práctica:** un commit es **un cambio con sentido**. El contrato es una unidad completa, así que va en su propio commit, separado de lo que hagas después.

<details markdown="1">
<summary>Ver solución</summary>

```bash
git switch -c feat/contrato-openapi
# ... escribes contrato/openapi.yaml ...
git add contrato/openapi.yaml
git commit -m "feat: añade el contrato OpenAPI de la agenda"
```

</details>

### 1.6. El mismo contrato en JSON

YAML y JSON son **dos formas de escribir lo mismo**. Para comprobarlo, crea un script `scripts/yaml_a_json.py` que lea `contrato/openapi.yaml` y lo guarde como `contrato/openapi.json`.

Ábrelos uno al lado del otro: mismas claves, mismos valores; solo cambian las llaves, las comillas y las comas. Y valida también el JSON con `openapi-spec-validator`.

**Pista:** `yaml.safe_load()` convierte YAML en diccionarios de Python, y `json.dumps()` convierte esos diccionarios en JSON.

<details markdown="1">
<summary>Ver solución</summary>

```python
# scripts/yaml_a_json.py
"""Convierte contrato/openapi.yaml en contrato/openapi.json."""

import json
from pathlib import Path

import yaml

ORIGEN = Path("contrato/openapi.yaml")
DESTINO = Path("contrato/openapi.json")

contrato = yaml.safe_load(ORIGEN.read_text(encoding="utf-8"))
DESTINO.write_text(json.dumps(contrato, indent=2, ensure_ascii=False) + "\n", encoding="utf-8")
print(f"Generado {DESTINO}")
```

```bash
python scripts/yaml_a_json.py
openapi-spec-validator contrato/openapi.json
git add scripts/yaml_a_json.py contrato/openapi.json
git commit -m "feat: añade la versión JSON del contrato"
```

Esta parte del JSON te sonará:

```json
"/contactos/{id}": {
  "parameters": [
    { "$ref": "#/components/parameters/ContactoId" }
  ],
  "get": {
    "summary": "Obtener un contacto",
    ...
```

Ojo: a partir de ahora, **el YAML es el original** y el JSON se genera a partir de él. Si cambias el contrato, cambia el YAML y vuelve a ejecutar el script. Nunca edites los dos a mano, o acabarán siendo distintos.

</details>

### 1.7. Que la CI valide el contrato

Añade en la misma rama el archivo `.github/workflows/ci.yml`. Tiene que ejecutarse en cada push a `main` y en cada pull request, y comprobar dos cosas:

1. Que el código pasa el linter: `ruff check .`
2. Que el contrato es válido: `openapi-spec-validator contrato/openapi.yaml`

Llama al job `ci`.

<details markdown="1">
<summary>Ver solución</summary>

```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [main]
  pull_request:

jobs:
  ci:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v5
      - uses: actions/setup-python@v6
        with:
          python-version: "3.13"
          cache: pip
      - run: pip install -r requirements-dev.txt
      - run: ruff check .
      - run: openapi-spec-validator contrato/openapi.yaml
```

```bash
git add .github/workflows/ci.yml
git commit -m "ci: valida el contrato OpenAPI y pasa el linter"
git push -u origin feat/contrato-openapi
```

</details>

### 1.8. Pull request, protección de main y primera versión

1. Abre un **pull request** de `feat/contrato-openapi` hacia `main`. Fíjate en que tiene **tres commits**, y en que la CI se ejecuta sola.
2. Cuando esté en verde, fusiónalo y borra la rama.
3. Protege `main` como en la [fase 1.3 de la calculadora]({% post_url 2026-11-10-ejercicio-calculadora-buenas-practicas %}#13-protege-main), exigiendo que el check `ci` esté en verde.
4. Crea la etiqueta **`v0.1.0`**: es la primera versión del contrato, aunque todavía no haya API.

> 💡 **Buena práctica:** la versión empieza por `0` porque la API todavía no existe y el contrato puede cambiar. La `1.0.0` llegará cuando la API funcione y cumpla el contrato.

<details markdown="1">
<summary>Ver solución</summary>

```bash
git switch main
git pull
git branch -d feat/contrato-openapi
git tag -a v0.1.0 -m "v0.1.0: primera versión del contrato"
git push origin v0.1.0
```

</details>

## Fase 2: un servidor falso (mock)

Tenemos un contrato, pero ninguna API que lo cumpla. Y sin embargo, ya podemos tener **un servidor que responde**.

Un **servidor mock** lee el archivo OpenAPI y responde a las peticiones como si fuera la API de verdad, usando los `example` del contrato. No guarda nada ni tiene lógica: solo finge. Pero eso basta para que el frontend pueda empezar a trabajar, o para probar que el contrato tiene sentido antes de programarlo.

Vamos a usar [Prism](https://docs.stoplight.io/docs/prism/), una herramienta de código abierto que hace justo esto.

### 2.1. Levanta el mock

Crea la rama `docs/servidor-mock` y, desde la raíz del proyecto, ejecuta:

```bash
npx @stoplight/prism-cli mock contrato/openapi.yaml
```

`npx` descarga Prism y lo ejecuta sin instalarlo. Verás una lista con todos tus endpoints y la dirección `http://127.0.0.1:4010`. Déjalo funcionando en esa terminal y abre otra.

### 2.2. Pruébalo con curl

Haz estas peticiones y fíjate en qué responde:

```bash
# Lista de contactos
curl http://127.0.0.1:4010/contactos

# Un contacto
curl http://127.0.0.1:4010/contactos/1

# Crear un contacto
curl -X POST http://127.0.0.1:4010/contactos \
  -H "Content-Type: application/json" \
  -d '{"nombre": "Luis", "telefono": "611222333"}'

# Crear un contacto SIN teléfono
curl -i -X POST http://127.0.0.1:4010/contactos \
  -H "Content-Type: application/json" \
  -d '{"nombre": "Luis"}'
```

Y responde a estas preguntas:

1. ¿De dónde salen los datos de las respuestas?
2. ¿Qué pasa con la petición sin teléfono? ¿Por qué?
3. Pide `/contactos/999`. ¿Responde con un 404? ¿Por qué?
4. Haz un DELETE de `/contactos/1` y después un GET de `/contactos/1`. ¿Qué pasa?

<details markdown="1">
<summary>Ver solución</summary>

1. De los **`example`** del contrato. Por eso siempre te devuelve a Ana García, aunque hayas creado a Luis.
2. Prism **valida la petición** contra el contrato. Como `telefono` es obligatorio en `ContactoNuevo`, responde con un **422** y explica qué falta. ¡Y sin haber programado ninguna validación!
3. No: devuelve un 200 con Ana García. El mock no tiene datos, así que no sabe que el contacto 999 no existe. Responde con la primera respuesta correcta que encuentra en el contrato.
4. El DELETE responde 204, pero el GET sigue devolviendo a Ana. **Un mock no tiene memoria**: cada petición es independiente.

Para probar respuestas concretas, como el 404, puedes pedírselas a Prism con una cabecera:

```bash
curl -i http://127.0.0.1:4010/contactos/999 -H "Prefer: code=404"
```

Y si quieres datos inventados en lugar de los ejemplos, arráncalo con `-d` (de *dynamic*):

```bash
npx @stoplight/prism-cli mock -d contrato/openapi.yaml
```

</details>

### 2.3. Documéntalo

Añade al `README.md` una sección "Servidor mock" que explique cómo levantarlo y en qué dirección queda. Haz commit, pull request y fusiona.

> 💡 **Buena práctica:** si algo solo lo sabes tú, no está documentado. El README es lo primero que lee cualquiera que llegue al proyecto, también tú dentro de seis meses.

<details markdown="1">
<summary>Ver solución</summary>

```markdown
## Servidor mock

Para probar la API antes de que exista, levanta un servidor falso a partir del contrato
(necesitas Node.js):

    npx @stoplight/prism-cli mock contrato/openapi.yaml

Quedará disponible en http://127.0.0.1:4010 y responderá con los ejemplos del contrato.
```

```bash
git switch -c docs/servidor-mock
git add README.md
git commit -m "docs: explica cómo levantar el servidor mock"
git push -u origin docs/servidor-mock
```

El tipo es `docs` porque solo cambia la documentación. Después del merge, vuelve a `main`, haz `git pull` y borra la rama.

</details>

## Fase 3: probar todas las peticiones con Postman

curl está bien para una petición suelta, pero para probar una API entera es más cómodo [Postman](https://learning.postman.com/docs/introduction/overview/): guardas las peticiones, las organizas y puedes lanzarlas todas de golpe con tests incluidos.

Lo mejor es que **no hace falta escribir las peticiones**: Postman las crea solas a partir del archivo OpenAPI.

Crea la rama `test/coleccion-postman` y deja el mock funcionando.

### 3.1. Importa el contrato

1. En Postman, pulsa **Import** y arrastra el archivo `contrato/openapi.yaml`.
2. Elige importarlo como **Postman Collection**.
3. Se creará una colección **Agenda de contactos** con una petición por cada endpoint, ya con sus parámetros y con cuerpos de ejemplo.

Ábrela y fíjate en que las URLs empiezan por `{{baseUrl}}`. Es una **variable**: Postman la sustituye por su valor al enviar la petición.

### 3.2. Dos entornos: mock y local

Un **entorno** (*environment*) es un conjunto de variables. Crea dos:

| Entorno | Variable | Valor |
|---|---|---|
| `Mock` | `baseUrl` | `http://127.0.0.1:4010` |
| `Local` | `baseUrl` | `http://127.0.0.1:8000` |

Elige el entorno **Mock** en el desplegable de arriba a la derecha y envía todas las peticiones de la colección una a una. Todas deberían responder.

> 💡 **Buena práctica:** nunca escribas una URL, un usuario o un token directamente en las peticiones. Ponlos en variables. Así, cambiar de servidor es cambiar de entorno, sin tocar ninguna petición.

### 3.3. Encadena las peticiones

En una API real, para obtener, actualizar o borrar un contacto necesitas su `id`, y ese `id` te lo da el POST al crearlo. Vamos a hacer que Postman lo recuerde:

1. En la petición **Crear un contacto**, abre la pestaña **Scripts → Post-response** y guarda el `id` de la respuesta en una variable del entorno llamada `contactoId`.
2. En las peticiones de `/contactos/:id`, cambia el valor del parámetro `id` por `{{contactoId}}`.

<details markdown="1">
<summary>Ver solución</summary>

```javascript
// Crear un contacto → Scripts → Post-response
pm.environment.set("contactoId", pm.response.json().id);
```

En las peticiones de `/contactos/:id`, en la pestaña **Params**, verás `id` en *Path Variables*. Escribe `{{contactoId}}` como valor.

</details>

### 3.4. Añade tests

En la misma pestaña **Post-response** puedes escribir **tests**: comprobaciones que se ejecutan al recibir la respuesta. Añade al menos uno a cada petición que compruebe el código de estado, y alguno más interesante. Por ejemplo, que el contacto creado tiene un `id`, o que el listado es una lista.

<details markdown="1">
<summary>Ver solución</summary>

```javascript
// Crear un contacto
pm.test("Responde con 201", () => {
    pm.response.to.have.status(201);
});
pm.test("El contacto tiene id", () => {
    pm.expect(pm.response.json()).to.have.property("id");
});
pm.environment.set("contactoId", pm.response.json().id);
```

```javascript
// Listar contactos
pm.test("Responde con 200", () => {
    pm.response.to.have.status(200);
});
pm.test("Devuelve una lista", () => {
    pm.expect(pm.response.json()).to.be.an("array");
});
```

```javascript
// Borrar un contacto
pm.test("Responde con 204", () => {
    pm.response.to.have.status(204);
});
```

Los demás son iguales, cambiando el código de estado.

</details>

### 3.5. Lánzalas todas: el Collection Runner

1. Ordena las peticiones de la colección arrastrándolas: **Crear → Listar → Obtener → Actualizar → Borrar**. El orden importa, porque las últimas usan el `contactoId` que guarda la primera.
2. Pulsa los tres puntos de la colección → **Run collection** y lánzala con el entorno **Mock**.

Verás todas las peticiones y sus tests, en verde o en rojo, en una sola pantalla.

**Reto:** duplica la petición **Obtener un contacto**, llámala **Obtener un contacto borrado**, ponla al final y escribe un test que espere un **404**. Lánzalo todo contra el mock. ¿Qué pasa? Acuérdate de la fase 2.

<details markdown="1">
<summary>Ver solución</summary>

El test del 404 **falla** contra el mock: como no tiene memoria, sigue devolviendo a Ana García aunque la hayas borrado. No es un error de tu colección: es el límite de un mock. Ese test pasará en la fase 5, contra la API real.

</details>

### 3.6. Guarda la colección en el repositorio

Exporta la colección (formato **Collection v2.1**) y los dos entornos, guárdalos en la carpeta `postman/` y haz commit.

> 💡 **Buena práctica:** la colección es parte del proyecto. Si está en el repositorio, cualquiera del equipo la importa y prueba la API en un minuto. Pero **cuidado con los secretos**: si algún día pones un token o una contraseña en un entorno, ese archivo **no** se sube.

<details markdown="1">
<summary>Ver solución</summary>

```bash
git switch -c test/coleccion-postman
# ... exportas los archivos a postman/ ...
git add postman/
git commit -m "test: añade la colección de Postman y sus entornos"
git push -u origin test/coleccion-postman
```

El tipo es `test` porque la colección sirve para probar la API. Pull request, merge, `git pull` en `main` y borra la rama.

</details>

---

# Parte B: code-first

Ahora vamos a hacer **lo mismo al revés**. Olvídate por un momento del contrato: vas a programar la API directamente con FastAPI y dejar que él genere el contrato a partir del código.

Usaremos los mismos nombres (`ContactoNuevo`, `Contacto`…) para poder comparar los dos contratos al final, pero no copies nada del YAML: programa la API a partir de la tabla del principio.

## Fase 4: la API con FastAPI

### 4.1. Las dependencias

Crea la rama `feat/api-fastapi` y añade las dependencias nuevas:

- En `requirements.txt`: `fastapi[standard]`, que incluye FastAPI, el servidor y las herramientas para los tests.
- En `requirements-dev.txt`: `pytest` y `pytest-cov`.

Instálalas y fija las versiones.

<details markdown="1">
<summary>Ver solución</summary>

```
# requirements.txt
fastapi[standard]==...
```

```
# requirements-dev.txt
-r requirements.txt
ruff==...
pyyaml==...
openapi-spec-validator==...
pytest==...
pytest-cov==...
```

Las versiones que veas tú con `pip freeze` serán otras: usa las tuyas.

```bash
git switch -c feat/api-fastapi
pip install -r requirements-dev.txt
git add requirements.txt requirements-dev.txt
git commit -m "build: añade FastAPI y pytest a las dependencias"
```

`build` es el tipo para cambios en las dependencias o en cómo se construye el proyecto.

</details>

### 4.2. Los modelos

Crea `app/__init__.py` (vacío) y `app/main.py`. En `main.py`, crea la aplicación de FastAPI con el mismo título, descripción y versión que el contrato, y define con **Pydantic** los modelos `ContactoNuevo` y `Contacto`.

**Pistas:**

- Un modelo de Pydantic es una clase que hereda de `BaseModel`, con un atributo por campo y su tipo.
- Un campo opcional se escribe `email: str | None = None`.
- `Contacto` puede **heredar** de `ContactoNuevo` y añadir solo el `id`. Es lo mismo que hicimos con `allOf` en el YAML.
- Para que el nombre no pueda estar vacío, usa `Field(min_length=1)`.

<details markdown="1">
<summary>Ver solución</summary>

```python
# app/main.py
from itertools import count

from fastapi import FastAPI, HTTPException
from pydantic import BaseModel, Field

app = FastAPI(
    title="Agenda de contactos",
    description="API para gestionar una agenda de contactos.",
    version="1.0.0",
)


class ContactoNuevo(BaseModel):
    nombre: str = Field(min_length=1)
    telefono: str
    email: str | None = None


class Contacto(ContactoNuevo):
    id: int


# Los contactos se guardan en memoria: se pierden al parar el servidor.
contactos: dict[int, Contacto] = {}
contador = count(1)
```

`count(1)` es un contador que da 1, 2, 3… cada vez que llamas a `next(contador)`. Así cada contacto nuevo tiene un `id` distinto, aunque hayas borrado otros.

</details>

### 4.3. Los endpoints, con sus tests

Programa los cinco endpoints y sus tests en `tests/test_contactos.py`. Ve de uno en uno: programa un endpoint, escribe sus tests, comprueba que pasan con `pytest` y haz commit.

> 💡 **Buena práctica:** commits pequeños y frecuentes. Si un día algo se rompe, es mucho más fácil encontrar el culpable entre cinco commits pequeños que dentro de uno enorme.

**Pistas:**

- Para que FastAPI sepa qué devuelve cada endpoint, indica el tipo de retorno de la función: `-> Contacto` o `-> list[Contacto]`.
- El código de estado se cambia en el decorador: `@app.post("/contactos", status_code=201)`.
- Para responder con un 404: `raise HTTPException(status_code=404, detail="Contacto no encontrado")`. Fíjate en el `detail`: es el campo que pusimos en el esquema `Error`.
- Para los tests, `TestClient` hace peticiones a tu API sin necesidad de arrancar el servidor.
- Como los contactos se guardan en memoria, un test podría ver los contactos que creó el anterior. Usa una *fixture* de pytest con `autouse=True` que vacíe la agenda antes de cada test.

Comprueba en cada test el código de estado **y** el contenido de la respuesta. Y no te olvides de los casos de error: crear sin nombre, pedir un contacto que no existe…

<details markdown="1">
<summary>Ver solución: los endpoints</summary>

```python
# app/main.py (debajo de lo anterior)


def buscar_contacto(id: int) -> Contacto:
    if id not in contactos:
        raise HTTPException(status_code=404, detail="Contacto no encontrado")
    return contactos[id]


@app.get("/contactos")
def listar_contactos(nombre: str | None = None) -> list[Contacto]:
    resultado = list(contactos.values())
    if nombre:
        resultado = [c for c in resultado if nombre.lower() in c.nombre.lower()]
    return resultado


@app.post("/contactos", status_code=201)
def crear_contacto(datos: ContactoNuevo) -> Contacto:
    contacto = Contacto(id=next(contador), **datos.model_dump())
    contactos[contacto.id] = contacto
    return contacto


@app.get("/contactos/{id}")
def obtener_contacto(id: int) -> Contacto:
    return buscar_contacto(id)


@app.put("/contactos/{id}")
def actualizar_contacto(id: int, datos: ContactoNuevo) -> Contacto:
    buscar_contacto(id)
    contacto = Contacto(id=id, **datos.model_dump())
    contactos[id] = contacto
    return contacto


@app.delete("/contactos/{id}", status_code=204)
def borrar_contacto(id: int) -> None:
    buscar_contacto(id)
    del contactos[id]
```

`buscar_contacto` no es un endpoint (no tiene decorador): es una función auxiliar para no repetir la comprobación del 404 en tres sitios.

</details>

<details markdown="1">
<summary>Ver solución: los tests</summary>

```python
# tests/test_contactos.py
from itertools import count

import pytest
from fastapi.testclient import TestClient

from app import main

cliente = TestClient(main.app)

ANA = {"nombre": "Ana García", "telefono": "600123456", "email": "ana@ejemplo.com"}
LUIS = {"nombre": "Luis Pérez", "telefono": "611222333"}


@pytest.fixture(autouse=True)
def agenda_vacia():
    main.contactos.clear()
    main.contador = count(1)


def test_crear_contacto():
    respuesta = cliente.post("/contactos", json=ANA)
    assert respuesta.status_code == 201
    assert respuesta.json() == {"id": 1, **ANA}


def test_crear_contacto_sin_email():
    respuesta = cliente.post("/contactos", json=LUIS)
    assert respuesta.status_code == 201
    assert respuesta.json()["email"] is None


def test_crear_contacto_sin_telefono_da_422():
    respuesta = cliente.post("/contactos", json={"nombre": "Ana"})
    assert respuesta.status_code == 422


def test_crear_contacto_con_nombre_vacio_da_422():
    respuesta = cliente.post("/contactos", json={"nombre": "", "telefono": "600123456"})
    assert respuesta.status_code == 422


def test_listar_agenda_vacia():
    respuesta = cliente.get("/contactos")
    assert respuesta.status_code == 200
    assert respuesta.json() == []


def test_listar_filtrando_por_nombre():
    cliente.post("/contactos", json=ANA)
    cliente.post("/contactos", json=LUIS)
    respuesta = cliente.get("/contactos", params={"nombre": "ana"})
    assert [c["nombre"] for c in respuesta.json()] == ["Ana García"]


def test_obtener_contacto():
    cliente.post("/contactos", json=ANA)
    respuesta = cliente.get("/contactos/1")
    assert respuesta.status_code == 200
    assert respuesta.json()["nombre"] == "Ana García"


def test_obtener_contacto_que_no_existe_da_404():
    respuesta = cliente.get("/contactos/999")
    assert respuesta.status_code == 404
    assert respuesta.json() == {"detail": "Contacto no encontrado"}


def test_actualizar_contacto():
    cliente.post("/contactos", json=ANA)
    respuesta = cliente.put("/contactos/1", json=LUIS)
    assert respuesta.status_code == 200
    assert respuesta.json()["nombre"] == "Luis Pérez"
    assert cliente.get("/contactos/1").json()["nombre"] == "Luis Pérez"


def test_actualizar_contacto_que_no_existe_da_404():
    respuesta = cliente.put("/contactos/999", json=LUIS)
    assert respuesta.status_code == 404


def test_borrar_contacto():
    cliente.post("/contactos", json=ANA)
    respuesta = cliente.delete("/contactos/1")
    assert respuesta.status_code == 204
    assert cliente.get("/contactos/1").status_code == 404


def test_borrar_contacto_que_no_existe_da_404():
    respuesta = cliente.delete("/contactos/999")
    assert respuesta.status_code == 404


def test_los_ids_no_se_repiten_al_borrar():
    cliente.post("/contactos", json=ANA)
    cliente.delete("/contactos/1")
    respuesta = cliente.post("/contactos", json=LUIS)
    assert respuesta.json()["id"] == 2
```

Una posible secuencia de commits:

```bash
git commit -m "feat: añade los modelos de contacto"
git commit -m "feat: añade la creación y el listado de contactos"
git commit -m "feat: añade obtener, actualizar y borrar contactos"
```

Cada uno con sus tests incluidos, y con `pytest` y `ruff check .` pasando antes de hacerlo.

</details>

### 4.4. Pruébala a mano

Arranca la API:

```bash
fastapi dev app/main.py
```

Y abre [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs). Ahí está tu **Swagger UI**, generado solo a partir del código. Crea un par de contactos con **Try it out**, pide uno que no existe, intenta crear uno sin teléfono…

Fíjate en que ahora sí **hay memoria**: lo que creas, se queda (hasta que pares el servidor).

### 4.5. Los tests en la CI y el pull request

Añade a la CI un paso que ejecute los tests con cobertura (`pytest --cov=app`), haz push y abre el pull request. Cuando esté en verde, fusiónalo.

<details markdown="1">
<summary>Ver solución</summary>

Añade este paso al final de `.github/workflows/ci.yml`:

```yaml
      - run: pytest --cov=app
```

```bash
git add .github/workflows/ci.yml
git commit -m "ci: ejecuta los tests con cobertura"
git push -u origin feat/api-fastapi
```

¿Por qué no lo añadimos antes? Porque pytest falla si no encuentra ningún test. La CI crece a la vez que el proyecto.

</details>

## Fase 5: comparar los contratos y probar la API real

Tenemos dos contratos de la misma API: el que escribiste tú en la fase 1 y el que genera FastAPI. Vamos a compararlos.

### 5.1. Exporta el contrato generado

El contrato generado está en [http://127.0.0.1:8000/openapi.json](http://127.0.0.1:8000/openapi.json), pero es mejor tenerlo en el repositorio. Crea la rama `docs/contrato-generado` y un script `scripts/exportar_openapi.py` que lo guarde en `contrato/openapi-generado.json`.

**Pista:** `app.openapi()` devuelve el contrato como un diccionario, sin necesidad de arrancar el servidor.

<details markdown="1">
<summary>Ver solución</summary>

```python
# scripts/exportar_openapi.py
"""Guarda en contrato/openapi-generado.json el contrato que genera FastAPI."""

import json
from pathlib import Path

from app.main import app

DESTINO = Path("contrato/openapi-generado.json")

DESTINO.write_text(
    json.dumps(app.openapi(), indent=2, ensure_ascii=False) + "\n", encoding="utf-8"
)
print(f"Generado {DESTINO}")
```

Se ejecuta con `-m` desde la raíz del proyecto, para que Python encuentre el paquete `app`:

```bash
python -m scripts.exportar_openapi
```

</details>

### 5.2. Busca las diferencias

Abre `contrato/openapi.json` (el tuyo) y `contrato/openapi-generado.json` (el de FastAPI) uno al lado del otro. En VS Code puedes seleccionar los dos, hacer clic derecho y elegir **Compare Selected**.

Busca al menos cinco diferencias. Para cada una, piensa: ¿cuál de los dos está mejor? ¿Se puede ajustar el código para que el generado se parezca al tuyo?

<details markdown="1">
<summary>Ver solución</summary>

| | Escrito a mano | Generado por FastAPI |
|---|---|---|
| **`servers`** | Mock y API real | No aparece |
| **`example`** | En todos los campos | Ninguno |
| **`email`** | `format: email` | Un texto cualquiera (o `null`) |
| **404** | Documentado, con el esquema `Error` | **No aparece**: FastAPI no sabe que `buscar_contacto` puede lanzarlo |
| **422** | Solo la descripción | Documentado al detalle (`HTTPValidationError`), y en todos los endpoints con parámetros |
| **`operationId`** | `crearContacto` | `crear_contacto_contactos_post` |
| **`Contacto`** | `allOf` con `ContactoNuevo` | Todos los campos repetidos |

Ninguno es "el bueno". El generado es **más exacto** en lo que FastAPI sabe (las validaciones y el 422), y el tuyo es **más completo** en lo que solo sabe una persona (los ejemplos, el 404, los servidores).

La lección más importante es la del 404: **un contrato generado solo describe lo que el framework puede ver**. Si no le cuentas que un endpoint puede devolver un 404, no lo documenta.

</details>

### 5.3. Mejora el código

Ajusta el código para que el contrato generado se acerque al tuyo:

1. **Documenta el 404** en los tres endpoints que lo pueden devolver, con un modelo `Error`. Pista: el parámetro `responses` del decorador.
2. **Añade ejemplos** a los campos. Pista: `Field(examples=[...])`.
3. **Valida el email** de verdad. Pista: Pydantic tiene un tipo `EmailStr`. Escribe un test que compruebe que un email incorrecto da un 422.

Vuelve a exportar el contrato y compáralo otra vez.

<details markdown="1">
<summary>Ver solución</summary>

```python
from pydantic import BaseModel, EmailStr, Field


class ContactoNuevo(BaseModel):
    nombre: str = Field(min_length=1, examples=["Ana García"])
    telefono: str = Field(examples=["600123456"])
    email: EmailStr | None = Field(default=None, examples=["ana@ejemplo.com"])


class Contacto(ContactoNuevo):
    id: int = Field(examples=[1])


class Error(BaseModel):
    detail: str = Field(examples=["Contacto no encontrado"])


NO_ENCONTRADO = {404: {"model": Error, "description": "No hay ningún contacto con ese id"}}


@app.get("/contactos/{id}", responses=NO_ENCONTRADO)
def obtener_contacto(id: int) -> Contacto: ...


@app.put("/contactos/{id}", responses=NO_ENCONTRADO)
def actualizar_contacto(id: int, datos: ContactoNuevo) -> Contacto: ...


@app.delete("/contactos/{id}", status_code=204, responses=NO_ENCONTRADO)
def borrar_contacto(id: int) -> None: ...
```

El cuerpo de las funciones no cambia; solo cambian los decoradores. `EmailStr` necesita la librería `email-validator`, que ya viene incluida en `fastapi[standard]`.

```python
# tests/test_contactos.py
def test_crear_contacto_con_email_incorrecto_da_422():
    datos = {"nombre": "Ana", "telefono": "600123456", "email": "esto-no-es-un-email"}
    respuesta = cliente.post("/contactos", json=datos)
    assert respuesta.status_code == 422
```

```bash
python -m scripts.exportar_openapi
git add app/ tests/ scripts/exportar_openapi.py contrato/openapi-generado.json
git commit -m "docs: documenta el 404, los ejemplos y valida el email"
```

Abre otra vez `/docs`: ahora el 404 aparece en los tres endpoints y los ejemplos están rellenos.

</details>

### 5.4. La prueba final: Postman contra la API real

Este es el momento clave del ejercicio. Arranca la API con `fastapi dev app/main.py`, abre Postman, cambia el entorno de **Mock** a **Local** y lanza la colección entera con el Collection Runner.

**Es la misma colección que creaste en la fase 3**, a partir del contrato, antes de que existiera ni una línea de código. Si todo sale en verde, tu API **cumple el contrato**.

¿Y el test de **Obtener un contacto borrado**, el que fallaba contra el mock? Ahora debería pasar, porque la API de verdad sí recuerda que lo has borrado.

> Si algún test falla, no lo arregles cambiando el test: mira si el que no cumple el contrato es el código. Para eso está el contrato.

### 5.5. Publica la v1.0.0

Haz push de la rama, abre el pull request y fusiónalo. Después crea la etiqueta **`v1.0.0`**: la API existe, está probada y cumple el contrato.

<details markdown="1">
<summary>Ver solución</summary>

```bash
git push -u origin docs/contrato-generado
# ... pull request, CI en verde, merge ...
git switch main
git pull
git branch -d docs/contrato-generado
git log v0.1.0..HEAD --oneline       # todo lo que ha pasado desde el contrato
git tag -a v1.0.0 -m "v1.0.0: API de la agenda de contactos"
git push origin v1.0.0
```

</details>

## Design-first vs code-first: qué has aprendido

Has construido la misma API de las dos maneras. Esto es lo que has podido ver:

| | Design-first | Code-first |
|---|---|---|
| **Qué escribes primero** | El contrato (YAML) | El código (FastAPI) |
| **Cuándo puedes probar la API** | Desde el minuto uno, con el mock | Cuando el código funciona |
| **Trabajo en equipo** | Frontend y backend empiezan a la vez | El frontend espera al backend |
| **Documentación** | Tan completa como la escribas | Siempre al día, pero solo con lo que el framework ve |
| **Riesgo** | Que el código acabe sin cumplir el contrato | Que el contrato tenga huecos (como el 404) |
| **Esfuerzo** | Más al principio | Menos al principio |

En la práctica, muchos equipos **combinan** las dos: diseñan el contrato para acordarlo con el frontend y, después, comprueban que el código lo cumple. Que es justo lo que has hecho en la fase 5.

## Comprueba tu trabajo

En tu repositorio deberías ver:

- [ ] El contrato en `contrato/`, en YAML y en JSON, y el generado por FastAPI.
- [ ] La colección y los dos entornos de Postman en `postman/`.
- [ ] La pestaña **Pull requests → Closed** con un pull request por fase.
- [ ] Un historial en `main` (`git log --oneline`) con tipos variados: `chore`, `feat`, `ci`, `docs`, `test`, `build`.
- [ ] Dos etiquetas: **`v0.1.0`** (el contrato) y **`v1.0.0`** (la API).
- [ ] La CI en verde, validando el contrato, pasando el linter y ejecutando los tests.
- [ ] La colección de Postman en verde, tanto contra el mock (salvo el test del contacto borrado) como contra la API real.

## Retos extra

Cada uno en su rama y con su pull request:

1. **Que la CI avise si el JSON está desactualizado:** añade un paso que ejecute `python scripts/yaml_a_json.py` y después `git diff --exit-code contrato/openapi.json`. Si alguien cambia el YAML y se olvida de regenerar el JSON, la CI saldrá en rojo.
2. **Postman en la CI:** [Newman](https://learning.postman.com/docs/collections/using-newman-cli/command-line-integration-with-newman/) ejecuta colecciones de Postman desde la terminal. Añade a la CI un paso que arranque el mock y lance la colección contra él con `npx newman run`.
3. **Comprobar automáticamente que la API cumple el contrato:** [Schemathesis](https://schemathesis.readthedocs.io) lee tu `openapi.yaml` y lanza cientos de peticiones contra la API buscando respuestas que no lo cumplan. Pruébalo contra tu API: ¿encuentra algo?
4. **Un mock en la nube:** Postman también puede crear servidores mock alojados en internet, sin necesidad de Node.js. Busca *Mock servers* en su documentación y crea uno a partir de tu colección. Ojo: el plan gratuito tiene un límite de peticiones al mes.
5. **Autenticación con JWT:** protege los endpoints con un token [JWT]({% post_url 2026-11-09-jwt %}). Primero en el contrato (`securitySchemes`, como en el post de OpenAPI) y después en FastAPI. En Postman, guarda el token en una variable del entorno… y ese entorno ya **no** se sube al repositorio.
6. **Una versión que rompe algo:** cambia `telefono` por una lista `telefonos`, para que un contacto pueda tener varios. Quien use la API tendrá que cambiar su código, así que es un cambio incompatible. ¿Qué cambias primero, el contrato o el código? ¿Cómo sería el commit? ¿Y la versión?
7. **Repite el ejercicio con una tienda:** productos, un carrito y pedidos. Hay varios recursos relacionados entre sí, así que el contrato será más interesante… y más largo.

## Para terminar

Has construido la misma API de dos maneras. Con **design-first**, escribiste el contrato a mano en YAML y JSON, lo pusiste a funcionar con un servidor mock y lo probaste entero con Postman antes de programar nada. Con **code-first**, programaste la API con FastAPI y él generó el contrato. Y al final comprobaste lo más importante: que la API cumple el contrato.

Por el camino has trabajado como en un equipo real: una rama por fase, commits con significado, `main` protegida, una CI que crece con el proyecto y versiones que cuentan la historia del proyecto.

Si lo completas, ¡me encantaría ver tu repositorio! Y si algo no se entiende o ves algún error, dímelo.
