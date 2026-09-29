# Agente actual MasSeguro

> **Cliente:** +Seguro Travel (MasSeguro)  
> **Nombre en n8n:** `Agente actual MasSeguro`  
> **Archivo:** [`agente-masseguro.json`](./agente-masseguro.json) · **Nodos:** 190 · **Etiquetas:** — · **Activo al exportar:** no

## Objetivo

Agente de IA multicanal (WhatsApp, Instagram, Facebook vía GHL) llamado **Martina** para **+Seguro Travel**. Cubre todo el embudo: resolver dudas con RAG, recopilar datos de viaje, cotizar contra el sistema interno, enviar la cotización en PDF, generar el link de pago de Stripe, validar el pago, recopilar datos de titular y acompañantes y registrar la venta con su contrato. El estado de la conversación se guarda en el campo `current_task` del contacto en GHL.

## Disparador

Webhook POST `MasSeguroMegaBot` llamado desde un workflow de GHL con cada mensaje entrante.

## Funcionamiento paso a paso

### 1. Recepción y agrupación de mensajes

- Cada mensaje se guarda en Redis (`contact_<id>`) y se esperan 20 s (`Espera posible otro mensaje1`). Solo la última ejecución continúa, así se responden juntos los mensajes enviados seguidos.

### 2. Normalización y adjuntos

- `Switch3` corrige el canal de respuesta (WhatsApp→SMS, IG, FB).
- Si hay adjunto se descarga y procesa: audio → transcripción (OpenAI), imagen → descripción (OpenAI), documentos PDF/XLS/XLSX → texto. Todo converge en un solo mensaje (`FinalMessage`).

### 3. Reinicio de conversación

- `AI Agente de reseteo1` detecta si el usuario quiere reiniciar. Si es así se borra la memoria en MongoDB, se confirma al usuario y se limpian los campos del CRM.

### 4. Enrutamiento por estado (`Switch Central`)

- Según `current_task`: `awaiting_plan_selection` → agente de ventas; `awaiting_payment_validation` → validación de pago; `awaiting_holder_final_data` / `awaiting_companion_data` → contrato; resto → `AI Agente de redireccion`.
- `AI Agente de redireccion` clasifica la intención: emergencia, información o cotización, y deriva al agente correspondiente (todos con memoria MongoDB y RAG en MongoDB Atlas).

### 5. Cotización

- Cuando el agente tiene todos los datos (`send_quote`) se calculan los días de viaje, se obtiene el código de país y se llama a la API interna (`/api/quotation/products`). Con precios y edades (documento de Google Drive) se arma el JSON final y el agente de ventas presenta los planes.
- Al confirmar un plan se genera el PDF (`/api/quotation`), se sube a Google Drive, se crea el link de pago en Stripe y se envía al cliente (`current_task = awaiting_payment_validation`).

### 6. Pago, contrato y venta

- `AI Agente validacion pago` confirma el pago; luego `AI Agente Contrato de Venta` recopila datos de titular y acompañantes y se registra la venta (`/api/sale`). El contrato se sube a Drive y se envía al cliente.

### 7. Respuesta y seguimiento

- Las respuestas se dividen en partes (`Code Split`) y se envían por la API de conversaciones de GHL.
- Se actualizan campos de seguimiento (`session_expiration_time`, `seguimiento_normal`, `seguimiento_doce_horas`, `seguimiento_casi_un_dia`). Las escalaciones a humano disparan el webhook de GHL que pone la etiqueta *bot_off*.

## Integraciones

- GoHighLevel (LeadConnector API v2)
- Redis
- MongoDB Atlas (memoria de chat + vector store)
- OpenAI (GPT, embeddings, Whisper, visión)
- Google Gemini
- Google Drive
- Stripe
- API interna de cotización/venta (`69.164.198.106:3031`)

## Credenciales que hay que asignar al importar

| Tipo | Nombre original en n8n |
|---|---|
| Google Drive OAuth2 (`googleDriveOAuth2Api`) | Google Drive account |
| Google Gemini (PaLM) (`googlePalmApi`) | MasSeguro Google Gemini(PaLM) Api account |
| Header Auth (`httpHeaderAuth`) | Private Integration GHL v2, Stripe, Stripe Test |
| MongoDB (`mongoDb`) | MongoDB account AgenteTexto, MongoDB account ChatMemory |
| OpenAI (`openAiApi`) | OpenAi account MasSeguro |
| Redis (`redis`) | Redis account |

## Valores a reemplazar

No hay secretos escritos en los nodos: todo se maneja con credenciales de n8n.

## Configuración al importar

- Credenciales: GHL Private Integration v2 (Header Auth), Stripe (Header Auth), Redis, MongoDB (memoria y agente), OpenAI, Google Gemini, Google Drive.
- Actualizar las URL de la API interna si cambia el servidor.
- Revisar IDs de carpetas de Google Drive y las URLs de *webhook-trigger* de GHL (escalamiento / bot_off).
- Los prompts de cada agente están en el campo *System Message* de cada nodo `AI Agente ...`.

## Observaciones

- Es el flujo más grande del repositorio (190 nodos). Por tamaño, la documentación en el canvas se concentra en la ficha, la descripción general y las primeras etapas.
- Hay dos credenciales de Stripe (`Stripe` y `Stripe Test`): confirmar cuál queda activa en producción.

## Flujos relacionados

- Agente de Texto base (plantilla derivada de este flujo)

---
[⬅ Volver al índice](../../../README.md)
