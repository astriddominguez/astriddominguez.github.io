---
layout: post
title: "Git vs GitHub: ¿no son lo mismo?"
date: 2026-11-05
tags: [git, github, gitlab, principiantes]
---

Los principiantes a menudo confunden **Git** con **GitHub**. Son herramientas **diferentes**, pero **complementarias**:

- **Git** lleva el **control de versiones** de tu proyecto.
- **GitHub** es una **plataforma en la nube** donde guardas esas versiones y puedes **compartirlas con otros colaboradores**.

---

## 💻 Git: tu historial, en tu ordenador

- Se instala en tu **entorno local** (tu PC).
- Funciona **sin internet**.
- Cada vez que guardas una "foto" de tu proyecto se llama **commit**, y va acompañada de un **mensaje** que explica qué cambiaste.
- Así puedes **volver a una versión anterior** si la lías. 😅

```bash
git add .                         # elige qué cambios guardar
git commit -m "Añado la portada"  # guarda la foto con su mensaje
```

---

## 🌐 GitHub: tu proyecto, en la nube

- Es una **página web** que guarda una copia de tu proyecto (con todo su historial).
- Sirve para tener una **copia de seguridad** y **trabajar en equipo**.
- Extra: con **GitHub Pages** puedes publicar una web gratis (¡como este blog!).

```bash
git push   # sube tus commits a GitHub
git pull   # baja los cambios más recientes
```

---

## 🔀 Alternativas a GitHub

GitHub no es la única opción. Todas estas plataformas usan **Git** por debajo, así que si sabes Git, sabes usarlas:

| Plataforma | En pocas palabras |
|---|---|
| **GitHub** | La más popular. Mucha comunidad. |
| **GitLab** | Muy parecida a GitHub. Puedes instalarla **en tu propio servidor**. |
| **Bitbucket** | De Atlassian. Encaja con Jira y Trello. |
| **Codeberg** | Gratis y sin ánimo de lucro. |

### 🏠 GitLab en tu propio servidor

Imagina que **no quieres que tu código esté en internet**. Con GitLab puedes instalarlo en un **servidor propio** (de tu casa o de tu empresa) y tener algo muy parecido a GitHub, pero **bajo tu control**.

- GitLab tiene una **versión gratuita** que puedes instalar tú misma.
- Puede **ahorrarte dinero** si ya tienes los recursos (un servidor y tiempo para mantenerlo).
- Ojo: el servidor, las actualizaciones y las copias de seguridad corren **de tu cuenta**.

> 💡 GitHub también se puede instalar en un servidor propio (GitHub Enterprise Server), pero es **de pago** y está pensado para empresas. Por eso GitLab suele ser la opción preferida para esto.

---

## ✅ Resumen

1. **Git** guarda las versiones → en tu ordenador.
2. **GitHub, GitLab, Bitbucket…** guardan y comparten → en la nube (o en tu propio servidor).
3. Primero aprende Git. Luego elige plataforma.

¿Tú también pensabas que eran lo mismo? ¡Cuéntamelo! 💜
