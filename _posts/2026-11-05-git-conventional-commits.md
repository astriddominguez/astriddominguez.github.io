---
layout: post
title: "Conventional Commits: cómo escribir mensajes de commit que se entiendan"
date: 2026-11-05 02:00:00 +0000
tags: [git, github, conventional-commits, buenas-practicas, principiantes]
---

En la entrada anterior dije que el mensaje de un commit tiene que explicar qué cambiaste. Vale, pero ¿cómo? Si cada persona del equipo lo escribe a su manera, el historial acaba siendo un caos.

Para eso existe Conventional Commits: una forma de escribir los mensajes que usa muchísima gente. No es una regla oficial de Git (Git acepta cualquier texto), pero es lo más parecido a un estándar que hay.

## El problema

Esto es un historial real (bueno, casi):

```
ahora sí
arreglos
cambios
asdf
cambios 2
```

Ahora imagina que algo se ha roto y tienes que encontrar qué commit fue. Suerte.

## La forma

```
tipo(ámbito): descripción
```

- **tipo**: qué clase de cambio es. Es obligatorio.
- **ámbito**: qué parte del proyecto tocas. Es opcional y va entre paréntesis.
- **descripción**: qué has hecho, en pocas palabras.

Ejemplos:

```
feat: añade buscador al blog
fix(menu): arregla el menú en móvil
docs: actualiza el README
```

Con solo leer la primera palabra ya sabes de qué va cada commit.

## Los tipos

| Tipo | Cuándo lo uso |
|---|---|
| `feat` | Algo nuevo: una función, una página, un botón. |
| `fix` | Arreglo un error. |
| `docs` | Solo cambio documentación (README, comentarios…). |
| `style` | Formato: espacios, comas, sangría. No cambia lo que hace el código. |
| `refactor` | Reorganizo el código sin cambiar lo que hace. |
| `perf` | Hago que algo vaya más rápido. |
| `test` | Añado o cambio tests. |
| `build` | Cambios en cómo se construye el proyecto o en las dependencias. |
| `ci` | Cambios en la integración continua (de esto va la próxima entrada). |
| `chore` | Tareas de mantenimiento que no encajan en lo demás. |

Si no tienes claro cuál usar, pregúntate qué notaría alguien que usa tu web. Si hay algo nuevo, es `feat`. Si algo que antes fallaba ahora funciona, es `fix`. Y si no notaría nada, probablemente sea `refactor`, `style`, `docs` o `chore`.

## Algunas reglas para la descripción

- Que sea corta: la primera línea, de unos 50 caracteres como mucho.
- Que diga qué y por qué, no cómo. El cómo ya está en el código.
- Que sea siempre igual: elige un estilo ("añade…" o "añadir…") y no lo mezcles.
- Un commit, un cambio. Si te sale una "y" ("arregla el menú y cambia el footer"), son dos commits.

Así quedan algunos mensajes típicos, antes y después:

| Antes | Después |
|---|---|
| `cambios` | `style: ordena el CSS de la portada` |
| `arreglado` | `fix(menu): el enlace a contacto daba 404` |
| `nueva entrada` | `feat(blog): añade entrada sobre Git vs GitHub` |
| `readme` | `docs: explica cómo arrancar el blog en local` |

## Cuando necesitas explicar más

La primera línea es el título. Si hace falta contar más, deja una línea en blanco y escribe debajo. Si usas `git commit` sin `-m`, se abre un editor para escribirlo:

```
fix(menu): arregla el menú en móvil

En pantallas pequeñas el menú tapaba el título.
Ahora se esconde y aparece al pulsar el botón.

Closes #12
```

El cuerpo, en medio, explica el porqué. El pie, abajo, sirve para notas: `Closes #12` cierra la issue 12 de GitHub cuando el cambio llega a `main`.

## Cuando rompes algo a propósito

A veces un cambio hace que algo deje de funcionar como antes. Por ejemplo, cambias las URLs del blog y los enlaces viejos ya no van.

Eso se avisa de dos formas, y puedes usar una o las dos: con una `!` después del tipo, o con `BREAKING CHANGE:` en el pie explicando qué se rompe.

```
feat!: cambia la estructura de las URLs del blog

BREAKING CHANGE: los enlaces antiguos ya no funcionan.
```

Así nadie se lleva un susto al actualizar.

## Esto conecta con las versiones

Muchos proyectos numeran sus versiones así: v1.4.2. Se llama versionado semántico y cada número significa algo:

- El primero (**1**.4.2) cambia cuando hay algo grande que deja de funcionar como antes.
- El segundo (1.**4**.2), cuando hay algo nuevo que no rompe nada.
- El tercero (1.4.**2**), cuando solo hay arreglos.

¿Te suena? Es justo lo que dicen los tipos de commit:

| Si desde la última versión hay… | Subes… |
|---|---|
| algún `feat!` o `BREAKING CHANGE` | el primero → v2.0.0 |
| algún `feat` | el segundo → v1.5.0 |
| solo `fix` | el tercero → v1.4.3 |

Ya no tienes que pensar qué número poner: te lo dicen tus commits.

## Y con los mensajes de los tags

Cuando sacas una versión, la marcas con un tag: una etiqueta pegada a un commit. Mejor usar tags anotados, que son los que llevan mensaje:

```bash
git tag -a v1.5.0 -m "v1.5.0

feat: buscador en el blog
feat(blog): entrada sobre Git vs GitHub
fix(menu): arreglado el menú en móvil"
```

`-a` significa anotado, y `-m` es el mensaje, igual que en `commit`.

Fíjate en que el mensaje del tag es casi un copia y pega de tus commits. Para verlos rápido:

```bash
git log v1.4.2..HEAD --oneline   # commits desde la última versión
```

Si tus commits están bien escritos, el resumen de la versión ya está hecho.

Y no te olvides de subir el tag, porque `git push` normal no lo hace:

```bash
git push origin v1.5.0
```

## Bonus: los nombres de las ramas

Puedes usar los mismos tipos para nombrar tus ramas, en minúsculas y con guiones:

```bash
git switch -c feat/buscador
git switch -c fix/menu-movil
git switch -c docs/readme
```

También verás `feature/` en vez de `feat/`. Las dos valen.

## Chuleta

```
feat:      algo nuevo
fix:       arreglo
docs:      documentación
style:     formato, sin cambiar lo que hace
refactor:  reorganizar código, sin cambiar lo que hace
perf:      más rápido
test:      tests
build:     construcción y dependencias
ci:        integración continua
chore:     mantenimiento

feat!: / BREAKING CHANGE:   rompe algo que antes funcionaba
```

Toda la especificación está en español en [conventionalcommits.org](https://www.conventionalcommits.org/es/). Es cortita, se lee en cinco minutos.

En la próxima entrada hablaré de CI/CD, y entre otras cosas veremos cómo hacer que GitHub revise tus mensajes de commit solo y te avise si no siguen el formato.

¿Tú ya escribías así tus commits o eras más de "cambios 2"?
