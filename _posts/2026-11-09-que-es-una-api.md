---
layout: post
title: "¿Qué es una API? Peticiones, métodos HTTP y códigos de estado"
date: 2026-11-09 01:00:00 +0000
category: "Backend y seguridad"
tags: [api, http, backend, principiantes]
---

Cuando usamos una web o una app del móvil, estamos intercambiando información todo el rato: nuestro lado, el **cliente** (el navegador o la app), le pide cosas al **servidor**, y el servidor le responde. Ese intercambio pasa a través de una **API**. Y no siempre es entre un cliente y un servidor: dos servidores también pueden hablar entre ellos usando APIs.

## ¿Qué es una API?

API significa *Application Programming Interface* (interfaz de programación de aplicaciones). Es la "puerta" que un programa abre para que otros programas puedan hablar con él: pedirle datos, añadir datos nuevos, modificarlos o eliminarlos.

Una forma sencilla de entenderlo es pensar en un restaurante. Tú (el cliente) no entras en la cocina (el servidor) a hacerte la comida. Le pides al camarero lo que quieres de la carta, él lleva el pedido a la cocina y te trae el plato. La API es el camarero, y la carta es la lista de cosas que puedes pedir.

## Cómo se habla con una API: petición y respuesta

Una API tiene una serie de **endpoints**: direcciones (URLs) a las que puedes enviar peticiones. Por ejemplo, `/usuarios` para trabajar con usuarios o `/productos` para trabajar con productos.

Cada **petición** tiene estas partes:

- **Método:** qué quieres hacer (pedir, crear, modificar, borrar…). Los vemos en el siguiente apartado.
- **URL:** a qué endpoint va la petición.
- **Cabeceras:** información extra, como el formato de los datos o el token que demuestra quién eres.
- **Cuerpo:** los datos que envías, normalmente en formato JSON. No todas las peticiones lo llevan.

Y cada **respuesta** tiene:

- **Código de estado:** un número que dice si ha ido bien o mal.
- **Cabeceras.**
- **Cuerpo:** los datos que te devuelve el servidor, también en JSON.

Un ejemplo. Esta petición pide los datos del usuario con id 1:

```
GET /usuarios/1
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
```

Y el servidor responde:

```
200 OK
Content-Type: application/json

{
  "id": 1,
  "nombre": "Astrid",
  "email": "astrid@ejemplo.com"
}
```

¿Ves la cabecera `Authorization`? Muchas APIs manejan información privada, así que necesitan saber quién hace cada petición. Para eso se usan los tokens, como los JWT que expliqué en el [post anterior]({% post_url 2026-11-09-jwt %}). Por eso es habitual que una API tenga también endpoints para iniciar sesión, obtener un token o renovarlo. Otras APIs son públicas y no piden nada.

## Los métodos HTTP

El método indica **qué quieres hacer** con los datos. Estos son los principales:

| Método | Para qué sirve | Ejemplo |
|---|---|---|
| **GET** | Pedir datos | `GET /usuarios/1` |
| **POST** | Crear algo nuevo | `POST /usuarios` |
| **PUT** | Reemplazar algo entero | `PUT /usuarios/1` |
| **PATCH** | Modificar una parte | `PATCH /usuarios/1` |
| **DELETE** | Borrar | `DELETE /usuarios/1` |

### GET

Sirve para **pedir datos**. No modifica nada en el servidor: puedes hacer la misma petición mil veces y no cambiará nada. Cada vez que abres una página web, tu navegador está haciendo un GET.

```
GET /usuarios/1
```

### POST

Sirve para **crear algo nuevo**: registrar un usuario, publicar un comentario, hacer un pedido… Los datos van en el cuerpo de la petición:

```
POST /usuarios

{
  "nombre": "Astrid",
  "email": "astrid@ejemplo.com",
  "edad": 25
}
```

Fíjate en que la URL no lleva id: el servidor es el que le asigna uno al crearlo.

### PUT vs PATCH

Los dos sirven para modificar, y aquí es donde mucha gente se lía. La diferencia es:

- **PUT reemplaza el recurso entero.** Envías el objeto completo, tal y como quieres que quede.
- **PATCH cambia solo una parte.** Envías únicamente lo que quieres cambiar.

Imagina que tenemos este usuario guardado:

```json
{ "nombre": "Astrid", "email": "astrid@ejemplo.com", "edad": 25 }
```

Y queremos cambiar solo el email. Con **PATCH** enviamos solo eso:

```
PATCH /usuarios/1

{ "email": "nuevo@ejemplo.com" }
```

Resultado: `{ "nombre": "Astrid", "email": "nuevo@ejemplo.com", "edad": 25 }`. Perfecto.

Pero si hacemos lo mismo con **PUT**:

```
PUT /usuarios/1

{ "email": "nuevo@ejemplo.com" }
```

Como PUT reemplaza el objeto entero, el resultado sería `{ "email": "nuevo@ejemplo.com" }`. ¡Adiós al nombre y a la edad! Con PUT tienes que enviar siempre el objeto completo.

> En la práctica, cada API se programa a su manera y algunas no siguen esta regla al pie de la letra. Pero así es como *deberían* funcionar.

### DELETE

Sirve para **borrar**:

```
DELETE /usuarios/1
```

## Los códigos de estado

Cada respuesta trae un número de tres cifras que dice qué ha pasado. Lo más útil es fijarse en la primera cifra:

- **2xx:** todo ha ido bien.
- **4xx:** el error es tuyo, del cliente (has pedido algo mal).
- **5xx:** el error es del servidor.

Los que más vas a ver:

| Código | Significado | Cuándo aparece |
|---|---|---|
| **200** OK | Todo bien | Un GET que devuelve datos |
| **201** Created | Creado | Un POST que ha creado algo |
| **204** No Content | Bien, pero no hay nada que devolver | Un DELETE que ha borrado algo |
| **400** Bad Request | Petición incorrecta | Te falta un dato o lo has enviado mal |
| **401** Unauthorized | No sé quién eres | No has enviado token, o no es válido o ha caducado ([¿te suena?]({% post_url 2026-11-09-jwt %})) |
| **403** Forbidden | Sé quién eres, pero no tienes permiso | Un usuario normal intentando entrar en algo de administrador |
| **404** Not Found | No existe | Pides un usuario que no está |
| **500** Internal Server Error | El servidor ha fallado | Un error en el código del servidor |

La diferencia entre 401 y 403 es parecida a la de una discoteca: 401 es que no te dejan entrar porque no has enseñado el DNI, y 403 es que lo has enseñado, pero no estás en la lista VIP.

## Pruébalo tú

Hay APIs públicas pensadas para practicar. Una de ellas es [JSONPlaceholder](https://jsonplaceholder.typicode.com), que devuelve datos de prueba.

Abre esta dirección en el navegador y estarás haciendo un GET:

[https://jsonplaceholder.typicode.com/users/1](https://jsonplaceholder.typicode.com/users/1)

Te devolverá un JSON con los datos de un usuario de ejemplo. Prueba a cambiar el `1` por `9999`: como ese usuario no existe, recibirás un **404**.

Si tienes una terminal, puedes probar también un POST con `curl`:

```bash
curl -X POST https://jsonplaceholder.typicode.com/users \
  -H "Content-Type: application/json" \
  -d '{"name": "Astrid"}'
```

La respuesta será tu usuario con un id nuevo y un código **201**. (Tranquilidad: es una API de prueba y no guarda nada de verdad).

## Para terminar

Una API es la forma en que los programas hablan entre sí: el cliente hace una petición con un método (GET, POST, PUT, PATCH o DELETE) a un endpoint, y el servidor responde con un código de estado y, normalmente, unos datos en JSON.

Ahora bien, si tú creas una API, ¿cómo sabe otra persona qué endpoints tiene y qué datos tiene que enviar? De eso hablaré en el próximo post: OpenAPI y Swagger.

