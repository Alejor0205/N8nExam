# Asistente de Clima en n8n (Telegram + Gemini + Open-Meteo)

Este repositorio contiene un workflow de **n8n** que recibe mensajes desde Telegram, prepara contexto del usuario, y usa un agente de IA con una herramienta HTTP para decidir cómo responder sobre consultas de clima.

## Qué aprenderás con este flujo
- Cómo recibir eventos de Telegram en n8n.
- Cómo normalizar datos de entrada con nodos `Set`.
- Cómo conectar un **AI Agent** a un modelo (`Google Gemini`) y a una herramienta externa (`HTTP Request Tool`).
- Cómo diseñar un prompt con reglas de negocio (pedir ciudad cuando falta, no inventar datos, etc.).

## Estado actual del flujo (importante)
El flujo **sí procesa mensajes** y genera un texto de salida en el nodo `Salida Mensaje`, pero **no envía aún la respuesta de vuelta a Telegram** porque el workflow termina ahí.

Esto significa:
1. El usuario escribe al bot.
2. n8n ejecuta el agente.
3. La respuesta queda en ejecución interna (`mensaje`), pero no se publica en chat automáticamente.

## Arquitectura del workflow

### 1) Entrada: `Telegram Trigger`
- Escucha actualizaciones tipo `message`.
- Provee datos como:
  - `message.text`
  - `message.chat.first_name`

### 2) Preparación de datos: `Edit Fields`
Transforma la entrada en variables claras para el agente:
- `chatInput = {{$json.message.text}}`
- `nombre = {{$json.message.chat.first_name}}`

### 3) Orquestación IA: `AI Agent1`
Este nodo aplica reglas de conversación y decide la respuesta.

Reglas principales definidas en el system message:
- Saludar siempre usando el nombre del usuario.
- Si no hay ciudad en el mensaje: pedirla amablemente.
- Si hay ciudad: confirmar que consultará el clima.
- No inventar ciudades.
- Ser breve y natural.

### 4) Modelo de lenguaje: `Google Gemini Chat Model`
- Conectado al agente como `ai_languageModel`.
- Responsable de generar el texto final según prompt y contexto.

### 5) Herramienta externa del agente: `HTTP Request1`
- Nodo tipo `httpRequestTool` conectado como `ai_tool`.
- URL actual configurada:
  - `https://api.open-meteo.com/v1/forecast?latitude=7.1193&longitude=-73.1227&current_weather=true`

Nota pedagógica:
- Aunque el prompt indica "si menciona ciudad", la herramienta hoy está fija a coordenadas predefinidas (no cambia dinámicamente por ciudad).
- Para soportar múltiples ciudades necesitas geocodificación o mapeo ciudad -> lat/lon.

### 6) Salida intermedia: `Salida Mensaje`
Consolida datos para etapas posteriores:
- `chatInput`
- `nombre`
- `mensaje = {{$json.output}}`

Actualmente este nodo es el final del flujo.

## Diagrama lógico
`Telegram Trigger -> Edit Fields -> AI Agent1 -> Salida Mensaje`

Conexiones de soporte:
- `Google Gemini Chat Model -> AI Agent1` (`ai_languageModel`)
- `HTTP Request1 -> AI Agent1` (`ai_tool`)

## Credenciales requeridas
Configura estas credenciales en n8n antes de activar:

1. **Telegram API**
   - Usada por: `Telegram Trigger`
2. **Google Gemini (PaLM) API**
   - Usada por: `Google Gemini Chat Model`

## Cómo importar y ejecutar
1. Abre n8n.
2. Ve a **Workflows** -> **Import from file**.
3. Selecciona `n8n.json`.
4. Reasigna credenciales.
5. Activa el workflow.
6. Escribe al bot en Telegram.
7. Revisa la ejecución en n8n y valida el campo `mensaje` en `Salida Mensaje`.

## Pruebas sugeridas (didácticas)

### Caso A: sin ciudad
Mensaje de usuario: `Hola, ¿cómo está el clima?`
Esperado:
- Saludo con nombre.
- Solicitud de ciudad.
- No debe inventar datos de clima.

### Caso B: con ciudad
Mensaje de usuario: `Hola, ¿qué clima hay en Bucaramanga?`
Esperado:
- Saludo con nombre.
- Confirmación breve de consulta.
- Respuesta natural.

## Limitaciones actuales
1. No hay envío automático de respuesta a Telegram al final.
2. La URL de clima usa coordenadas fijas.
3. El texto del prompt muestra caracteres mal codificados (`Ã`, `â†’`), probablemente por encoding.

## Mejoras recomendadas (siguiente iteración)
1. Agregar nodo **Telegram -> Send Message** al final usando:
   - `chatId = {{$('Telegram Trigger').item.json.message.chat.id}}`
   - `text = {{$json.mensaje}}`
2. Implementar geocodificación de ciudad para obtener lat/lon dinámicos.
3. Corregir codificación UTF-8 del prompt para evitar caracteres extraños.
4. Añadir manejo de errores (API caída, ciudad inválida, respuesta vacía).

## Archivos del proyecto
- `n8n.json`: definición completa del workflow.
- `README.md`: documentación funcional y técnica.
