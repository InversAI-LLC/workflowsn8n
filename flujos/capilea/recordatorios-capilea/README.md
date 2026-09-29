# Recordatorios Capilea

> **Cliente:** Capilea (sucursal Belgrano)  
> **Nombre en n8n:** `Recordatorios Capilea`  
> **Archivo:** [`recordatorios-capilea.json`](./recordatorios-capilea.json) · **Nodos:** 31 · **Etiquetas:** Belgrano · **Activo al exportar:** sí

## Objetivo

Enviar recordatorios de las citas **del día siguiente**, separados en bloque *mañana* (09:00–14:15) y bloque *tarde*. Tiene dos ramas: una arma un **resumen de agenda** para el equipo interno y la otra dispara un workflow de GHL **por cada paciente** con cita confirmada.

## Disparador

Dos Schedule Triggers: 11:30 y 17:00 (rama resumen) y 11:35 y 17:00 (rama pacientes).

## Funcionamiento paso a paso

### Rama A: Resumen para el equipo (arriba)

- `Get Timestamp` calcula el rango de fechas de mañana en formato GHL y Skyia.
- `If2` filtra el día de la semana (intención: no enviar viernes ni sábado).
- `Get Events GHL ...` lee los eventos de los calendarios meso, revisaciones, dermo, PRP y controles.
- `If` elige bloque mañana (ejecución de las 11) o tarde. `Crear recordatorio ...` filtra por fecha/rango, excluye *Bloqueos*, deduplica y arma un texto con la lista de citas.
- `Stringify JSON` + `current_task seteo` guardan el resumen en el campo `barrido_recordatorios` de un contacto interno y `Mensaje Angelo` dispara el workflow de GHL que lo envía.

### Rama B: Recordatorio por paciente (abajo)

- Misma lectura de calendarios (`... 1`).
- `Crear recordatorio mañana1 / tarde1` se queda solo con citas **confirmadas**, excluye bloqueos y *Ocupado Skyia*, normaliza el nombre y detecta la especialidad.
- `HTTP Request` / `HTTP Request1` envían cada paciente al webhook de GHL que dispara el recordatorio (lotes de 2 cada 5 s).

## Integraciones

- GoHighLevel (LeadConnector API v2) – calendarios, contactos y webhooks entrantes (Inbound Webhook)

## Credenciales que hay que asignar al importar

| Tipo | Nombre original en n8n |
|---|---|
| Header Auth (`httpHeaderAuth`) | Auth Belgrano |

## Valores a reemplazar

No hay secretos escritos en los nodos: todo se maneja con credenciales de n8n.

## Configuración al importar

- Credencial *Header Auth* de GHL (`Auth Belgrano`).
- Revisar `locationId`, `calendarId`, el contacto interno (`ca0hQ9pCYPnxuZexRY22`) y las URLs de *webhook-trigger* de GHL.
- Los rangos horarios (`horaMin` / `horaMax`) se ajustan dentro de cada nodo `Crear recordatorio ...`.

## Observaciones

- `If2` / `If3` usan combinador **OR** con dos condiciones *distinto de* (viernes, sábado); esa condición siempre es verdadera. Si la intención es excluir ambos días, debe ser **AND**.
- Las fechas se calculan con `new Date()` (zona horaria del servidor). `Recordatorios Ausentes` ya usa la zona `America/Argentina/Buenos_Aires`; conviene unificar.
- `Get Events GHL cirugias` no está conectado y el código referencia un nodo `Get Events GHL clientes` que no existe (se ignora por el try/catch).

## Flujos relacionados

- Recordatorios Ausentes
- Vigilancia CRM Capilea

---
[⬅ Volver al índice](../../../README.md)
