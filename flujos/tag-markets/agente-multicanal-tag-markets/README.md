# Agente Multicanal Tag Markets

> **Cliente:** TAG Markets (broker)  
> **Nombre en n8n:** `Agente Multicanal Tag Markets`  
> **Archivo:** [`agente-multicanal-tag-markets.json`](./agente-multicanal-tag-markets.json) · **Nodos:** 144 · **Etiquetas:** v1 · **Activo al exportar:** no

## Objetivo

Agente de soporte multicanal **Mikea** para el broker **TAG Markets**. Un agente principal (nivel 0) saluda, pide nombre, email e idioma y muestra el menú. Según la opción, la conversación pasa a un **agente especialista** por nivel: 1 Broker, 2 Registro/KYC, 3 Cuentas Pro/Zero/Islámicas, 4 Amplify X12, 5 Embajadores/Referidos/Token, 6 Depósitos y Retiros (6.1 / 6.2), 7 Copy Trading, 8 Cuentas promocionales. El nivel actual se guarda en el campo `current_level` de GHL.

## Disparador

Webhook POST llamado desde GHL con cada mensaje entrante. Incluye además un disparador manual para ejecutar el sub-flujo **RAG**.

## Funcionamiento paso a paso

### Parte 1: Recepción

- Buffer de mensajes en Redis (20 s), corrección de canal y procesamiento de adjuntos (audio, imagen, documentos).

### Parte 2: Agentes

- Detector de intención para *reiniciar* (borra memoria y campos en GHL).
- `If2`: si `current_level` es 0 o vacío responde el **Agente Principal**; si no, `Switch2` envía al agente especialista del nivel.
- Cada agente responde un JSON `{action, responseText, values, current_level}`.

### Parte 3: Envío

- `JSON final` actualiza `current_level` y los datos del cliente en GHL; si `action = escalate` se aplica la etiqueta *bot_off*.
- `Switch` detecta respuestas de menú conocidas y usa textos fijos (`Code Opcion ...`); el resto se divide en hasta 3 partes y se envían por GHL con 10 s de pausa.

## Integraciones

- GoHighLevel (LeadConnector API v2)
- Redis
- MongoDB Atlas (memoria + vector store)
- OpenAI
- Sub-flujos: RAG y *Envio de audios*

## Credenciales que hay que asignar al importar

| Tipo | Nombre original en n8n |
|---|---|
| Header Auth (`httpHeaderAuth`) | Private Integration GHL v2 |
| MongoDB (`mongoDb`) | MongoDB account, MongoDB account ChatMemory |
| OpenAI (`openAiApi`) | OpenAi account |
| Redis (`redis`) | Redis account |

## Valores a reemplazar

No hay secretos escritos en los nodos: todo se maneja con credenciales de n8n.

## Configuración al importar

- Credenciales: GHL Private Integration v2, Redis, MongoDB, OpenAI.
- El nodo `Call RAG` apunta al workflow **RAG** de este repositorio: tras importarlo, seleccionarlo de nuevo en el nodo (el ID cambia).
- `Call Envio de audios` apunta a un workflow que no está en el repositorio.

## Observaciones

- Ya contaba con notas en el canvas; se añadió la ficha del flujo.
- Ver la versión V2 para las diferencias.

## Flujos relacionados

- V2 Agente Multicanal Tag Markets
- RAG

---
[⬅ Volver al índice](../../../README.md)
