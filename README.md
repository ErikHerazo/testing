# Bot Test Console

Página estática (`index.html`, sin dependencias/build) para probar el bot de un
cliente directo por HTTP, sin pasar por Zoho ni por su UI real.

Pensada para reutilizarse con cualquier cliente que exponga el mismo contrato
de endpoint de prueba (`POST <backend>/web/chat/test-hexagonal`), no solo AGB.

## Uso

1. Abre `index.html` en el navegador (doble click, o `python3 -m http.server`
   en esta carpeta y entra a `http://localhost:8000`).
2. En "Backend URL" pon la URL del backend del cliente que quieras probar
   (ej. `http://localhost:8000` si corres el backend en local con Docker).
3. En "X-Test-Secret" pon el valor de `CHAT_TEST_SECRET` del `.env` de ese
   backend.
4. En "Modelo" elegí qué LLM responde (GPT-4o o Claude, si el backend lo
   soporta — ver requisito abajo).
5. Escribe mensajes y revisa las respuestas del bot en tiempo real.

La configuración (URL, secret, canal, modelo) se guarda en `localStorage` del
navegador, así que no hay que reescribirla cada vez.

## Requisito del lado del backend

El backend del cliente debe exponer:

```
POST /web/chat/test-hexagonal
Headers: X-Test-Secret: <valor de CHAT_TEST_SECRET>
Body: {"message": str, "channel": str, "session_id": str|null, "visitor_language": str|null, "engine": "azure_openai"|"claude"}
Response: {"session_id": str, "answer": str}
```

`engine` es opcional (default `azure_openai`) — un backend que no soporte
elegir motor puede ignorarlo y siempre va a responder igual.

En `AngelBotBackend` (cliente AGB) esto vive en
`app/web/routes/chat_test_hexagonal.py` — deshabilitado (404) a menos que
`CHAT_TEST_SECRET` esté definido en el `.env` de ese backend. Existe también
`POST /web/chat/test` (mismo contrato, sin el campo `engine`, siempre
GPT-4o) para clientes que todavía no migraron al pipeline hexagonal.
