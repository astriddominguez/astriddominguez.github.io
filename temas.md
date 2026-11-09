---
layout: page
title: Temas
permalink: /temas/
---

Aquí tienes todas las entradas agrupadas por tema, ordenadas para leerlas de la primera a la última.

{% assign temas = site.categories | sort %}
{% for tema in temas %}
## {{ tema[0] }}

{% assign entradas = tema[1] | reverse %}
{% for post in entradas %}
1. [{{ post.title }}]({{ post.url | relative_url }}) <small>({{ post.date | date: "%d/%m/%Y" }})</small>
{% endfor %}
{% endfor %}
