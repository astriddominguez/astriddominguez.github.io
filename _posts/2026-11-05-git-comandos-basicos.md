---
layout: post
title: "Git para principiantes: los comandos básicos"
date: 2026-11-05 01:00:00 +0000
category: "Git y buenas prácticas"
tags: [git, principiantes, terminal]
---

En el post anterior expliqué que Git y GitHub no son lo mismo. Hoy toca usar Git de verdad, con los comandos que más he necesitado mientras montaba este mismo blog.

Antes de empezar, una forma sencilla de entenderlo: Git es como una partida de videojuego. Puedes guardar la partida cuando quieras y volver atrás si algo sale mal. Y además es una partida multijugador: otras personas pueden jugar la misma desde su casa, subiendo sus avances y bajando los tuyos. Esas dos cosas, no perder tu trabajo y trabajar en equipo, son para lo que sirve Git.

## Antes de nada: instalar y presentarte

Si no tienes Git instalado, la [guía oficial en español](https://git-scm.com/book/es/v2/Inicio---Sobre-el-Control-de-Versiones-Instalaci%C3%B3n-de-Git) lo explica para cada sistema. Para comprobar que funciona:

```bash
git --version
```

Si sale un número de versión, listo.

Después hay que decirle a Git quién eres, porque cada cambio que guardes irá firmado con tu nombre. Se hace una sola vez por ordenador, y conviene usar el mismo email que en GitHub:

```bash
git config --global user.name "Tu Nombre"
git config --global user.email "tu@email.com"
```

## Trabajando sola

### git init

```bash
git init
```

Convierte una carpeta normal en un repositorio. Lo que hace es crear una carpeta oculta, `.git`, donde se guarda todo el historial. Si la borras, pierdes el historial, así que mejor no tocarla.

### git status

```bash
git status
```

Te dice qué archivos han cambiado. Los que salen en rojo están cambiados pero no preparados para guardar; los verdes, sí. Yo lo uso cada vez que no sé en qué punto estoy. No modifica nada, así que puedes escribirlo todas las veces que quieras.

### git diff

```bash
git diff
```

Enseña línea a línea qué has añadido y qué has quitado. Para salir, pulsa `q`. Si no lo sabes, es fácil quedarse atascada en esa pantalla sin saber cómo volver.

### git add

```bash
git add index.html   # prepara un archivo concreto
git add .            # prepara todos los cambios
```

Aquí todavía no se guarda nada: solo eliges qué cambios van a entrar en el próximo guardado. Es como meter cosas en la cesta antes de pasar por caja.

### git commit

```bash
git commit -m "Añado la página de inicio"
```

Ahora sí: guarda una "foto" de lo que había en la cesta, con un mensaje. El mensaje es para ti dentro de un mes o para quien trabaje contigo, así que merece la pena que diga algo. "Arreglo el menú" sirve; "cambios" no.

### git log

```bash
git log --oneline
```

Muestra el historial de commits, del más reciente al más antiguo. También se sale con `q`.

## Trabajando en equipo

### git clone

```bash
git clone <url-del-repo>
```

Descarga un proyecto entero, con todo su historial. Es lo primero que haces cuando te unes a un proyecto que ya existe.

### git remote

```bash
git remote -v                          # ver a dónde está conectado
git remote add origin <url-del-repo>   # conectarlo a GitHub
```

`origin` es el nombre que se suele dar a la copia que está en GitHub. Si el proyecto lo has descargado con `git clone`, ya viene conectado.

Con este comando me pasó algo que quiero contar. La primera vez escribí la URL mal: puse `.gi` en lugar de `.git` al final. Me di cuenta, lo volví a escribir bien, y Git me contestó `error: remoto origin ya existe`. Claro, el primero, el mal escrito, ya se había guardado. Luego el `push` fallaba con "Repository not found" y yo no entendía por qué. La solución era cambiar la URL en vez de volver a añadirla:

```bash
git remote set-url origin <url-correcta>
```

### git push

```bash
git push
```

Sube tus commits a GitHub para que los demás los vean.

Otra cosa que me pilló por sorpresa: la primera vez, la terminal me pidió usuario y contraseña, y mi contraseña de GitHub no funcionaba. GitHub ya no acepta la contraseña de la cuenta desde la terminal; hay que crear un *token* (en Settings → Developer settings → Personal access tokens) y pegarlo donde pide la contraseña. Al pegarlo no se ve nada en pantalla, pero sí se está escribiendo.

### git pull

```bash
git pull
```

Trae lo que han subido los demás y lo junta con lo tuyo. Lo mejor es hacerlo antes de ponerte a trabajar, así partes de la versión más reciente y te ahorras conflictos.

### git branch y git switch

Una rama es como una línea temporal paralela: puedes probar cosas sin tocar la versión principal, que normalmente se llama `main`.

```bash
git branch                  # ver las ramas (la actual lleva un *)
git switch -c nuevo-menu    # crear una rama nueva y cambiarte a ella
git switch main             # volver a la principal
```

Si cada persona trabaja en su rama, nadie pisa el trabajo de nadie. En tutoriales antiguos verás `git checkout` para lo mismo; `switch` es la versión moderna.

### git merge

```bash
git switch main          # te pones en la rama que va a recibir los cambios
git merge nuevo-menu     # traes los cambios de nuevo-menu
```

Une el trabajo de una rama con otra. Si dos personas han cambiado la misma línea, Git no sabe con cuál quedarse y te avisa con un conflicto. Asusta la primera vez, pero solo tienes que abrir el archivo y elegir qué versión se queda.

En GitHub, lo normal es no hacer el merge directamente, sino abrir un *Pull Request*: pides que alguien revise tu rama antes de unirla a `main`.

## Un día normal

Así suele quedar el ciclo cuando trabajas con más gente:

```bash
git pull                       # bajo lo último
git switch -c mi-tarea         # creo mi rama
# ...trabajo...
git status                     # miro qué he cambiado
git add .
git commit -m "Lo que he hecho"
git push -u origin mi-tarea    # subo mi rama
```

Y después abro un Pull Request en GitHub para que lo revisen.

## Resumen

| Comando | Para qué sirve |
|---|---|
| `git --version` | Comprobar que Git está instalado |
| `git config` | Decir quién eres |
| `git init` | Empezar un proyecto con Git |
| `git status` | Ver qué ha cambiado |
| `git diff` | Ver los cambios línea a línea |
| `git add` | Preparar cambios para guardar |
| `git commit -m "..."` | Guardar con un mensaje |
| `git log --oneline` | Ver el historial |
| `git clone` | Descargar un proyecto |
| `git remote` | Ver o cambiar la conexión con GitHub |
| `git push` | Subir tus cambios |
| `git pull` | Bajar los cambios de los demás |
| `git branch` / `git switch` | Ver, crear y cambiar de rama |
| `git merge` | Juntar ramas |

Si quieres tenerlos todos a mano, la [chuleta oficial de Git](https://git-scm.com/cheat-sheet) está muy bien para imprimirla o dejarla abierta en otra pestaña.


