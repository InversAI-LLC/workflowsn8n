# Vigilancia CRM Capilea

> **Cliente:** Capilea (sucursal Belgrano)  
> **Nombre en n8n:** `Vigilancia CRM Capilea`  
> **Archivo:** [`vigilancia-crm-capilea.json`](./vigilancia-crm-capilea.json) · **Nodos:** 40 · **Etiquetas:** Belgrano · **Activo al exportar:** sí

## Objetivo

Mantener los calendarios de GHL como **espejo de la agenda de Skyia** para los próximos 30 días (meso, PRP, revisaciones, controles y dermo). Crea las citas que faltan (y el contacto si no existe), marca como *Ocupado Skyia* los horarios tomados, elimina bloqueos obsoletos y actualiza estados (cancelado, no-show, asistido).

## Disparador

Schedule Trigger por intervalo de minutos (valor por defecto del nodo).

## Funcionamiento paso a paso

### 1. Lectura de datos

- `Get URL Skyia` + `Get Free Slots CRM Capilea` traen los turnos libres de Skyia (hoy a +30 días, bloques de 15 min).
- `Get Events GHL ...` leen los eventos de cada calendario en GHL.
- `Get Info ... CRM Capilea` traen los turnos ocupados de Skyia por recurso, y `Get_Skyia_Events_Details` los une.

### 2. Cerebro de sincronización

- `Cerebro Sincronizacion` recorre día por día y calendario por calendario y genera una lista de acciones: `delete` (bloqueos inválidos), `updateStatus` (cancelled / noshow / showed) y `create` (citas o bloqueos nuevos).
- `Switch` reparte cada acción a su rama.

### 3. Ejecución en GHL

- `Borrar cita GHL` y `Update Status` aplican borrados y cambios de estado.
- `Contacto vs Ocupado`: si es un bloqueo → `Set Ocupado Skyia`; si es un paciente → se normaliza el teléfono (`Estandarizacion Numeros Argentinos`), se busca el contacto y se crea si no existe (manejando el error de contacto duplicado).
- Se actualiza `especialidad_a_tratar` y se crea la cita en GHL (`Crear cita en GHL`, `2`, `3`).

## Integraciones

- GoHighLevel (LeadConnector API v2)
- Skyia API
- OpenAI (nodo `Analyze image` suelto, sin uso)

## Credenciales que hay que asignar al importar

| Tipo | Nombre original en n8n |
|---|---|
| Header Auth (`httpHeaderAuth`) | Auth Belgrano |
| OpenAI (`openAiApi`) | OpenAi account 2 |

## Valores a reemplazar

Los siguientes secretos estaban escritos directamente en los nodos y se reemplazaron por marcadores antes de subirlos al repositorio. Busca el marcador en el flujo (Ctrl+F en el JSON o revisando los nodos HTTP/Code) y coloca el valor real:

- `REEMPLAZAR_SKYIA_API_KEY`
- `REEMPLAZAR_TOKEN_SKYIA`

## Configuración al importar

- Reemplazar `REEMPLAZAR_SKYIA_API_KEY` y `REEMPLAZAR_TOKEN_SKYIA`.
- Credencial *Header Auth* de GHL (`Auth Belgrano`).
- Los IDs de calendario de GHL y de recurso de Skyia están en `calendarConfig` dentro de `Cerebro Sincronizacion`.
- El contacto usado para los bloqueos *Ocupado Skyia* (`OlMibXDKwBTqvxrwHRhu`) está fijo en `Set Ocupado Skyia`.

## Observaciones

- Nodos sin conectar (restos de pruebas): `Get Info MESOTERAPIA CRM Capilea1` (fechas fijas de mayo 2026), `Validacion contacto1`, `Analyze image`, `Notificar falla de sincronización`.
- La salida *Contactos pasados* de `Switch1` no está conectada a propósito: las citas en el pasado se ignoran.

## Flujos relacionados

- Vigilancia CRM Capilea Cirugias
- Calendarios Capilea

---
[⬅ Volver al índice](../../../README.md)
