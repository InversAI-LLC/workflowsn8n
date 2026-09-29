# Recordatorios Ausentes

> **Cliente:** Capilea (sucursal Belgrano)  
> **Nombre en n8n:** `Recordatorios Ausentes`  
> **Archivo:** [`recordatorios-ausentes.json`](./recordatorios-ausentes.json) · **Nodos:** 18 · **Etiquetas:** Belgrano · **Activo al exportar:** sí

## Objetivo

Detectar al final del día los pacientes que **no asistieron** a su cita, cruzando los eventos de GHL con la agenda de Skyia, para que GHL ejecute el seguimiento de ausentes.

## Disparador

Schedule Trigger diario a las 23:00.

## Funcionamiento paso a paso

### 1. Rango del día

- `Get Timestamp` calcula el rango del día actual (hora de Argentina) en formatos GHL y Skyia.

### 2. Eventos en GHL

- `Get Events GHL ...` lee los calendarios de meso, revisaciones, dermo, PRP y controles.

### 3. Turnos en Skyia

- Los nodos `Get Info ... CRM Capilea` leen la agenda de Skyia por recurso (ausentes, cancelados, controles, dermo, revisaciones, mesoterapia, PRP) y `Get_Skyia_Events_Details` los une.

### 4. Cruce y envío

- `Check Ausentes y Asistidos` compara por fecha, hora y coincidencia de nombre (por palabras, ignorando términos genéricos) y marca cada cita como AUSENTE o ASISTIDO.
- `If3` deja pasar solo los AUSENTE y `HTTP Request` los envía al *Inbound Webhook* de GHL (lotes de 2).

## Integraciones

- GoHighLevel (LeadConnector API v2)
- Skyia API

## Credenciales que hay que asignar al importar

| Tipo | Nombre original en n8n |
|---|---|
| Header Auth (`httpHeaderAuth`) | Auth Belgrano |

## Valores a reemplazar

Los siguientes secretos estaban escritos directamente en los nodos y se reemplazaron por marcadores antes de subirlos al repositorio. Busca el marcador en el flujo (Ctrl+F en el JSON o revisando los nodos HTTP/Code) y coloca el valor real:

- `REEMPLAZAR_SKYIA_API_KEY`
- `REEMPLAZAR_TOKEN_SKYIA`

## Configuración al importar

- Reemplazar `REEMPLAZAR_SKYIA_API_KEY` (en las URLs) y `REEMPLAZAR_TOKEN_SKYIA` (header Authorization).
- Credencial *Header Auth* de GHL (`Auth Belgrano`).
- Revisar `locationId`, `calendarId` y la URL del *webhook-trigger* de GHL.

## Observaciones

- Tiene configurado un *Error Workflow* (`2nO4EfS90U7HC6hG`) que no forma parte de este repositorio.

## Flujos relacionados

- Recordatorios Capilea
- Vigilancia CRM Capilea

---
[⬅ Volver al índice](../../../README.md)
