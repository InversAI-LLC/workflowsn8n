# RAG

> **Cliente:** Utilidad compartida (usada por Tag Markets)  
> **Nombre en n8n:** `RAG`  
> **Archivo:** [`rag-ingesta-google-drive.json`](./rag-ingesta-google-drive.json) · **Nodos:** 40 · **Etiquetas:** — · **Activo al exportar:** sí

## Objetivo

Cargar la base de conocimiento de un agente. Recorre las carpetas de Google Drive indicadas, extrae el texto de cada archivo (Google Docs, Google Sheets, Excel, PDF e imágenes) y lo guarda como vectores (`text-embedding-3-large`) en MongoDB Atlas con el `doc_id` como metadato.

## Disparador

*Execute Workflow Trigger*: se ejecuta como sub-flujo (por ejemplo desde `Call RAG` en el agente de Tag Markets).

## Funcionamiento paso a paso

### 1. Carpetas y archivos

- `Code URLs1` define el array de URLs de carpetas de Drive a cargar.
- `Search files and folders1` lista los archivos; si no están compartidos, `Share file2` los comparte.

### 2. Extracción por tipo (`Switch`)

- Google Docs → `Get a document1`. Google Sheets → lectura hoja por hoja. Excel (xls/xlsx) → se convierte a Google Sheets en la carpeta *CorpusRAG* o se extrae directo.
- PDF → `Extract from File2`. Imágenes → `Analyze image` (GPT-4o-mini) describe la imagen; todas las imágenes de una carpeta comparten el id de la carpeta.

### 3. Indexación

- `TextConvergence` unifica `texto` + `docId`. `Document Chunker1` (splitter markdown) → `Document Section Loader1` → `MongoDB Vector Store Inserter1` con embeddings de OpenAI.

## Integraciones

- Google Drive / Docs / Sheets
- OpenAI (embeddings y visión)
- MongoDB Atlas Vector Search

## Credenciales que hay que asignar al importar

| Tipo | Nombre original en n8n |
|---|---|
| Google Docs OAuth2 (`googleDocsOAuth2Api`) | Google Docs account |
| Google Drive OAuth2 (`googleDriveOAuth2Api`) | Google Drive account |
| Google OAuth2 (`googleOAuth2Api`) | Google account |
| Google Sheets OAuth2 (`googleSheetsOAuth2Api`) | Google Sheets account |
| MongoDB (`mongoDb`) | MongoDB account |
| OpenAI (`openAiApi`) | OpenAi account |

## Valores a reemplazar

No hay secretos escritos en los nodos: todo se maneja con credenciales de n8n.

## Configuración al importar

- Editar el array de carpetas en `Code URLs1` antes de ejecutar.
- Credenciales: Google Drive, Google Docs, Google Sheets, Google OAuth2, OpenAI, MongoDB.
- El índice vectorial de MongoDB Atlas debe tener la dimensión de `text-embedding-3-large` (3072).

## Observaciones

- No evita duplicados: subir la misma carpeta dos veces duplica los vectores (ver nota *Mejoras* en el flujo de Tag Markets).

## Flujos relacionados

- Agente Multicanal Tag Markets (nodo `Call RAG`)

---
[⬅ Volver al índice](../../../README.md)
