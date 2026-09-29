# Agente de Texto base

> **Cliente:** Plantilla interna (derivada de MasSeguro)  
> **Nombre en n8n:** `Agente de Texto base`  
> **Archivo:** [`agente-de-texto-base.json`](./agente-de-texto-base.json) · **Nodos:** 55 · **Etiquetas:** Beta · **Activo al exportar:** no

## Objetivo

Esqueleto reutilizable para nuevos agentes de texto sobre GHL. Contiene las piezas comunes de los agentes de producción: agrupación de mensajes en Redis, corrección de canal, procesamiento de adjuntos, detector de reinicio y un agente principal con memoria en MongoDB y RAG. El prompt actual es el de *Martina* (+Seguro Travel) como ejemplo.

## Disparador

Webhook POST (path heredado `MasSeguroMegaBot`; cambiarlo al reutilizar la plantilla).

## Funcionamiento paso a paso

### 1. Recepción y buffer

- Mensajes agrupados en Redis con espera de 20 s; solo la última ejecución continúa.

### 2. Normalización y adjuntos

- Corrección de canal (SMS/IG/FB) y conversión de audio, imagen y documentos a texto.

### 3. Reinicio

- `AI Agente de reseteo1` detecta la intención *reset*: borra la memoria en MongoDB, confirma al usuario y limpia campos en GHL.

### 4. Agente principal

- `Cotizacion - AI Agente de recopilacion de info` responde con memoria MongoDB y RAG (MongoDB Atlas + embeddings OpenAI). Aquí se conecta el envío de la respuesta.

## Integraciones

- GoHighLevel (LeadConnector API v2)
- Redis
- MongoDB Atlas
- OpenAI
- Google Gemini

## Credenciales que hay que asignar al importar

| Tipo | Nombre original en n8n |
|---|---|
| Google Gemini (PaLM) (`googlePalmApi`) | Google Gemini(PaLM) Api account |
| Header Auth (`httpHeaderAuth`) | Private Integration GHL v2 Invers |
| MongoDB (`mongoDb`) | MongoDB account ChatMemory, MongoDB account WebScrapper |
| OpenAI (`openAiApi`) | OpenAi account 2 |
| Redis (`redis`) | Redis account |

## Valores a reemplazar

Los siguientes secretos estaban escritos directamente en los nodos y se reemplazaron por marcadores antes de subirlos al repositorio. Busca el marcador en el flujo (Ctrl+F en el JSON o revisando los nodos HTTP/Code) y coloca el valor real:

- `REEMPLAZAR_SKYIA_API_KEY`
- `REEMPLAZAR_TOKEN_SKYIA`

## Configuración al importar

- Cambiar el path del webhook, el prompt del agente y la base vectorial al crear un agente nuevo.
- Credenciales: GHL Private Integration v2, Redis, MongoDB, OpenAI, Google Gemini.

## Observaciones

- El agente principal no tiene nodo de salida: falta conectar el envío de la respuesta (ver `Agente actual MasSeguro`).
- `MongoDB Chat Memory` usa `$('Webhook Meso')` en la *session key*, un nodo que no existe en este flujo; debe apuntar a `Webhook`.
- `Get Free Slots CRM Capilea1` es un nodo suelto copiado de Capilea (se sanitizó su token).

## Flujos relacionados

- Agente actual MasSeguro

---
[⬅ Volver al índice](../../../README.md)
