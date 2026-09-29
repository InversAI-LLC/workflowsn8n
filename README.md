# Flujos n8n · InversAI

Repositorio con los flujos (workflows) de **n8n** desarrollados por Invers para sus clientes. Cada flujo está exportado en formato JSON y viene con su propia documentación, tanto dentro del flujo (notas en el canvas) como en un `README.md` en su carpeta.

---

## Contenido

| Cliente | Flujo | Qué hace | Nodos |
|---|---|---|---|
| **Capilea** | [Calendarios Capilea](flujos/capilea/calendarios-capilea/) | Recibe las citas agendadas en los calendarios de GoHighLevel, valida la disponibilidad en Skyia y crea el paciente y el turno en Skyia. | 95 |
| **Capilea** | [Recordatorios Capilea](flujos/capilea/recordatorios-capilea/) | Dos veces al día arma los recordatorios de las citas del día siguiente: un resumen para el equipo y un aviso por paciente. | 31 |
| **Capilea** | [Recordatorios Ausentes](flujos/capilea/recordatorios-ausentes/) | Cada noche cruza las citas del día en GHL con Skyia y envía a GHL los pacientes que no asistieron. | 18 |
| **Capilea** | [Vigilancia CRM Capilea](flujos/capilea/vigilancia-crm-capilea/) | Sincronización periódica Skyia → GHL de los próximos 30 días (crea, bloquea, borra y actualiza estados de citas). | 40 |
| **Capilea** | [Vigilancia CRM Capilea Cirugias](flujos/capilea/vigilancia-crm-capilea-cirugias/) | La misma sincronización, exclusiva para el calendario de quirófano. | 33 |
| **+Seguro Travel** | [Agente actual MasSeguro](flujos/masseguro/agente-masseguro/) | Agente de IA "Martina": informa, cotiza seguros de viaje, genera PDF y link de pago de Stripe, valida el pago y registra la venta. | 190 |
| **TAG Markets** | [Agente Multicanal Tag Markets](flujos/tag-markets/agente-multicanal-tag-markets/) | Agente de soporte "Mikea" con menú por niveles y agentes especialistas (v1). | 144 |
| **TAG Markets** | [V2 Agente Multicanal Tag Markets](flujos/tag-markets/agente-multicanal-tag-markets-v2/) | Segunda versión de "Mikea" con prompts ajustados. | 143 |
| **Speed Up** | [Servicios Técnicos Speed Up](flujos/speed-up/servicios-tecnicos-speed-up/) | Registra en Google Sheets el tiempo de cada etapa del servicio de un vehículo y avisa por Telegram al terminar. | 14 |
| Plantillas | [Agente de Texto base](flujos/plantillas-y-utilidades/agente-de-texto-base/) | Plantilla de agente de texto: buffer de mensajes, adjuntos, detector de reinicio y agente con RAG. | 55 |
| Plantillas | [Flujo_AgenteMulticanal](flujos/plantillas-y-utilidades/agente-multicanal-rag/) | Agente de soporte "Camila" con RAG y respuesta en texto o audio (ElevenLabs). | 48 |
| Utilidades | [RAG](flujos/plantillas-y-utilidades/rag-ingesta-google-drive/) | Sub-flujo que indexa carpetas de Google Drive como embeddings en MongoDB Atlas. | 40 |

### Estructura

```
flujos/
├── capilea/
├── masseguro/
├── tag-markets/
├── speed-up/
└── plantillas-y-utilidades/
    └── <flujo>/
        ├── <flujo>.json   ← archivo que se importa en n8n
        └── README.md      ← documentación del flujo
```

---

## Cómo cargar un flujo en n8n

### 1. Importar el archivo

Elige una de estas opciones:

**A. Desde archivo (la más común)**

1. Descarga el `.json` del flujo (en GitHub: abre el archivo → botón **Download raw file**).
2. En n8n, crea un workflow nuevo (**+ → Workflow**).
3. Abre el menú **⋯** (arriba a la derecha) → **Import from File…** y selecciona el `.json`.

**B. Copiar y pegar**

1. En GitHub abre el `.json`, pulsa **Raw** y copia todo el contenido.
2. En un workflow vacío de n8n, haz clic en el canvas y pega con **Ctrl + V** (Cmd + V en Mac).

**C. Por URL** *(solo si el repositorio es público)*

En el menú **⋯ → Import from URL…** pega la URL *raw* del archivo, por ejemplo:
`https://raw.githubusercontent.com/InversAI-LLC/workflowsn8n/main/flujos/speed-up/servicios-tecnicos-speed-up/servicios-tecnicos-speed-up.json`

**D. Por línea de comandos** *(n8n self-hosted)*

```bash
n8n import:workflow --input=flujos/capilea/recordatorios-capilea/recordatorios-capilea.json
```

### 2. Asignar las credenciales

Las credenciales **no viajan** en el JSON: el flujo solo guarda el nombre de la credencial que usaba. Después de importar:

1. Abre cada nodo marcado con ⚠️ y selecciona (o crea) la credencial correspondiente.
2. La lista de credenciales que necesita cada flujo está en su `README.md` y en la nota **📋 Ficha del flujo** dentro del canvas.

### 3. Reemplazar los valores retirados

Algunos flujos tenían tokens escritos directamente en los nodos. Antes de subirlos se reemplazaron por marcadores que empiezan con `REEMPLAZAR_`:

| Marcador | Qué va ahí |
|---|---|
| `REEMPLAZAR_SKYIA_API_KEY` | API key de Skyia (parámetro `x` de las URLs y de los nodos Code) |
| `REEMPLAZAR_TOKEN_SKYIA` | Token del header `Authorization: Bearer …` de Skyia |
| `REEMPLAZAR_TOKEN_GHL_PRIVATE_INTEGRATION` | Token de *Private Integration* de GoHighLevel |

> 💡 Recomendado: en lugar de volver a pegar el token en el nodo, crea una credencial **Header Auth** en n8n y selecciónala en el nodo HTTP Request. Así el secreto no queda en el JSON la próxima vez que se exporte.

### 4. Revisar la configuración propia de cada cliente

- **IDs externos:** `locationId` y `calendarId` de GHL, carpetas de Google Drive, hojas de cálculo, chat de Telegram, etc.
- **Sub-flujos:** los nodos *Execute Workflow* (por ejemplo `Call RAG`) apuntan al ID del flujo en la instancia original. Importa primero el sub-flujo y vuelve a seleccionarlo en el nodo.
- **Error Workflow:** algunos flujos de Capilea tienen configurado un flujo de errores en *Settings*. Reasígnalo o quítalo.
- **Webhooks:** la URL de producción depende de tu instancia (`https://<tu-n8n>/webhook/<path>`). Actualízala en GHL o en el sistema que llama al webhook.

### 5. Probar y activar

1. Ejecuta el flujo con **Execute workflow** (o **Listen for test event** en los webhooks) y revisa cada nodo.
2. Cuando todo funcione, activa el flujo con el interruptor **Active** (al importarlos, los flujos quedan desactivados).

---

## Documentación dentro de cada flujo

Al abrir un flujo en n8n verás notas (sticky notes) con el prefijo **Doc -**:

- **📋 Ficha del flujo** (roja): cliente, objetivo, disparador, integraciones, credenciales y valores a reemplazar.
- **Descripción del flujo**: el funcionamiento paso a paso y las observaciones.
- **Secciones** (fondos de color): agrupan los nodos por etapa.

Las notas que ya existían en los flujos se conservaron tal cual.

---

## Buenas prácticas para agregar o actualizar flujos

1. Exporta desde n8n: menú **⋯ → Download**.
2. Antes de hacer commit, busca en el JSON `Bearer`, `api_key`, `token`, `x=`, `pit-` y reemplaza cualquier secreto por un marcador `REEMPLAZAR_…`.
3. Guarda el archivo en `flujos/<cliente>/<nombre-del-flujo>/` junto con su `README.md`.
4. Agrega el flujo a la tabla de **Contenido** de este README.
