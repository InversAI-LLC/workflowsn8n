# Vigilancia CRM Capilea Cirugias

> **Cliente:** Capilea (sucursal Belgrano)  
> **Nombre en n8n:** `Vigilancia CRM Capilea Cirugias`  
> **Archivo:** [`vigilancia-crm-capilea-cirugias.json`](./vigilancia-crm-capilea-cirugias.json) · **Nodos:** 33 · **Etiquetas:** Belgrano · **Activo al exportar:** sí

## Objetivo

Misma lógica que **Vigilancia CRM Capilea**, pero solo para el **calendario de quirófano** (recurso Skyia 6). Usa estados propios de cirugía: se crea la cita cuando el turno está en *ESTUDIOS REALIZADOS* o *REALIZADO*.

## Disparador

Schedule Trigger por intervalo de minutos (valor por defecto del nodo).

## Funcionamiento paso a paso

### 1. Lectura de datos

- Turnos libres de Skyia (`Get URL Skyia`, `Get Free Slots CRM Capilea`), eventos del calendario de quirófano en GHL (`Get Events GHL quirofano`) y agenda de quirófano en Skyia (`Get Info QUIROFANO CRM Capilea`).

### 2. Cerebro de sincronización

- `Cerebro Sincronizacion` (configuración exclusiva de quirófano) genera las acciones `delete`, `updateStatus` y `create`.

### 3. Ejecución en GHL

- Idéntica a Vigilancia CRM: borrado, cambio de estado, bloqueos *Ocupado Skyia*, búsqueda/creación de contacto y creación de la cita.

## Integraciones

- GoHighLevel (LeadConnector API v2)
- Skyia API

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
- El calendario de GHL de quirófano está en `calendarConfig` (`Cerebro Sincronizacion`).

## Observaciones

- Nodos sin conectar: `Sincronizacion Bloques`, `Comparador_bloques`, `Sincronizar info bloques` (versión anterior por bloques), `Get Info REVISACIONES CRM Capilea`, `Get Info MESOTERAPIA CRM Capilea1`, `Validacion contacto1`, `Analyze image`, `Notificar falla de sincronización`.

## Flujos relacionados

- Vigilancia CRM Capilea

---
[⬅ Volver al índice](../../../README.md)
