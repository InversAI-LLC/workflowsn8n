# Agente Servicios Tecnicos Speed Up

> **Cliente:** Speed Up (taller / servicio técnico automotriz)  
> **Nombre en n8n:** `Agente Servicios Tecnicos Speed Up`  
> **Archivo:** [`servicios-tecnicos-speed-up.json`](./servicios-tecnicos-speed-up.json) · **Nodos:** 14 · **Etiquetas:** Beta · **Activo al exportar:** sí

## Objetivo

Medir cuánto tiempo pasa cada vehículo en cada etapa del pipeline de servicio de GHL. Cada cambio de etapa se registra en Google Sheets; al finalizar el servicio se calcula el tiempo total, se notifica a un grupo de Telegram y se limpian los campos del contacto.

## Disparador

Webhook POST `speedUpRegistroVehicular` llamado desde GHL al cambiar de etapa en el pipeline.

## Funcionamiento paso a paso

### 1. Registro de la etapa

- `Date & Time` calcula las horas entre `hora_inicio_step` y `hora_fin_step`; `TiempoHoras` redondea a 0,05 h y `Code` lo formatea en horas y minutos.
- `Append row in sheet` agrega una fila con datos del cliente, vehículo, etapa y duración.

### 2. Cierre del servicio

- `Finalizó el servicio?` revisa `IsFinalized`. Si es verdadero se calcula el tiempo total entre inicio y fin del servicio (`Date & Time1`, `Tiempo de todo el servicio`).
- `Get row(s) in sheet` + `Quitar valores de Pausa` restan del total las duraciones registradas y `Update row in sheet` guarda el resultado.
- `Send a text message` envía el resumen a Telegram y `Resetear CF en CRM` limpia los campos del vehículo en GHL.

## Integraciones

- GoHighLevel (LeadConnector API v2)
- Google Sheets
- Telegram

## Credenciales que hay que asignar al importar

| Tipo | Nombre original en n8n |
|---|---|
| Google Sheets OAuth2 (`googleSheetsOAuth2Api`) | Google Sheets account |
| Header Auth (`httpHeaderAuth`) | Private Integration GHL |
| Telegram (`telegramApi`) | Telegram account Inverbot AI |

## Valores a reemplazar

No hay secretos escritos en los nodos: todo se maneja con credenciales de n8n.

## Configuración al importar

- Credenciales: Google Sheets OAuth2, Telegram (bot), GHL Private Integration (Header Auth).
- Cambiar la URL de la hoja de cálculo y el `chatId` del grupo de Telegram si se usa en otra cuenta.

## Observaciones

- `Get row(s) in sheet` no filtra por vehículo/transacción: lee todas las filas de la hoja. Conviene filtrar por `Contact_ID` o `Transaction_ID` para que el cálculo sea solo del servicio actual.

---
[⬅ Volver al índice](../../../README.md)
