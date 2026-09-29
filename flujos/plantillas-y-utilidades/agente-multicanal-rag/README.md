# Flujo_AgenteMulticanal

> **Cliente:** Invers (plantilla / alternativa)  
> **Nombre en n8n:** `Flujo_AgenteMulticanal`  
> **Archivo:** [`agente-multicanal-rag.json`](./agente-multicanal-rag.json) · **Nodos:** 48 · **Etiquetas:** Alternativas · **Activo al exportar:** no

## Objetivo

Versión inicial / alternativa del agente multicanal. Recibe el mensaje desde GHL, procesa texto, audio, imagen o documento, busca contexto en la base vectorial (RAG) y responde como **Camila**, asistente de soporte de Invers. Si el usuario envía audio, responde con audio generado en ElevenLabs.

## Disparador

Webhook POST llamado desde GHL.

## Funcionamiento paso a paso

### Parte 1: Recepción

- Corrección de canal, detección de adjuntos y conversión a texto.

### Parte 2: RAG + agente

- Búsqueda vectorial en MongoDB Atlas, armado del prompt con contexto y respuesta del `Knowledge Base Agent`.

### Parte 3: Envío

- Texto dividido en 3 partes o audio (ElevenLabs → Google Drive) enviados por la API de conversaciones de GHL; opcionalmente comparte el documento fuente.

## Integraciones

- GoHighLevel (LeadConnector API v2)
- MongoDB Atlas
- OpenAI
- ElevenLabs
- Google Drive

## Credenciales que hay que asignar al importar

| Tipo | Nombre original en n8n |
|---|---|
| ElevenLabs (`elevenLabsApi`) | ElevenLabs Cris account |
| Google Drive OAuth2 (`googleDriveOAuth2Api`) | Google Drive account |
| MongoDB (`mongoDb`) | MongoDB account ChatMemory, MongoDB account DB AgenteSoporte |
| OpenAI (`openAiApi`) | OpenAi account 2 |

## Valores a reemplazar

Los siguientes secretos estaban escritos directamente en los nodos y se reemplazaron por marcadores antes de subirlos al repositorio. Busca el marcador en el flujo (Ctrl+F en el JSON o revisando los nodos HTTP/Code) y coloca el valor real:

- `REEMPLAZAR_TOKEN_GHL_PRIVATE_INTEGRATION`

## Configuración al importar

- Reemplazar `REEMPLAZAR_TOKEN_GHL_PRIVATE_INTEGRATION` en los nodos de envío. Mejor: usar una credencial *Header Auth* de GHL.
- Credenciales: MongoDB, OpenAI, ElevenLabs, Google Drive.

## Observaciones

- Ya contaba con notas en el canvas; se añadió la ficha del flujo.

## Flujos relacionados

- RAG

---
[⬅ Volver al índice](../../../README.md)
