---
layout: post
title: "Ejercicio: una calculadora en Python con Git, tests y CI/CD"
date: 2026-11-10 03:00:00 +0000
category: "Ejercicios prácticos"
tags: [ejercicio, python, git, github, ci-cd, github-actions, buenas-practicas]
published: true
---

Leer sobre buenas prácticas está bien, pero se aprenden de verdad aplicándolas. En este ejercicio vas a crear un proyecto completo desde cero, como se haría en un equipo de desarrollo: una **calculadora en Python** que empieza siendo básica y acaba siendo científica.

Lo importante no es la calculadora (el código es sencillo a propósito), sino **todo lo que la rodea**:

- Un repositorio bien organizado, con su `.gitignore`.
- Un entorno virtual y las dependencias separadas en `requirements.txt` y `requirements-dev.txt`.
- Una rama para cada funcionalidad, con mensajes de commit que sigan Conventional Commits.
- Tests automáticos y un linter.
- La rama `main` protegida: nada entra sin pasar por un pull request con la CI en verde.
- Un pipeline de **CI** que comprueba cada cambio y uno de **CD** que publica cada versión nueva en GitHub.

Al final tendrás un proyecto que puedes enseñar en tu portfolio.

## Antes de empezar

**Necesitas:**

- [Git](https://git-scm.com) instalado y configurado con tu nombre y tu email.
- Una cuenta de [GitHub](https://github.com).
- Python 3.12 o superior. Compruébalo con `python3 --version`.

**Conviene haber leído antes:**

- [Git para principiantes: los comandos básicos]({% post_url 2026-11-05-git-comandos-basicos %})
- [Conventional Commits]({% post_url 2026-11-05-git-conventional-commits %})
- [CI/CD: que tu código se pruebe y se publique solo]({% post_url 2026-11-10-ci-cd %})

**Cómo funciona el ejercicio:** cada fase tiene un objetivo y unas pistas. Intenta hacerlo por tu cuenta y, si te atascas, despliega la **solución**. No hay una única forma correcta: si tu código es distinto pero los tests pasan, está bien.

Este es el proyecto que vas a construir:

```
calculadora/
├── .github/
│   └── workflows/
│       ├── ci.yml              # CI: tests y linter en cada cambio
│       └── release.yml         # CD: publica cada versión nueva
├── calculadora/
│   ├── __init__.py
│   ├── basica.py               # fase 2
│   └── cientifica.py           # fase 3
├── tests/
│   ├── test_basica.py
│   └── test_cientifica.py
├── .gitignore
├── pyproject.toml
├── README.md
├── requirements.txt
└── requirements-dev.txt
```

## Fase 0: preparar el proyecto

### 0.1. Crea la carpeta y el entorno virtual

Un **entorno virtual** es una instalación de Python aislada solo para este proyecto. Las librerías que instales en él no se mezclan con las de otros proyectos ni con las del sistema.

```bash
mkdir calculadora
cd calculadora
git init -b main

python3 -m venv .venv
```

Y actívalo:

```bash
source .venv/bin/activate      # Linux y macOS
.venv\Scripts\activate         # Windows
```

Sabrás que está activo porque verás `(.venv)` al principio de la línea de la terminal. **Actívalo siempre** que vayas a trabajar en el proyecto.

### 0.2. Separa las dependencias

Hay dos tipos de dependencias:

- **`requirements.txt`**: lo que la aplicación necesita **para funcionar**.
- **`requirements-dev.txt`**: lo que tú necesitas **para desarrollarla**: tests, linter… Quien solo quiera usar la calculadora no lo necesita.

Nuestra calculadora solo usa la librería estándar de Python (el módulo `math`), así que `requirements.txt` empieza sin dependencias. Lo creamos igualmente: es donde irán si algún día hacen falta, y es lo primero que buscará cualquiera que llegue al proyecto.

```
# requirements.txt
# Dependencias de la aplicación. De momento solo usa la librería estándar.
```

Para desarrollar usaremos tres herramientas:

- **pytest**, para los tests.
- **pytest-cov**, para saber qué porcentaje del código cubren los tests.
- **ruff**, un linter que revisa el estilo y detecta errores comunes.

```
# requirements-dev.txt
-r requirements.txt
pytest
pytest-cov
ruff
```

La primera línea, `-r requirements.txt`, significa "instala también todo lo de `requirements.txt`". Así, con un solo comando tienes todo lo necesario:

```bash
pip install -r requirements-dev.txt
```

**Ahora fija las versiones.** Para que tu ordenador y la CI usen exactamente lo mismo, mira qué versiones se han instalado con `pip freeze` y escríbelas en `requirements-dev.txt`. Quedará algo así (tus números serán otros):

```
-r requirements.txt
pytest==8.3.5
pytest-cov==6.1.1
ruff==0.11.8
```

### 0.3. Configura pytest y ruff

Crea un archivo `pyproject.toml` con la configuración de las herramientas:

```toml
[tool.pytest.ini_options]
pythonpath = ["."]
testpaths = ["tests"]

[tool.ruff]
line-length = 100
```

`pythonpath = ["."]` permite que los tests encuentren el paquete `calculadora`. `testpaths` le dice a pytest dónde están los tests.

### 0.4. El .gitignore

El `.gitignore` le dice a Git qué archivos **no** debe guardar. Piensa qué carpetas y archivos se generan solos en este proyecto y no deberían subirse nunca.

<details markdown="1">
<summary>Ver solución</summary>

```gitignore
# Entorno virtual
.venv/

# Archivos que genera Python
__pycache__/
*.pyc

# Cachés y resultados de las herramientas
.pytest_cache/
.ruff_cache/
.coverage
htmlcov/

# Configuración de editores y del sistema
.vscode/
.idea/
.DS_Store
```

El más importante es `.venv/`: el entorno virtual pesa mucho, depende de tu sistema operativo y cualquiera puede volver a crearlo con `requirements-dev.txt`. Si buscas "gitignore Python" encontrarás plantillas mucho más completas, como la que ofrece GitHub al crear un repositorio.

</details>

### 0.5. El paquete y el README

Crea la carpeta del código con un `__init__.py` vacío (le dice a Python que esa carpeta es un paquete) y la de los tests:

```bash
mkdir calculadora tests
touch calculadora/__init__.py
```

Y escribe un `README.md` corto: qué es el proyecto, cómo crear el entorno virtual, cómo instalar las dependencias y cómo ejecutar los tests.

### 0.6. Primer commit y repositorio en GitHub

Haz el primer commit directamente en `main`. Será el único que no pase por un pull request, porque todavía no hay nada que proteger.

1. Comprueba con `git status` que **no** aparece `.venv/`. Si aparece, revisa tu `.gitignore`.
2. Crea el commit con un mensaje de Conventional Commits. ¿Qué tipo usarías para "preparar la estructura del proyecto"?
3. Crea un repositorio **público** en GitHub llamado `calculadora`, **vacío** (sin README ni `.gitignore`, que ya los tienes), y sube tu código.

<details markdown="1">
<summary>Ver solución</summary>

```bash
git add .
git commit -m "chore: prepara la estructura del proyecto"
git remote add origin https://github.com/TU_USUARIO/calculadora.git
git push -u origin main
```

`chore` es el tipo para tareas de mantenimiento que no añaden funcionalidad ni arreglan nada.

El repositorio tiene que ser público para que las reglas de protección de ramas de la fase 1 sean gratis. En los repositorios privados dependen de tu plan de GitHub.

</details>

## Fase 1: CI y protección de main

Montamos la CI **antes** de escribir la calculadora. Así, desde la primera funcionalidad, cada cambio se comprobará automáticamente.

### 1.1. El pipeline de CI

A partir de aquí **no vuelvas a trabajar directamente en `main`**. Crea una rama para la CI y, en ella, el archivo `.github/workflows/ci.yml`. Tiene que ejecutarse en cada push a `main` y en cada pull request, y hacer esto:

1. Descargar el código.
2. Instalar Python 3.13.
3. Instalar las dependencias de desarrollo.
4. Pasar el linter: `ruff check .`
5. Pasar los tests con cobertura: `pytest --cov=calculadora`

Si no te acuerdas de cómo es un workflow, repasa el [post de CI/CD]({% post_url 2026-11-10-ci-cd %}#ci-en-la-práctica-github-actions).

<details markdown="1">
<summary>Ver solución</summary>

```bash
git switch -c ci/github-actions
mkdir -p .github/workflows
```

```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [main]
  pull_request:

jobs:
  tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v5
      - uses: actions/setup-python@v6
        with:
          python-version: "3.13"
          cache: pip
      - run: pip install -r requirements-dev.txt
      - run: ruff check .
      - run: pytest --cov=calculadora
```

`cache: pip` guarda las librerías descargadas entre ejecuciones, para que la CI tarde menos.

```bash
git add .github/workflows/ci.yml
git commit -m "ci: añade tests y linter con GitHub Actions"
git push -u origin ci/github-actions
```

</details>

### 1.2. Tu primer pull request

En GitHub, abre un **pull request** de `ci/github-actions` hacia `main`. Verás que la CI se ejecuta sola dentro del propio pull request.

Esta vez **fallará**, y es normal: todavía no hay ningún test, y pytest lo trata como un error. Para arreglarlo, añade en la misma rama un test de prueba que compruebe que el paquete se puede importar.

<details markdown="1">
<summary>Ver solución</summary>

```python
# tests/test_paquete.py
import calculadora


def test_el_paquete_se_puede_importar():
    assert calculadora is not None
```

```bash
git add tests/test_paquete.py
git commit -m "test: comprueba que el paquete se puede importar"
git push
```

Al hacer push a la misma rama, el pull request se actualiza y la CI se vuelve a ejecutar. Ahora debería salir en verde. Fusiona el pull request con **Merge** y borra la rama.

En tu ordenador, vuelve a `main` y tráete los cambios:

```bash
git switch main
git pull
git branch -d ci/github-actions
```

</details>

### 1.3. Protege main

Ahora que la CI existe, vamos a hacer que sea **obligatoria**. En tu repositorio, ve a *Settings → Rules → Rulesets* y crea una regla para la rama principal que:

- Obligue a pasar por un **pull request** para cambiar `main`.
- Exija que la CI (el job `tests`) esté en **verde** antes de fusionar.
- Impida los *force push* y que se borre la rama.

<details markdown="1">
<summary>Ver solución</summary>

1. *Settings → Rules → Rulesets → New ruleset → New branch ruleset*.
2. **Ruleset name**: `proteger-main`. **Enforcement status**: `Active`.
3. En **Target branches**, pulsa *Add target → Include default branch*.
4. Marca estas reglas:
   - **Restrict deletions**: nadie puede borrar `main`.
   - **Block force pushes**: nadie puede reescribir su historial.
   - **Require a pull request before merging**. Deja las aprobaciones (*Required approvals*) a `0`: si trabajas sola, GitHub no te deja aprobar tus propios pull requests. En un equipo pondrías al menos `1`.
   - **Require status checks to pass**: pulsa *Add checks* y elige `tests`. Solo aparece si la CI se ha ejecutado al menos una vez, por eso la montamos antes.
5. Guarda con **Create**.

Pruébalo: intenta hacer un push directamente a `main`. GitHub lo rechazará.

Los nombres de los menús de GitHub cambian de vez en cuando. Si no encuentras alguno, busca "rulesets" en la documentación de GitHub.

</details>

## Fase 2: la calculadora básica

Crea el módulo `calculadora/basica.py` con estas funciones, cada una con sus tests en `tests/test_basica.py`:

| Función | Qué hace |
|---|---|
| `sumar(a, b)` | Devuelve `a + b` |
| `restar(a, b)` | Devuelve `a - b` |
| `multiplicar(a, b)` | Devuelve `a * b` |
| `dividir(a, b)` | Devuelve `a / b`. Si `b` es `0`, lanza un `ValueError` con un mensaje claro |

**Las reglas del juego**, que seguirás también en la fase 3:

- **Una rama por funcionalidad**, con nombres como `feat/suma-resta` y `feat/multiplicacion-division`.
- **Cada funcionalidad con sus tests**, en la misma rama.
- **Commits con Conventional Commits**, usando `basica` como ámbito: `feat(basica): ...`, `test(basica): ...`.
- **Un pull request por rama**, que solo fusionas cuando la CI esté en verde.
- Antes de hacer push, pasa los tests y el linter en tu ordenador: `ruff check .` y `pytest`. Mejor enterarte tú que la CI.

Pista para los tests: para comprobar que una función lanza un error, pytest tiene `pytest.raises`.

<details markdown="1">
<summary>Ver solución: el flujo de trabajo con Git</summary>

El ciclo es el mismo para cada funcionalidad:

```bash
git switch main
git pull                                # parte siempre de main actualizado
git switch -c feat/suma-resta

# ... escribes el código y los tests ...
ruff check .
pytest

git add calculadora/basica.py tests/test_basica.py
git commit -m "feat(basica): añade suma y resta"
git push -u origin feat/suma-resta
```

Después: abre el pull request, espera a la CI en verde, fusiona, borra la rama y vuelve a empezar con `feat/multiplicacion-division`.

Si prefieres separar el código y los tests en dos commits, también vale: `feat(basica): añade suma y resta` y `test(basica): añade tests de suma y resta`.

</details>

<details markdown="1">
<summary>Ver solución: el código</summary>

```python
# calculadora/basica.py
"""Operaciones básicas de la calculadora."""


def sumar(a: float, b: float) -> float:
    return a + b


def restar(a: float, b: float) -> float:
    return a - b


def multiplicar(a: float, b: float) -> float:
    return a * b


def dividir(a: float, b: float) -> float:
    if b == 0:
        raise ValueError("No se puede dividir entre cero")
    return a / b
```

```python
# tests/test_basica.py
import pytest

from calculadora.basica import dividir, multiplicar, restar, sumar


def test_sumar():
    assert sumar(2, 3) == 5


def test_sumar_negativos():
    assert sumar(-2, -3) == -5


def test_restar():
    assert restar(5, 3) == 2


def test_multiplicar():
    assert multiplicar(4, 3) == 12


def test_multiplicar_por_cero():
    assert multiplicar(7, 0) == 0


def test_dividir():
    assert dividir(10, 4) == 2.5


def test_dividir_entre_cero_lanza_error():
    with pytest.raises(ValueError):
        dividir(1, 0)
```

Fíjate en que no solo se prueba el caso normal: también los negativos, el cero y el error. Los fallos suelen esconderse en los casos límite.

</details>

### Tu primera versión: v1.0.0

Cuando la calculadora básica esté en `main`, ya tienes algo que publicar. Pero para eso necesitamos la CD.

## Fase 3: CD, publicar versiones automáticamente

Vamos a hacer que, cada vez que subas un **tag** de versión (como `v1.0.0`), GitHub:

1. Vuelva a pasar los tests.
2. Si pasan, cree una **Release**: una página con la versión, la lista de cambios y el código para descargar.

Esto es **entrega continua**: tú decides cuándo sale una versión creando el tag, y el pipeline hace todo lo demás.

En una rama `ci/release`, crea el archivo `.github/workflows/release.yml`. Pistas:

- Se ejecuta con `on: push: tags: ["v*"]`.
- Necesita permiso para crear releases: `permissions: contents: write`.
- Para crear la release puedes usar el comando `gh`, que ya viene instalado en las máquinas de GitHub: `gh release create <tag> --generate-notes`. La opción `--generate-notes` escribe la lista de cambios a partir de los pull requests.

<details markdown="1">
<summary>Ver solución</summary>

{% raw %}
```yaml
# .github/workflows/release.yml
name: Release

on:
  push:
    tags: ["v*"]

permissions:
  contents: write

jobs:
  tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v5
      - uses: actions/setup-python@v6
        with:
          python-version: "3.13"
          cache: pip
      - run: pip install -r requirements-dev.txt
      - run: ruff check .
      - run: pytest --cov=calculadora

  publicar:
    needs: tests
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v5
      - run: gh release create "$GITHUB_REF_NAME" --generate-notes
        env:
          GH_TOKEN: ${{ github.token }}
```
{% endraw %}

- `needs: tests`: si los tests fallan, no se publica.
- `$GITHUB_REF_NAME` es una variable que GitHub rellena con el nombre del tag, por ejemplo `v1.0.0`.
- `github.token` es un token que GitHub crea solo para cada ejecución. No tienes que guardar ningún secreto.

```bash
git switch main && git pull
git switch -c ci/release
# ... creas el archivo ...
git add .github/workflows/release.yml
git commit -m "ci: publica una release con cada tag de versión"
git push -u origin ci/release
```

Pull request, CI en verde, fusionar.

</details>

### Publica la v1.0.0

Crea un **tag anotado** `v1.0.0` sobre `main` y súbelo. Después, ve a la pestaña **Actions** para ver el pipeline y, cuando termine, a **Releases** (en la columna derecha de tu repositorio).

<details markdown="1">
<summary>Ver solución</summary>

```bash
git switch main
git pull
git tag -a v1.0.0 -m "v1.0.0: calculadora básica"
git push origin v1.0.0
```

Recuerda que `git push` normal no sube los tags: hay que subirlos por su nombre.

</details>

## Fase 4: la calculadora científica

Ahora amplía la calculadora con un módulo nuevo, `calculadora/cientifica.py`:

| Función | Qué hace |
|---|---|
| `potencia(base, exponente)` | Devuelve `base` elevado a `exponente` |
| `raiz_cuadrada(x)` | Devuelve la raíz cuadrada. Si `x` es negativo, lanza un `ValueError` |
| `logaritmo(x, base=10)` | Devuelve el logaritmo de `x`. Si `x` es `0` o negativo, lanza un `ValueError` |
| `seno(grados)`, `coseno(grados)`, `tangente(grados)` | Trigonometría, con el ángulo **en grados** |

Las mismas reglas que en la fase 2, con `cientifica` como ámbito de los commits. Por ejemplo, con estas ramas:

- `feat/potencia-raiz`
- `feat/logaritmo`
- `feat/trigonometria`

**Pistas:**

- El módulo `math` de Python tiene casi todo lo que necesitas. Pero ojo: sus funciones trigonométricas trabajan en **radianes**, no en grados. Busca cómo convertirlos.
- Los números decimales en informática no son exactos. `seno(30)` no da exactamente `0.5`, sino algo como `0.49999999999999994`. Para compararlos en los tests, usa `pytest.approx`.

<details markdown="1">
<summary>Ver solución: el código</summary>

```python
# calculadora/cientifica.py
"""Operaciones científicas de la calculadora."""

import math


def potencia(base: float, exponente: float) -> float:
    return base**exponente


def raiz_cuadrada(x: float) -> float:
    if x < 0:
        raise ValueError("No existe la raíz cuadrada real de un número negativo")
    return math.sqrt(x)


def logaritmo(x: float, base: float = 10) -> float:
    if x <= 0:
        raise ValueError("El logaritmo solo existe para números positivos")
    return math.log(x, base)


def seno(grados: float) -> float:
    return math.sin(math.radians(grados))


def coseno(grados: float) -> float:
    return math.cos(math.radians(grados))


def tangente(grados: float) -> float:
    return math.tan(math.radians(grados))
```

```python
# tests/test_cientifica.py
import pytest

from calculadora.cientifica import (
    coseno,
    logaritmo,
    potencia,
    raiz_cuadrada,
    seno,
    tangente,
)


def test_potencia():
    assert potencia(2, 10) == 1024


def test_potencia_exponente_cero():
    assert potencia(5, 0) == 1


def test_raiz_cuadrada():
    assert raiz_cuadrada(16) == 4


def test_raiz_cuadrada_de_negativo_lanza_error():
    with pytest.raises(ValueError):
        raiz_cuadrada(-1)


def test_logaritmo_base_10():
    assert logaritmo(1000) == pytest.approx(3)


def test_logaritmo_otra_base():
    assert logaritmo(8, base=2) == pytest.approx(3)


def test_logaritmo_de_cero_lanza_error():
    with pytest.raises(ValueError):
        logaritmo(0)


def test_seno():
    assert seno(30) == pytest.approx(0.5)


def test_coseno():
    assert coseno(60) == pytest.approx(0.5)


def test_tangente():
    assert tangente(45) == pytest.approx(1)
```

</details>

### Publica la versión nueva

Cuando las tres ramas estén fusionadas, toca publicar. ¿Qué número de versión le corresponde? Repasa la tabla de [versionado semántico]({% post_url 2026-11-05-git-conventional-commits %}#esto-conecta-con-las-versiones).

<details markdown="1">
<summary>Ver solución</summary>

Desde la v1.0.0 solo hemos añadido funcionalidades (`feat`) sin romper nada de lo que ya existía, así que sube el número del medio: **v1.1.0**.

```bash
git switch main
git pull
git log v1.0.0..HEAD --oneline       # los cambios desde la última versión
git tag -a v1.1.0 -m "v1.1.0: calculadora científica"
git push origin v1.1.0
```

Mira la release nueva: la lista de cambios incluye los pull requests de las tres ramas. Por eso conviene que sus títulos también sigan Conventional Commits.

</details>

## Comprueba tu trabajo

Si has llegado hasta aquí, en tu repositorio deberías ver:

- [ ] La pestaña **Actions** con una ejecución por cada pull request y por cada tag.
- [ ] La pestaña **Pull requests → Closed** con un pull request por cada funcionalidad.
- [ ] Un historial limpio en `main` (`git log --oneline`), con mensajes que se entienden sin abrir el código.
- [ ] Dos releases, **v1.0.0** y **v1.1.0**, cada una con su lista de cambios.
- [ ] Ni rastro de `.venv/` ni de `__pycache__/` en el repositorio.

Y una última prueba, la más importante: crea una rama, **rompe un test a propósito** y abre un pull request. La CI saldrá en rojo y GitHub no te dejará fusionarlo. Para eso has montado todo esto.

## Retos extra

¿Te ha sabido a poco? Prueba a añadir alguna de estas mejoras, cada una en su rama y con su pull request:

1. **Una insignia en el README** que muestre si la CI está en verde. Añade esta línea, con tu usuario:
   ```markdown
   ![CI](https://github.com/TU_USUARIO/calculadora/actions/workflows/ci.yml/badge.svg)
   ```
2. **Cobertura mínima:** cambia el comando de la CI por `pytest --cov=calculadora --cov-fail-under=90` para que falle si los tests cubren menos del 90 % del código.
3. **Varias versiones de Python:** usa una *matrix* en la CI para pasar los tests con Python 3.12 y 3.13 a la vez. Busca `strategy: matrix` en la documentación de GitHub Actions.
4. **Un menú interactivo:** crea un `main.py` que pregunte al usuario qué operación quiere hacer y con qué números. Pista: usa `input()` y maneja los `ValueError` para que el programa no se cierre si alguien divide entre cero.
5. **Revisar los mensajes de commit en la CI:** la herramienta [commitlint](https://commitlint.js.org) comprueba que los commits siguen Conventional Commits, y tiene una *action* lista para usar en GitHub. Añádela a la CI de los pull requests para que salga en rojo si algún mensaje no sigue el formato.
6. **Una versión que rompe algo:** cambia `logaritmo` para que su base por defecto sea `e` en vez de `10`. Quien la use sin indicar la base obtendrá otro resultado, así que es un cambio incompatible. ¿Cómo sería el commit? ¿Y el número de la siguiente versión?

## Para terminar

Has hecho, en pequeño, lo mismo que se hace en un equipo de desarrollo real: un entorno reproducible, dependencias separadas, ramas por funcionalidad, commits con significado, tests, una rama `main` protegida y un pipeline que comprueba cada cambio y publica cada versión.

La calculadora es lo de menos. Ese flujo de trabajo es el mismo para cualquier proyecto, sea del tamaño que sea.

Si lo completas, ¡me encantaría ver tu repositorio! Y si algo no se entiende o ves algún error, dímelo.
