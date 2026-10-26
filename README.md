# News Reporter — Agente automatizado de noticias con IA #

News Reporter es un agente de generación de contenidos que desarrollé utilizando **n8n, inteligencia artificial generativa y APIs externas**.

Su función es obtener noticias reales de tecnología, resumirlas con un lenguaje informativo y cercano, generar una imagen ilustrativa y publicar el resultado automáticamente en Telegram.

Desarrollé este proyecto como parte de mi formación en automatización e inteligencia artificial, con un objetivo principal: **crear una solución funcional utilizando herramientas gratuitas y sin realizar gastos**.

## Funcionalidades principales

- Obtención de noticias reales desde un RSS en español.
- Conversión y procesamiento de datos XML y JSON.
- Selección de la noticia más reciente.
- Prevención de publicaciones duplicadas.
- Generación de resúmenes mediante Google Gemini.
- Creación automática de prompts visuales en inglés.
- Generación de imágenes mediante Pollinations AI.
- Publicación del contenido en Telegram, incluyendo el enlace a la fuente original.
- Registro de las publicaciones mediante Data Tables de n8n.
- Validación de respuestas de IA y gestión básica de errores.

## Tecnologías utilizadas

| Tecnología                     | Función                                           |
| ------------------------------ | ------------------------------------------------- |
| n8n 2.31                       | Orquestación del workflow                         |
| Docker                         | Ejecución local de n8n                            |
| RSS de El País                 | Fuente de noticias tecnológicas                   |
| Google Gemini 3.5 Flash-Lite   | Generación de resúmenes y prompts                 |
| Pollinations AI / Flux Schnell | Generación de imágenes                            |
| Telegram Bot API               | Canal de publicación                              |
| n8n Data Tables                | Registro y prevención de duplicados               |
| JavaScript                     | Validación de la respuesta estructurada de Gemini |

El proyecto utiliza cuotas gratuitas y créditos promocionales. Su funcionamiento depende de que estos recursos sigan disponibles.

## Arquitectura y funcionamiento

El workflow está compuesto por **15 nodos funcionales**, organizados en seis etapas.

### 1. Obtención de noticias

Utilizo un nodo `HTTP Request` para consultar el RSS de Tecnología de El País.

La respuesta llega en formato XML, por lo que la convierto a JSON y separo las noticias en elementos individuales.

### 2. Preparación de datos

Normalizo las fechas de publicación, ordeno las noticias cronológicamente y selecciono la más reciente.

Después preparo los campos que necesita la inteligencia artificial: título, descripción, enlace, fecha y fuente.

### 3. Control de duplicados

Antes de procesar la noticia, compruebo si su enlace ya está registrado en una Data Table.

Si la noticia fue publicada anteriormente, el workflow se detiene para evitar llamadas innecesarias a los servicios de IA.

### 4. Procesamiento con inteligencia artificial

Utilizo Google Gemini para generar:

- `resumen`: texto informativo y divulgativo en español.
- `prompt_imagen`: instrucciones en inglés para generar una imagen ilustrativa.

Como el RSS proporciona principalmente un título y un extracto, configuré el prompt para impedir que el modelo invente datos o dé a entender que leyó el artículo completo.

Posteriormente, utilizo un nodo `Code` con JavaScript para validar la respuesta JSON y comprobar que contiene los campos obligatorios.

### Generación de imágenes

Envío el prompt visual a Pollinations AI mediante una petición HTTP, utilizando el modelo Flux Schnell.

Mejoré las instrucciones visuales para evitar logotipos, marcas inventadas, carteles y textos visibles que puedan resultar engañosos.

### 6. Publicación y registro

Finalmente, preparo un mensaje que incluye:

- Título de la noticia.
- Resumen generado por Gemini.
- Enlace a la fuente original.
- Imagen ilustrativa generada por IA.

Publico el contenido mediante un bot de Telegram.

Cuando Telegram confirma el envío, registro la noticia en Data Table mediante la operación `Upsert`.

## 📥 Instalación e importación

Para utilizar este proyecto en otra instalación de n8n:

1. Descargar el archivo `News_Reporter_Entrega.json` desde este repositorio.
2. Importar el archivo en una instalación compatible de n8n.
3. Configurar las credenciales de Google Gemini, Pollinations AI y Telegram.
4. Crear una Data Table llamada `news_reporter_publicadas`.
5. Configurar los nodos Data Table para utilizar la nueva tabla.
6. Reemplazar `REEMPLAZAR_CHAT_ID` por el identificador del chat de Telegram de destino.
7. Comprobar la configuración y ejecutar el workflow manualmente.

### Estructura de la Data Table

| Columna               | Tipo      |
| --------------------- | --------- |
| `enlace`              | String    |
| `titulo`              | String    |
| `fecha_publicacion`   | Date/Time |
| `telegram_message_id` | Number    |

**Importante:** el archivo JSON contiene la configuración del workflow, pero no incluye las credenciales secretas ni los registros de la Data Table original.

## Seguridad

Para compartir el proyecto de forma segura:

- No incluí las claves API de los servicios.
- No incluí el token del bot de Telegram.
- Sustituí mi Chat ID por el marcador `REEMPLAZAR_CHAT_ID`.
- Utilicé el sistema de credenciales de n8n para almacenar los secretos.

Cada persona que importe el workflow deberá utilizar sus propias credenciales.

## Pruebas realizadas

Durante el desarrollo comprobé:

- La obtención y el procesamiento correcto de noticias reales desde el RSS.
- La generación de resúmenes y prompts mediante Gemini.
- La validación de respuestas JSON.
- La generación de imágenes con Pollinations AI.
- La publicación real de noticias con imagen y enlace en Telegram.
- El registro de noticias en Data Table y el bloqueo de enlaces ya publicados.

La configuración final de `Upsert` y el almacenamiento del identificador del mensaje están incluidos en el workflow exportado.

## Limitaciones conocidas

- El agente depende de servicios externos y de sus cuotas o créditos gratuitos.
- El RSS puede ofrecer únicamente un extracto de la noticia.
- Actualmente utilizo un disparador manual.
- Solo proceso la noticia más reciente; si ya fue publicada, el workflow se detiene.
- En determinados fallos de confirmación de Telegram podrían producirse publicaciones duplicadas, por lo que sería necesario comprobar manualmente el chat antes de reintentar.

## Mejoras futuras

Me gustaría incorporar un disparador programado, una gestión de errores más avanzada y la posibilidad de seleccionar otras noticias cuando la más reciente ya esté publicada.

También considero interesante ampliar el agente a nuevas fuentes RSS y otros canales de publicación.

## Documentación

La memoria técnica del proyecto explica las decisiones de diseño, los servicios utilizados, las pruebas realizadas y las limitaciones.

Este repositorio contiene el workflow exportado y su documentación. La ejecución original se realiza en una instalación local de n8n mediante Docker.

---

**Proyecto académico desarrollado como parte de mi formación en automatización e inteligencia artificial.**
