---
layout: post
title: "Git para principiantes: los comandos básicos"
date: 2026-11-05
tags: [git, principiantes, terminal]
---

> ⏱️ **En 10 segundos**
> - Git sirve para **dos cosas**: **no perder tu trabajo** y **trabajar en equipo** sin pisaros.
> - Primero, **instálalo**. Luego, el ciclo de siempre: **cambias → `add` → `commit` → `push`**.
> - Para colaborar: **`pull`** (bajar cambios), **ramas** (`branch`, `switch`) y **`merge`** (juntar).

En la entrada anterior vimos que **Git** y **GitHub** no son lo mismo. Hoy toca **usar Git** de verdad. 💪

---

## 🎮 La idea clave: una partida compartida

Imagina que tu proyecto es una partida de videojuego:

- **Puntos de guardado:** guardas la partida (**commit**) y, si algo sale mal, **vuelves atrás**.
- **Modo multijugador:** tus amigas juegan **la misma partida** desde su casa. Cada una sube sus avances y baja los de las demás.

Git hace **las dos cosas**. No es solo una copia de seguridad: es la forma de **trabajar con otras personas** en el mismo proyecto. 

---

## 📥 Paso 0: instala Git

Antes de nada, necesitas **tener Git en tu ordenador**.

- Sigue este tutorial oficial (en español): 👉 [Instalación de Git](https://git-scm.com/book/es/v2/Inicio---Sobre-el-Control-de-Versiones-Instalaci%C3%B3n-de-Git)
- Comprueba que funciona abriendo la terminal y escribiendo:

```bash
git --version   # si ves un número de versión, ¡ya está! 🎉
```

---

## ⚙️ Paso 1: preséntate (solo la primera vez)

Git necesita saber **quién eres** para firmar tus commits. Así, en un equipo, se sabe **quién cambió qué**:

```bash
git config --global user.name "Tu Nombre"
git config --global user.email "tu@email.com"
```

- Se hace **una sola vez** por ordenador.
- Usa el **mismo email** que en GitHub.

---

# 🧍 Parte 1: trabajar tú sola

## 📁 `git init`: empieza a vigilar una carpeta

```bash
git init
```

- Convierte una carpeta normal en un **repositorio** (un proyecto con Git).
- Crea una carpeta oculta `.git` donde se guarda **todo el historial**.
- ⚠️ No borres `.git`, ¡o pierdes el historial!

## 👀 `git status`: ¿qué ha cambiado?

```bash
git status
```

- 🟥 Rojo = cambiado, pero **no** elegido.
- 🟩 Verde = elegido para el próximo commit.
- **Truco:** úsalo **siempre** que te pierdas. No rompe nada.

## 🔍 `git diff`: ¿qué he cambiado exactamente?

```bash
git diff
```

- Te enseña **línea a línea** lo que has añadido (➕) o quitado (➖).
- Para salir, pulsa **`q`**.

## 🧺 `git add`: elige qué guardar

```bash
git add index.html   # añade un archivo concreto
git add .            # añade todos los cambios
```

- Es como **meter cosas en la cesta** antes de pagar.
- Todavía **no se ha guardado** nada. Solo has elegido.

## 📸 `git commit`: guarda la partida

```bash
git commit -m "Añado la página de inicio"
```

- Guarda una **"foto"** de lo que había en la cesta.
- El **mensaje** explica **qué cambiaste**. Tus compañeras lo leerán, ¡que se entienda!
- "Arreglo el menú" ✅ — "cosas" ❌

## 📜 `git log`: mira el historial

```bash
git log --oneline
```

- Muestra **todos los commits** (tuyos y de tu equipo), del más nuevo al más viejo.
- Para salir, pulsa **`q`**.

---

# 👯 Parte 2: trabajar en equipo

## 📦 `git clone`: descarga un proyecto

```bash
git clone <url-del-repo>
```

- Descarga un proyecto **entero, con su historial**, a tu ordenador.
- Es lo primero que haces al **unirte a un proyecto** que ya existe.

## 🔗 `git remote`: ¿a dónde se conecta?

```bash
git remote -v                          # ver a dónde está conectado
git remote add origin <url-del-repo>   # conectar tu proyecto a GitHub
```

- **`origin`** es el nombre que se suele dar a la copia **en GitHub**.
- Si usaste `git clone`, ya viene conectado. 👌

## ⬆️ `git push`: sube tus cambios

```bash
git push
```

- Sube tus commits a GitHub para que **tu equipo los vea**.

## ⬇️ `git pull`: baja los cambios de los demás

```bash
git pull
```

- Trae lo que **tus compañeras han subido** y lo junta con lo tuyo.
- **Regla de oro:** haz `git pull` **antes de empezar a trabajar**. Así evitas líos. ✨

## 🌿 `git branch` y `git switch`: trabaja en tu propia rama

Una **rama** es como una **línea temporal paralela**: puedes probar cosas **sin tocar la versión principal** (`main`).

```bash
git branch                  # ver las ramas (la tuya lleva *)
git switch -c nuevo-menu    # crear una rama nueva y saltar a ella
git switch main             # volver a la rama principal
```

- Cada persona trabaja en **su rama** → nadie pisa el trabajo de nadie.
- Verás también `git checkout`: es el comando antiguo, hace lo mismo.

## 🤝 `git merge`: junta dos ramas

```bash
git switch main          # ve a la rama donde quieres juntar
git merge nuevo-menu     # trae los cambios de "nuevo-menu"
```

- Une el trabajo de una rama con otra.
- 💥 Si dos personas cambiaron **la misma línea**, Git te avisa con un **conflicto**. Tranquila: solo tienes que **elegir qué versión se queda**. 

> 💡 En GitHub, lo normal es juntar ramas con un **Pull Request**: pides que revisen tu rama antes de unirla a `main`.

---

## 🔁 El ciclo de cada día (en equipo)

```bash
git pull                       # 1. bajo lo nuevo del equipo
git switch -c mi-tarea         # 2. creo mi rama
# ...trabajo...
git status                     # 3. ¿qué he cambiado?
git add .                      # 4. lo meto en la cesta
git commit -m "Mi mensaje"     # 5. guardo la partida
git push -u origin mi-tarea    # 6. subo mi rama a GitHub
```

Después, abres un **Pull Request** en GitHub, tu equipo lo revisa y se junta con `main`. 

---

## ✅ Resumen

| Comando | Para qué sirve |
|---|---|
| `git --version` | Comprobar que Git está instalado |
| `git config` | Decir quién eres |
| `git init` | Empezar un proyecto con Git |
| `git status` | Ver qué ha cambiado |
| `git diff` | Ver los cambios línea a línea |
| `git add` | Elegir qué guardar |
| `git commit -m "..."` | Guardar con un mensaje |
| `git log --oneline` | Ver el historial |
| `git clone` | Descargar un proyecto |
| `git remote` | Conectar con GitHub |
| `git push` | Subir tus cambios |
| `git pull` | Bajar los cambios del equipo |
| `git branch` / `git switch` | Ver, crear y cambiar de rama |
| `git merge` | Juntar ramas |

📄 ¿Quieres tenerlos todos a mano? Puedes echar un vistazo a la **chuleta oficial de Git**: 👉 [Git Cheat Sheet](https://git-scm.com/cheat-sheet)

¿Cuál fue el primer comando de Git que aprendiste? ¡Cuéntamelo! 💜
