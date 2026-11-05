---
layout: post
title: "Git vs GitHub: ¿no son lo mismo?"
date: 2026-11-05
tags: [git, github, gitlab, principiantes]
---

Cuando se empieza a programar es muy habitual mezclar Git y GitHub, porque casi siempre aparecen juntos. Pero son dos cosas distintas que se complementan: Git es la herramienta que guarda las versiones de tu proyecto, y GitHub es una web donde subes esas versiones para tenerlas en la nube y compartirlas con otras personas.

## Git: el historial, en tu ordenador

Git es un programa que se instala en tu ordenador y funciona sin internet. Cada vez que quieres guardar el estado de tu proyecto haces un *commit*, que es como sacar una foto de cómo están tus archivos en ese momento, junto con un mensaje que explica qué has cambiado. Gracias a eso, si algo se rompe, puedes volver a una versión anterior.

```bash
git add .                         # preparo todos los cambios
git commit -m "Añado la portada"  # los guardo con un mensaje
```

Todo esto ocurre en tu ordenador. Nadie más lo ve todavía.

## GitHub: el proyecto, en la nube

GitHub es una página web que guarda una copia de tu proyecto con todo su historial. Sirve como copia de seguridad, para trabajar en equipo y para enseñar tu código a otras personas.

Para pasar los cambios de tu ordenador a GitHub y al revés, se usan dos comandos de Git:

```bash
git push   # subo mis commits a GitHub
git pull   # bajo los cambios que hay en GitHub
```

Un ejemplo cercano: este blog. Los posts los escribo en mi ordenador y los guardo con Git. Cuando hago `git push`, se suben a un repositorio de GitHub, y GitHub Pages los convierte en la web que estás leyendo, gratis.

## No solo existe GitHub

GitHub es la plataforma más conocida, pero hay otras. Todas usan Git por debajo, así que lo que aprendas de Git te sirve en cualquiera:

| Plataforma | En pocas palabras |
|---|---|
| GitHub | La más popular, con mucha comunidad. |
| GitLab | Muy parecida a GitHub. Se puede instalar en tu propio servidor. |
| Bitbucket | De Atlassian. Se integra bien con Jira y Trello. |
| Codeberg | Gratuita y sin ánimo de lucro. |

### GitLab en un servidor propio

Esto me pareció lo más interesante de las alternativas. Si no quieres que tu código esté en servidores de otra empresa, puedes instalar GitLab en un servidor tuyo, en casa o en tu empresa, y tener algo muy parecido a GitHub pero bajo tu control.

GitLab tiene una versión gratuita que puedes instalar tú misma, y si ya tienes un servidor puede salir más barato que pagar por un servicio. La parte menos bonita es que entonces el mantenimiento es cosa tuya: actualizaciones, copias de seguridad y que el servidor no se caiga.

GitHub también se puede instalar en un servidor propio (se llama GitHub Enterprise Server), pero es de pago y está pensado para empresas grandes. Por eso, para esto, GitLab suele ser la opción más elegida.

## En resumen

Git guarda las versiones de tu proyecto en tu ordenador. GitHub, GitLab, Bitbucket y compañía guardan una copia en la nube (o en tu propio servidor) para compartirla. Por eso tiene sentido aprender primero Git: la plataforma la puedes elegir después, y cambiar de una a otra no cuesta mucho.

En el siguiente post explico los comandos básicos de Git para empezar a usarlo.
