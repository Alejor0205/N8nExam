# Bot de Clima con n8n + Telegram + Gemini

Este proyecto contiene un flujo de **n8n** que recibe mensajes desde Telegram, consulta el clima actual en Open-Meteo, genera una respuesta natural usando IA (Gemini) y la devuelve al usuario por Telegram.

## Objetivo
Automatizar respuestas de clima en un bot de Telegram con un mensaje corto, claro y amigable.

## Flujo General
1. **Webhook (`setWebhook`)** recibe un `POST` desde Telegram en la ruta `telegram-bot`.
2. **HTTP Request** consulta el clima actual en Open-Meteo (lat/lon configuradas).
3. **Edit Fields** extrae:
   - `temperature` desde `current_weather.temperature`
   - `windspeed` desde `current_weather.windspeed`
4. **AI Agent1** usa un prompt para redactar un mensaje natural de clima.
5. **Salida Mensaje** guarda el texto final en el campo `menssage`.
6. **Send a text message** envía la respuesta al `chat.id` de Telegram.

## Diagrama de Nodos
`setWebhook -> HTTP Request -> Edit Fields -> AI Agent1 -> Salida Mensaje -> Send a text message`

Nodo adicional:
- `Google Gemini Chat Model` conectado a `AI Agent1` como modelo de lenguaje.

## Datos Clave del Flujo
- **Webhook path:** `telegram-bot`
- **Método webhook:** `POST`
- **API clima:** `https://api.open-meteo.com/v1/forecast?latitude=7.1193&longitude=-73.1227&current_weather=true`
- **Ubicación actual configurada:** lat `7.1193`, lon `-73.1227`

## Credenciales Necesarias en n8n
Configura estas credenciales en tu instancia de n8n antes de activar el flujo:

1. **Telegram API**
   - Usada por el nodo: `Send a text message`
2. **Google Gemini (PaLM) API**
   - Usada por el nodo: `Google Gemini Chat Model`

## Importar el Flujo
1. Abre n8n.
2. Ve a **Workflows** -> **Import from file**.
3. Selecciona `n8n.json`.
4. Reasigna credenciales de Telegram y Gemini.
5. Activa el workflow.

## Configuración de Telegram (resumen)
1. Crea tu bot con `@BotFather` y obtén el token.
2. Configura el webhook de Telegram para apuntar al webhook de n8n (`/webhook/telegram-bot` o `/webhook-test/telegram-bot` según modo).
3. Envía un mensaje al bot para disparar el flujo.

## Prueba Rápida
1. Envía un mensaje cualquiera al bot de Telegram.
2. Verifica en n8n que el workflow se ejecute.
3. Deberías recibir un mensaje corto describiendo el clima actual.

## Estructura de Respuesta
El flujo construye una respuesta basada en:
- Temperatura actual (`°C`)
- Velocidad del viento (`km/h`)

Luego IA la convierte en lenguaje natural para el usuario final.

## Notas
- El campo final se llama `menssage` (con doble “s”); está bien mientras se use de forma consistente en los nodos.
- Si quieres cambiar ciudad/ubicación, modifica `latitude` y `longitude` en el nodo `HTTP Request`.
- Si el texto del prompt se ve con caracteres extraños (acentos mal codificados), revisa la codificación UTF-8 del archivo/exportación.

## Archivo Principal
- `n8n.json`: definición completa del workflow.
