# Calendarios Capilea

> **Cliente:** Capilea (sucursal Belgrano)  
> **Nombre en n8n:** `Calendarios Capilea`  
> **Archivo:** [`calendarios-capilea.json`](./calendarios-capilea.json) · **Nodos:** 95 · **Etiquetas:** Belgrano · **Activo al exportar:** sí

## Objetivo

Sincronizar hacia **Skyia** (sistema de agenda de la clínica) cada cita que se agenda en los calendarios de GoHighLevel. Hay una rama idéntica por especialidad: **Mesoterapia, Dermopigmentación, Revisaciones, PRP y Controles**.

## Disparador

5 Webhooks POST (uno por calendario de GHL): `MesoCapileaCRM`, `CapileaCRMdermopigmentacion`, `CapileaCRMrevisiones`, `PRPCapileaCRM`, `ControlCapileaCRM`.

## Funcionamiento paso a paso

### 1. Recepción de la cita

- GHL llama al webhook de la especialidad cuando se agenda una cita. El nodo `Respond to Webhook` responde de inmediato para que GHL no espere.
- `Construir URL Free Slots Skyia` arma la consulta `getAgendaTurnosLibres` a Skyia para la fecha de la cita.

### 2. Validación de disponibilidad

- `Get Free Slots CRM Capilea` trae los turnos libres de Skyia.
- `Check Disponibilidad Skyia` busca un turno libre con el mismo `idRecurso` (profesional/especialidad, valor fijo por rama) y la misma fecha y hora.
- Si **no** hay turno libre → `Motivo Sin Reserva` decide el mensaje (producto no reconocido o sin disponibilidad).

### 3. Paciente en Skyia

- `Tiene DNI` verifica que el contacto tenga el campo *NUMERO DE IDENTIDAD*. Si no lo tiene, se avisa por WhatsApp al equipo interno.
- `Construir URL Get Paciente` + `Get Paciente Capilea` buscan al paciente por DNI en Skyia.
- Si no existe → `Construir Body Post Paciente` + `Post Paciente Skyia` lo crean (`postPaciente`).

### 4. Reserva del turno

- Los nodos `Code in JavaScript*` arman la llamada `postAgendaEvento` con el `idRecurso` fijo de la rama y el `idProducto` según el texto de la especialidad.
- `Post CRM Capilea*` crea el turno en Skyia.

### 5. Alertas internas

- Los nodos `Msg ...` construyen el texto de alerta (sin DNI, producto no reconocido) y `Enviar WhatsApp ...` lo envía por GHL a un contacto interno fijo.

## Integraciones

- GoHighLevel (webhooks de calendario + API de mensajes)
- Skyia API (`api.cloud.skyia.ar`)

## Credenciales que hay que asignar al importar

| Tipo | Nombre original en n8n |
|---|---|
| Header Auth (`httpHeaderAuth`) | Auth Belgrano, Auth RAMOS |

## Valores a reemplazar

Los siguientes secretos estaban escritos directamente en los nodos y se reemplazaron por marcadores antes de subirlos al repositorio. Busca el marcador en el flujo (Ctrl+F en el JSON o revisando los nodos HTTP/Code) y coloca el valor real:

- `REEMPLAZAR_SKYIA_API_KEY`
- `REEMPLAZAR_TOKEN_SKYIA`

## Configuración al importar

- Reemplazar `REEMPLAZAR_SKYIA_API_KEY` (parámetro `x` en los nodos Code) y `REEMPLAZAR_TOKEN_SKYIA` (header Authorization) por los valores reales. Lo ideal es moverlos a una credencial *Header Auth* de n8n.
- Configurar en GHL la URL de producción de cada webhook (`/webhook/<path>`).
- Revisar el `contactId` interno que recibe las alertas de WhatsApp (`Enviar WhatsApp ...`).
- Los `idRecurso` están fijos por rama (Meso=10, Dermo=13, Revisaciones=5, PRP=7, Control=3) y están marcados en el código como *sin confirmar* contra el catálogo de Skyia Belgrano.

## Observaciones

- Los nodos `Msg Sin Disponibilidad ...` no están conectados a ningún envío, por lo que ese aviso nunca sale. Como `categoriaNecesaria` es fijo, `Motivo Sin Reserva` siempre toma la rama *sin disponibilidad*.
- Tiene configurado un *Error Workflow* (`2nO4EfS90U7HC6hG`) que no forma parte de este repositorio.

## Flujos relacionados

- Vigilancia CRM Capilea (sincronización inversa Skyia → GHL)

---
[⬅ Volver al índice](../../../README.md)
