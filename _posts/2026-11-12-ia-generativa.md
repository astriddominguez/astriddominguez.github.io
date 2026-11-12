---
layout: post
title: "IA generativa: qué es, qué modelos existen y cómo hacer que responda con tus datos (RAG, BM25 y re-ranking)"
date: 2026-11-12 00:00:00 +0000
category: "Inteligencia artificial"
tags: [ia, ia-generativa, llm, rag, bm25, embeddings, re-ranking, ollama, python, principiantes]
published: true
---

En el [post de frameworks]({% post_url 2026-11-11-fastapi-django-flask %}) comentaba que FastAPI es la opción más habitual para **servir modelos de IA** como APIs. Hoy toca hablar de esos modelos: qué es la **IA generativa**, qué modelos se usan ahora mismo (en la nube y en tu propio ordenador) y qué técnicas existen para que respondan bien a preguntas sobre **tus propios datos**: RAG, BM25, búsqueda semántica, búsqueda híbrida y re-ranking.

Es un post largo, así que vamos por partes.

## ¿Qué es la IA generativa?

La **inteligencia artificial "clásica"** (o, más bien, el machine learning clásico) se dedica sobre todo a **clasificar o predecir**: ¿este correo es spam o no?, ¿cuánto costará esta casa?, ¿qué hay en esta foto?

La **IA generativa** da un paso más: **crea contenido nuevo**. Texto, código, imágenes, audio, vídeo… que no existían antes y que se parecen a los datos con los que se entrenó el modelo.

Algunos ejemplos:

- **Texto y código:** ChatGPT, Claude, Gemini, GitHub Copilot…
- **Imágenes:** Midjourney, DALL·E, Stable Diffusion, FLUX…
- **Audio y voz:** generación de música, voces sintéticas, transcripción…
- **Vídeo:** modelos que generan clips a partir de una descripción.

En este post me voy a centrar en los modelos de **texto**, que son los que más vas a usar como desarrollador/a.

## ¿Cómo funciona un LLM?

Los modelos de texto se llaman **LLM** (*Large Language Models*, modelos de lenguaje grandes). La idea de base es sorprendentemente simple: **un LLM predice cuál es la siguiente palabra más probable**.

Bueno, en realidad no trabaja con palabras, sino con **tokens**, que son trozos de palabra. Por ejemplo, "programación" podría partirse en `program` + `ación`. Como regla aproximada, en español 100 tokens son unas 70 palabras.

Cuando le escribes "La capital de Francia es", el modelo calcula la probabilidad de cada posible token siguiente ("París" muy alta, "Roma" muy baja…), elige uno, lo añade al texto y **vuelve a empezar**. Así, token a token, genera la respuesta completa.

¿Y cómo "sabe" qué token viene después? Por el **entrenamiento**, que tiene varias fases:

1. **Preentrenamiento:** el modelo lee una cantidad gigantesca de texto (webs, libros, código…) y aprende a predecir el siguiente token. Aquí aprende el idioma, datos del mundo y a razonar en cierta medida.
2. **Ajuste fino con instrucciones:** se le entrena con ejemplos de conversaciones para que se comporte como un asistente que sigue instrucciones, en lugar de limitarse a continuar texto.
3. **Alineamiento con preferencias humanas:** se refinan sus respuestas para que sean útiles, honestas y seguras (técnicas como RLHF).

Por dentro, casi todos los LLM actuales usan una arquitectura llamada **Transformer**, presentada por Google en 2017 en el artículo [*Attention Is All You Need*](https://arxiv.org/abs/1706.03762). Su pieza clave es el mecanismo de **atención**, que permite al modelo fijarse en las partes relevantes de todo el texto a la vez para decidir el siguiente token.

### Conceptos que vas a ver en todas partes

- **Prompt:** el texto que le das al modelo (tu pregunta más las instrucciones).
- **System prompt:** instrucciones generales que definen cómo debe comportarse el modelo ("Eres un asistente que responde en español y de forma breve").
- **Ventana de contexto:** cuántos tokens puede "tener en la cabeza" a la vez (tu prompt + su respuesta). Los modelos actuales van desde unos pocos miles hasta más de un millón de tokens.
- **Temperatura:** controla la aleatoriedad. Cerca de 0, respuestas más predecibles; más alta, más creativas (y más propensas a inventar).
- **Parámetros:** los "números" internos que el modelo ajusta durante el entrenamiento. Se miden en miles de millones (*B*, de *billion*): un modelo de 8B tiene 8.000 millones de parámetros. Más parámetros suele significar más capacidad, pero también más memoria y más lentitud.

### Sus limitaciones (importante)

- **Alucinaciones:** como lo que hace es generar texto *probable*, a veces genera texto probable… pero falso. Y lo dice con total seguridad.
- **Fecha de corte:** solo sabe lo que había en sus datos de entrenamiento. No conoce lo que ha pasado después.
- **No conoce tus datos:** no ha leído la documentación interna de tu empresa, tus apuntes ni tu base de datos.

Las dos últimas limitaciones son justo las que resuelven las técnicas de la segunda mitad del post. Pero antes, veamos qué modelos hay.

## Modelos en la nube (no locales)

Son modelos que se ejecutan en los servidores de una empresa. Tú les envías el prompt a través de su **API** (sí, una [API]({% post_url 2026-11-09-que-es-una-api %}) como las que hemos visto, normalmente con una clave secreta en una cabecera) y te devuelven la respuesta. Pagas por tokens consumidos.

Las familias más usadas ahora mismo son:

| Empresa | Modelos | Dónde se usa |
|---|---|---|
| [Anthropic](https://www.anthropic.com) | **Claude** (Opus, Sonnet, Haiku) | claude.ai, API, Claude Code |
| [OpenAI](https://openai.com) | **GPT** y sus modelos de razonamiento | ChatGPT, API |
| [Google](https://ai.google.dev) | **Gemini** (Pro, Flash) | Gemini, Google AI Studio, Vertex AI |
| [xAI](https://x.ai) | **Grok** | X, API |
| [Mistral AI](https://mistral.ai) | **Mistral Large**, Medium… | Le Chat, API |

Casi todas las familias tienen varios tamaños: uno **grande y potente** (más lento y caro), uno **intermedio** y uno **pequeño y rápido** (barato, ideal para tareas sencillas o mucho volumen). Por ejemplo, en Claude son Opus, Sonnet y Haiku.

> Las versiones concretas cambian cada pocos meses, así que no te las aprendas de memoria. Consulta siempre la página de modelos de cada proveedor.

**Ventajas:**

- Son los modelos **más potentes** que existen.
- **No necesitas hardware**: funcionan igual desde un portátil viejo.
- Empiezas en minutos y escalan solos.

**Desventajas:**

- **Pagas por uso**, y con mucho tráfico la factura crece.
- **Tus datos salen de tu ordenador**: hay que revisar las condiciones de privacidad, sobre todo con datos sensibles o personales.
- **Dependes del proveedor**: si cambia precios, retira un modelo o tiene una caída, te afecta.
- Necesitas conexión a internet.

## Modelos locales (open weights)

Son modelos cuyos **pesos** (los parámetros entrenados) se publican para que cualquiera los descargue y los ejecute en su propio ordenador o servidor. Se les suele llamar **open weights** o "modelos abiertos".

Las familias más conocidas:

| Familia | Empresa | Notas |
|---|---|---|
| **Llama** | Meta | Una de las primeras familias abiertas populares |
| **Qwen** | Alibaba | Muy buenos en código y en varios idiomas, muchos tamaños |
| **DeepSeek** | DeepSeek | Muy potentes en razonamiento |
| **Mistral** | Mistral AI | Empresa europea, modelos eficientes |
| **Gemma** | Google | Versión abierta y pequeña de la tecnología de Gemini |
| **gpt-oss** | OpenAI | Los modelos abiertos de OpenAI |
| **Phi** | Microsoft | Modelos pequeños, pensados para equipos modestos |

Todos ellos los encuentras en [Hugging Face](https://huggingface.co/models), que es algo así como el "GitHub de los modelos de IA".

### ¿Qué necesito para ejecutarlos?

Lo que más importa es la **memoria** (la VRAM de la tarjeta gráfica o, en Mac con chip Apple, la memoria unificada). Para que quepan en equipos normales se usa la **cuantización**: guardar cada parámetro con menos precisión (por ejemplo, 4 bits en lugar de 16). El modelo pierde un poco de calidad, pero ocupa unas cuatro veces menos.

Como orientación muy aproximada, con cuantización a 4 bits:

| Tamaño del modelo | Memoria aproximada | Equipo |
|---|---|---|
| 1B – 4B | 2 – 4 GB | Casi cualquier portátil |
| 7B – 8B | 5 – 6 GB | Gráfica de 8 GB o Mac con 16 GB |
| 14B | 9 – 10 GB | Gráfica de 12 – 16 GB |
| 30B – 32B | 20 GB | Gráfica de 24 GB o Mac con 32 GB+ |
| 70B | 40 GB+ | Equipo potente o servidor |

### Herramientas para ejecutarlos

- **[Ollama](https://ollama.com):** la forma más fácil. Lo instalas y con un comando descargas y ejecutas un modelo. Además levanta una API local en `http://localhost:11434`.
- **[LM Studio](https://lmstudio.ai):** aplicación de escritorio con interfaz gráfica para buscar, descargar y chatear con modelos.
- **[llama.cpp](https://github.com/ggml-org/llama.cpp):** el motor que hay debajo de muchas de estas herramientas. Usa el formato de archivo **GGUF**, el más común para modelos cuantizados.
- **[vLLM](https://docs.vllm.ai):** pensado para servidores en producción con muchas peticiones a la vez.

Probarlo con Ollama es así de sencillo:

```bash
# Descarga el modelo (solo la primera vez) y abre un chat en la terminal
ollama run llama3.2

# Lista los modelos que tienes descargados
ollama list
```

Y desde Python, con su librería oficial (`pip install ollama`):

```python
from ollama import chat

respuesta = chat(
    model="llama3.2",
    messages=[{"role": "user", "content": "Explícame qué es una API en una frase"}],
)
print(respuesta.message.content)
```

**Ventajas:**

- **Privacidad total**: los datos nunca salen de tu máquina.
- **Sin coste por uso**: una vez tienes el equipo, es gratis.
- **Funcionan sin internet**.
- **Control total**: puedes elegir versión, ajustarlos (fine-tuning) y nadie te lo va a retirar.

**Desventajas:**

- Necesitas **hardware**, y los modelos grandes requieren equipos caros.
- Los modelos que caben en un ordenador normal son **menos capaces** que los grandes de la nube.
- **Más lentos** si tu equipo es modesto.
- Instalarlos, actualizarlos y mantenerlos en producción es trabajo tuyo.

### ¿Nube o local?

| Si… | Mejor |
|---|---|
| Necesitas la máxima calidad (razonamiento complejo, código difícil) | **Nube** |
| Trabajas con datos sensibles que no pueden salir de tu empresa | **Local** |
| Quieres empezar rápido y sin invertir en hardware | **Nube** |
| Vas a hacer muchísimas peticiones sencillas | **Local** (o un modelo pequeño en la nube) |
| Quieres aprender y experimentar sin gastar | **Local** |

Muchas empresas combinan las dos cosas: un modelo local para tareas sencillas o con datos privados y uno en la nube para lo más complejo.

## Hacer que el modelo use tus datos

Volvamos a las limitaciones: el modelo **no conoce tus datos** y **no sabe nada posterior a su fecha de corte**. Si le preguntas "¿cuántos días de vacaciones tengo según el convenio de mi empresa?", se lo inventará.

Hay tres grandes caminos para solucionarlo:

1. **Meterlo en el prompt:** si la información es poca, pégala directamente en el prompt. Simple y efectivo, pero no escala: no puedes pegar 10.000 documentos.
2. **Fine-tuning:** volver a entrenar el modelo con tus datos. Útil para cambiar el **estilo** o enseñarle un formato concreto, pero caro, lento de actualizar y no muy fiable para enseñarle **hechos** nuevos. Hoy se hace sobre todo con técnicas ligeras como **LoRA**, que solo entrenan una pequeña parte de los parámetros.
3. **RAG:** buscar la información relevante y dársela al modelo en el momento de la pregunta. Es la opción más usada, y la que vamos a ver en detalle.

## RAG: Retrieval-Augmented Generation

**RAG** significa "generación aumentada por recuperación". La idea es la de un examen con apuntes: en lugar de pedirle al modelo que se lo sepa todo de memoria, le dejamos **consultar los apuntes** antes de responder.

Funciona en dos fases:

**1. Indexación (se hace una vez, o cada vez que cambian los documentos):**

1. Cargas tus documentos (PDF, web, Markdown, base de datos…).
2. Los divides en trozos más pequeños, llamados **chunks** (por ejemplo, párrafos de unas 300 – 500 palabras). Esto se llama **chunking**.
3. Guardas esos trozos en un **índice** donde se puedan buscar rápido.

**2. Consulta (cada vez que alguien pregunta):**

1. **Recuperación (*retrieval*):** buscas en el índice los trozos más relevantes para la pregunta.
2. **Aumento:** los metes en el prompt junto con la pregunta.
3. **Generación:** el modelo responde basándose en esos trozos.

```text
Pregunta ──► Buscador ──► Top 5 trozos ──► Prompt (trozos + pregunta) ──► LLM ──► Respuesta
```

Un prompt de RAG típico se parece a esto:

```text
Responde a la pregunta usando SOLO la información del contexto.
Si la respuesta no está en el contexto, di que no lo sabes.

Contexto:
[1] ...trozo de documento...
[2] ...trozo de documento...

Pregunta: ¿Qué es un JWT?
```

Ventajas: el modelo responde con información actualizada, puedes **citar las fuentes** y, si cambian los documentos, solo tienes que actualizar el índice (sin reentrenar nada).

Todo depende de una cosa: **que el buscador encuentre los trozos correctos**. Si le das al modelo los trozos equivocados, responderá mal por muy bueno que sea. Por eso la mayor parte del trabajo en un RAG está en la **búsqueda**, y ahí es donde entran BM25, los embeddings y el re-ranking.

Para los ejemplos vamos a usar como "base de conocimiento" unas frases sobre los temas de este blog:

```python
documentos = [
    "Un JWT es un token firmado que el servidor entrega al usuario tras iniciar sesión.",
    "Swagger UI genera una página web interactiva para probar los endpoints de una API.",
    "OpenAPI es una especificación para describir una API REST en un archivo YAML o JSON.",
    "FastAPI valida automáticamente los datos de entrada usando los tipos de Python y Pydantic.",
    "Git es un sistema de control de versiones y GitHub es una plataforma para alojar repositorios.",
    "La integración continua ejecuta los tests automáticamente cada vez que subes cambios.",
    "Para autenticar a un usuario, la API comprueba sus credenciales y devuelve un token de acceso.",
]
```

## Búsqueda léxica: BM25

La forma más clásica de buscar es por **palabras clave**: encontrar los documentos que contienen las palabras de la pregunta. Es lo que hacían los buscadores de toda la vida, y el algoritmo estrella es **BM25** (*Best Matching 25*), que se usa desde los años 90 y sigue siendo el que hay por defecto en motores como Elasticsearch, OpenSearch o Lucene.

BM25 puntúa cada documento según tres ideas muy intuitivas:

1. **Frecuencia del término (TF):** si una palabra de la pregunta aparece varias veces en el documento, el documento es más relevante. Pero con **saturación**: que aparezca 20 veces no es 20 veces mejor que una. A partir de cierto punto, repetirla apenas suma.
2. **Frecuencia inversa de documento (IDF):** las palabras **raras** valen más que las comunes. Si buscas "qué es un JWT", la palabra "JWT" aparece en pocos documentos y es muy informativa; "es" y "un" aparecen en todos y casi no aportan nada.
3. **Normalización por longitud:** un documento muy largo contiene muchas palabras por pura casualidad, así que se le penaliza un poco frente a uno corto que va al grano.

La fórmula, para quien tenga curiosidad:

```text
score(D, Q) = Σ (para cada palabra q de la pregunta)
              IDF(q) · f(q, D) · (k1 + 1) / ( f(q, D) + k1 · (1 − b + b · |D| / longitud_media) )
```

Donde `f(q, D)` es cuántas veces aparece la palabra `q` en el documento `D`, `|D|` es la longitud del documento, y hay dos parámetros que puedes ajustar:

- **`k1`** (normalmente entre 1,2 y 2): controla la saturación. Cuanto más alto, más cuenta repetir una palabra.
- **`b`** (normalmente 0,75): controla cuánto se penaliza la longitud. Con 0, nada; con 1, al máximo.

En Python, con la librería `rank_bm25` (`pip install rank_bm25`):

```python
import re
from rank_bm25 import BM25Okapi

def tokenizar(texto):
    # Minúsculas y solo palabras (quitamos signos de puntuación)
    return re.findall(r"\w+", texto.lower())

bm25 = BM25Okapi([tokenizar(doc) for doc in documentos])

pregunta = "¿Qué es un JWT?"
puntuaciones = bm25.get_scores(tokenizar(pregunta))

ranking = sorted(zip(puntuaciones, documentos), reverse=True)
for puntuacion, doc in ranking[:3]:
    print(f"{puntuacion:.2f}  {doc}")
```

El documento del JWT sale el primero con mucha diferencia, porque "jwt" es una palabra rara y solo está en él.

**Ventajas de BM25:**

- **Muy rápido** y barato: no necesita GPU ni modelos.
- Excelente con **términos exactos**: nombres propios, códigos de error (`ERR_403`), referencias de producto, siglas…
- Fácil de entender y de depurar: sabes exactamente por qué un documento ha salido primero.

**Su gran problema: no entiende el significado.** Si preguntas *"¿cómo inicio sesión en la API?"*, no encontrará un documento que hable de "autenticar a un usuario con sus credenciales", porque no comparten las palabras importantes. No sabe que son sinónimos.

## Búsqueda semántica: embeddings

Para buscar por **significado** se usan los **embeddings**: un modelo convierte cada texto en un **vector**, una lista de cientos de números que representa su significado. La magia es que **textos con significado parecido tienen vectores cercanos**, aunque usen palabras distintas.

Así, "iniciar sesión" y "autenticar a un usuario" acaban muy cerca, y "receta de paella" muy lejos de los dos.

Para buscar:

1. Calculas el embedding de cada trozo de documento (una sola vez) y lo guardas.
2. Calculas el embedding de la pregunta.
3. Buscas los trozos cuyos vectores están más cerca, normalmente con la **similitud del coseno** (1 = mismo significado, 0 = nada que ver).

En Python, con `sentence-transformers` (`pip install sentence-transformers`) y un modelo que entiende español:

```python
from sentence_transformers import SentenceTransformer, util

modelo = SentenceTransformer("paraphrase-multilingual-MiniLM-L12-v2")

vectores_docs = modelo.encode(documentos)

pregunta = "¿Cómo inicio sesión en la API?"
vector_pregunta = modelo.encode(pregunta)

similitudes = util.cos_sim(vector_pregunta, vectores_docs)[0]

ranking = sorted(zip(similitudes.tolist(), documentos), reverse=True)
for similitud, doc in ranking[:3]:
    print(f"{similitud:.2f}  {doc}")
```

Ahora sí aparecen arriba los documentos de autenticación y del JWT, aunque no contengan las palabras "inicio" ni "sesión" en la misma forma.

A esto también se le llama **búsqueda densa** (porque los vectores están "llenos" de números), frente a la **búsqueda dispersa** de BM25 (cuyos vectores tienen un hueco por cada palabra del vocabulario y casi todos a cero).

Cuando tienes miles o millones de trozos, no puedes comparar la pregunta con todos uno a uno. Para eso existen las **bases de datos vectoriales**, que hacen la búsqueda de vecinos cercanos de forma muy rápida: [Chroma](https://www.trychroma.com), [Qdrant](https://qdrant.tech), [Weaviate](https://weaviate.io), [Milvus](https://milvus.io), [Pinecone](https://www.pinecone.io) o la extensión [pgvector](https://github.com/pgvector/pgvector) de PostgreSQL.

**Ventajas:**

- Entiende **sinónimos, paráfrasis e incluso otros idiomas**.
- Funciona bien con preguntas en lenguaje natural.

**Desventajas:**

- Peor que BM25 con **términos exactos**: un código de producto o un nombre poco común puede "diluirse" en el significado general.
- Necesita un modelo de embeddings (más cómputo) y es más difícil de depurar: no es obvio por qué dos vectores están cerca.

## Búsqueda híbrida: lo mejor de los dos

Como BM25 y los embeddings fallan en cosas distintas, lo más habitual hoy es **usar los dos a la vez** y combinar los resultados. Es la **búsqueda híbrida**.

El problema es que sus puntuaciones no se pueden comparar directamente (BM25 da números como 4,7 y el coseno da 0,63). La forma más usada de combinarlos es **Reciprocal Rank Fusion (RRF)**, que en lugar de mirar las puntuaciones mira **las posiciones**:

```text
RRF(documento) = Σ  1 / (k + posición del documento en cada ranking)
```

Con `k` normalmente igual a 60. Un documento que sale bien colocado en los dos rankings suma más que uno que solo destaca en uno.

```python
def rrf(rankings, k=60):
    """Combina varios rankings (listas de índices de documento, del mejor al peor)."""
    puntuaciones = {}
    for ranking in rankings:
        for posicion, doc_id in enumerate(ranking, start=1):
            puntuaciones[doc_id] = puntuaciones.get(doc_id, 0) + 1 / (k + posicion)
    return sorted(puntuaciones, key=puntuaciones.get, reverse=True)

# Índices de documentos ordenados por cada buscador
ranking_bm25 = [0, 6, 3, 1]
ranking_embeddings = [6, 0, 2, 3]

print(rrf([ranking_bm25, ranking_embeddings]))  # [0, 6, 3, 2, 1]
```

La mayoría de bases de datos vectoriales y motores como Elasticsearch ya traen la búsqueda híbrida incorporada.

## Re-ranking: una segunda opinión más precisa

Los buscadores anteriores tienen que ser **rápidos**, porque recorren todos los documentos. Para conseguirlo, el modelo de embeddings procesa la pregunta y cada documento **por separado** (por eso se llama **bi-encoder**) y luego solo compara vectores. Es rápido, pero pierde matices, porque nunca "lee" la pregunta y el documento juntos.

El **re-ranking** añade una segunda fase:

1. El buscador (BM25, embeddings o híbrido) recupera muchos candidatos, por ejemplo los **50 mejores**. Aquí buscamos no dejarnos nada importante fuera (**recall**).
2. Un **re-ranker** vuelve a puntuar solo esos 50 con más cuidado y los reordena. Nos quedamos con los **5 mejores** para el prompt. Aquí buscamos que lo que pasa sea realmente relevante (**precisión**).

El re-ranker suele ser un **cross-encoder**: un modelo que recibe **la pregunta y el documento juntos** y devuelve directamente una puntuación de relevancia. Como los lee a la vez, entiende mucho mejor si el documento responde de verdad a la pregunta. El precio es que es mucho más lento, por eso solo se aplica sobre unos pocos candidatos y no sobre toda la colección.

```text
             BM25 / embeddings              Re-ranker
Millones ──────────────────────► 50 ───────────────────► 5 ──► LLM
de trozos       (rápido)          candidatos  (preciso)
```

Con `sentence-transformers` y un re-ranker multilingüe:

```python
from sentence_transformers import CrossEncoder

reranker = CrossEncoder("BAAI/bge-reranker-v2-m3")

pregunta = "¿Cómo inicio sesión en la API?"
candidatos = [documentos[i] for i in [0, 6, 3, 2]]  # los que devolvió la búsqueda híbrida

puntuaciones = reranker.predict([(pregunta, doc) for doc in candidatos])

for puntuacion, doc in sorted(zip(puntuaciones, candidatos), reverse=True):
    print(f"{puntuacion:.3f}  {doc}")
```

Además de los modelos abiertos como los de la familia **bge-reranker**, hay re-rankers como servicio, como [Cohere Rerank](https://docs.cohere.com/docs/rerank-overview) o los de [Jina AI](https://jina.ai/reranker/). Y también se puede usar **el propio LLM como re-ranker**, pidiéndole que puntúe la relevancia de cada trozo, aunque es más caro.

En la práctica, añadir un re-ranker es de las mejoras que **más calidad aportan a un RAG con menos esfuerzo**.

## Otras estrategias que conviene conocer

### Para mejorar la recuperación

- **Buen chunking:** respetar la estructura del documento (cortar por secciones o párrafos, no a mitad de frase) y dejar algo de **solapamiento** entre trozos para no perder contexto en los cortes.
- **Metadatos y filtros:** guardar con cada trozo su autor, fecha, categoría… y filtrar antes de buscar ("solo documentos de 2026", "solo del departamento de RR. HH.").
- **Reescritura de la consulta (*query rewriting*):** usar el LLM para convertir la pregunta del usuario en una mejor consulta de búsqueda. Sobre todo en conversaciones: "¿y cuánto cuesta?" no se puede buscar sin saber de qué se hablaba antes.
- **Multi-query:** generar varias versiones de la pregunta, buscar con todas y combinar resultados (con RRF, por ejemplo).
- **HyDE (*Hypothetical Document Embeddings*):** pedirle al LLM que escriba una respuesta inventada a la pregunta y buscar con el embedding de **esa respuesta**. Suena raro, pero una respuesta se parece más a los documentos que una pregunta.
- **Contextual retrieval:** antes de indexar cada trozo, añadirle una frase que explique de qué documento viene y de qué trata ("Este fragmento pertenece al post sobre JWT y habla de…"). Así un trozo suelto como "caduca a los 15 minutos" deja de ser ambiguo.

### Más allá de RAG

- **Prompt engineering:** instrucciones claras, ejemplos de lo que esperas (*few-shot*) y pedir que razone paso a paso. Antes de montar nada complejo, mejora el prompt.
- **Salida estructurada:** pedir la respuesta en JSON con un esquema concreto (muy cómodo para validarla luego con Pydantic, como en FastAPI).
- **Tool use y agentes:** dejar que el modelo **llame a funciones** (consultar una base de datos, llamar a una API, buscar en la web…) y decida él qué hacer en cada paso. Un RAG puede ser simplemente una de esas herramientas.
- **MCP (*Model Context Protocol*):** un estándar abierto para conectar modelos con herramientas y fuentes de datos externas, de forma que una misma herramienta sirva para distintas aplicaciones de IA.

### Evaluar

Por último, algo que se suele olvidar: **mide**. Prepara un conjunto de preguntas con su respuesta correcta y comprueba:

- Si el buscador encuentra el trozo correcto entre los primeros (métricas como *recall@k* o *MRR*).
- Si la respuesta final es correcta y está basada en el contexto, no inventada.

Herramientas como [Ragas](https://docs.ragas.io) ayudan a automatizarlo. Sin evaluación, no sabrás si añadir un re-ranker o cambiar el chunking ha mejorado algo o lo ha empeorado.

## Resumen de técnicas

| Técnica | Qué hace | Cuándo usarla |
|---|---|---|
| **Prompt engineering** | Mejorar las instrucciones | Siempre, es lo primero |
| **Fine-tuning / LoRA** | Reentrenar el modelo | Cambiar estilo o formato, tareas muy específicas |
| **RAG** | Buscar información y dársela al modelo | Responder con tus datos o datos actualizados |
| **BM25** | Búsqueda por palabras clave | Términos exactos, códigos, nombres propios |
| **Embeddings** | Búsqueda por significado | Preguntas en lenguaje natural, sinónimos |
| **Búsqueda híbrida + RRF** | Combinar BM25 y embeddings | Casi siempre en un RAG serio |
| **Re-ranking** | Reordenar candidatos con un modelo más preciso | Cuando la calidad de los resultados importa |
| **Query rewriting / HyDE** | Mejorar la consulta antes de buscar | Preguntas vagas o conversaciones |
| **Agentes / tool use** | El modelo decide qué herramientas usar | Tareas de varios pasos |

## Para terminar

La IA generativa son modelos que **generan contenido nuevo** prediciendo, token a token, lo más probable. Puedes usarlos **en la nube** (Claude, GPT, Gemini…), con la máxima potencia y sin hardware, o **en local** (Llama, Qwen, DeepSeek, Gemma… con Ollama o LM Studio), con privacidad total y sin coste por uso.

Para que respondan con **tus datos**, la técnica estrella es **RAG**, y su calidad depende casi por completo de la búsqueda. Un esquema que funciona muy bien para empezar:

1. **Búsqueda híbrida**: BM25 + embeddings, combinados con RRF.
2. **Re-ranking** de los 20 – 50 mejores candidatos con un cross-encoder.
3. Los **5 mejores** al prompt, pidiendo al modelo que responda solo con ese contexto y cite las fuentes.

Si quieres practicar, junta todo lo del post: instala Ollama, indexa los Markdown de tus apuntes con BM25 y embeddings, añade el re-ranker y monta un endpoint `POST /preguntar` con [FastAPI]({% post_url 2026-11-11-fastapi-django-flask %}) que devuelva la respuesta y las fuentes. Tendrás tu propio asistente que responde sobre tus apuntes, ejecutándose entero en tu ordenador.
