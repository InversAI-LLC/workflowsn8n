# V2 Agente Multicanal Tag Markets

> **Cliente:** TAG Markets (broker)  
> **Nombre en n8n:** `V2 Agente Multicanal Tag Markets`  
> **Archivo:** [`agente-multicanal-tag-markets-v2.json`](./agente-multicanal-tag-markets-v2.json) · **Nodos:** 143 · **Etiquetas:** v1 · **Activo al exportar:** no

## Objetivo

Segunda versión del agente **Mikea** para **TAG Markets**. Misma arquitectura que la versión 1 (agente principal + especialistas por nivel), con estas diferencias:

- Prompts actualizados en el agente principal y en todos los especialistas.
- Detector de reinicio reemplazado (`AI Agent3` en lugar de `AI Agente de reseteo`) y se quitó el nodo `Resetear current_level1`.
- Los modelos de OpenAI usan la credencial `OpenAi account MasSeguro`.
- Path de webhook distinto.

## Disparador

Webhook POST llamado desde GHL con cada mensaje entrante.

## Funcionamiento paso a paso

### Parte 1: Recepción

- Buffer de mensajes en Redis (20 s), corrección de canal y procesamiento de adjuntos.

### Parte 2: Agentes

- Detector de reinicio → agente principal (nivel 0) o especialista según `current_level`.

### Parte 3: Envío

- Actualización de GHL, escalamiento (*bot_off*), respuestas fijas de menú y envío dividido en hasta 3 mensajes.

## Integraciones

- GoHighLevel (LeadConnector API v2)
- Redis
- MongoDB Atlas
- OpenAI
- Sub-flujos: RAG y *Envio de audios*

## Credenciales que hay que asignar al importar

| Tipo | Nombre original en n8n |
|---|---|
| Header Auth (`httpHeaderAuth`) | Private Integration GHL v2 |
| MongoDB (`mongoDb`) | MongoDB account, MongoDB account ChatMemory |
| OpenAI (`openAiApi`) | OpenAi account, OpenAi account MasSeguro |
| Redis (`redis`) | Redis account |

## Valores a reemplazar

No hay secretos escritos en los nodos: todo se maneja con credenciales de n8n.

## Configuración al importar

- Credenciales: GHL Private Integration v2, Redis, MongoDB, OpenAI.
- Reasignar el sub-flujo en `Call RAG` después de importar.

## Observaciones

- Ya contaba con notas en el canvas; se añadió la ficha del flujo.

## Flujos relacionados

- Agente Multicanal Tag Markets
- RAG

---
[⬅ Volver al índice](../../../README.md)
