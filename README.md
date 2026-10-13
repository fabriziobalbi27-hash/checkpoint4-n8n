# MODULO 4

**Alumno:** Fabrizio Balbi
**Proyecto:** asistente de atención de Inmobiliaria Balbi (continuación del Módulo 3)

## Archivos

- `checkpoint4_fabrizio_balbi.json`: workflow principal (Manager). Se importa en n8n con *Import from File*.
- `workers/`: los dos sub-workflows que llama el Manager (sin cambios respecto del Módulo 3).
- `evidencia.pdf`: evidencia de la prueba con un email real.

## Qué hace

El Manager del Módulo 3 ahora atiende dos canales:

1. **Entradas:** el chat y la casilla de soporte (Gmail) se normalizan al mismo formato y llegan a una entrada unificada.
2. **Tramo común:** memoria en Airtable (para el email, la sesión es la dirección del remitente), Router de intención y los dos Workers del Módulo 3.
3. **Salida por canal:** el chat responde directo. El email pasa por el CRM (HubSpot), queda como borrador en Gmail para aprobación humana y se avisa al equipo en Slack.

## Los 4 controles de la rúbrica

| Nodo | Qué hace | Error que evita |
|---|---|---|
| ① `IF - ¿Es auto-reply?` | Está inmediatamente después del Gmail Trigger. Corta asuntos Auto-reply, Out of office, Undeliverable, remitentes no-reply@ / mailer-daemon, la propia casilla y el header Auto-Submitted. | Bucle infinito de auto-respuestas |
| ② `HubSpot - Look up Contacto` | Busca el contacto por email antes de escribir. Si existe va a *Update*; si no, a *Create*. | Error 409 (contactos duplicados) |
| ③ `Gmail - Create Draft (HITL)` | La respuesta de la IA queda en Borradores. No hay ningún nodo de envío: una persona la revisa y la envía. | Envíos sin aprobación humana |
| ④ `Set - Limpieza de Payload` | Deja solo remitente, asunto, cuerpo (máx. 3000 caracteres) y thread. Un IF descarta emails sin remitente válido o con el cuerpo vacío. Antes de Slack hay otro Set que recorta el mensaje. | Error 400 (payload mal formado) y canales saturados |

## Conexiones y mínimo privilegio

| Herramienta | Autenticación | Alcance |
|---|---|---|
| Gmail (casilla de soporte) | OAuth2 | Solo lee correos no leídos (`-from:me`) y crea borradores. |
| HubSpot (CRM) | Clave de servicio | Solo 3 permisos: `crm.objects.contacts.read`, `crm.objects.contacts.write` y `crm.schemas.contacts.read`. |
| Slack (canal de operaciones) | OAuth2 | Solo publica en `#todo-n8n`. |
| Airtable (memoria) | Token personal | Solo la base de sesiones del Módulo 3. |

**Sobre HubSpot:** HubSpot ya no permite crear apps OAuth "legacy" en cuentas nuevas. Su reemplazo oficial son las claves de servicio, que además permiten elegir permisos puntuales: el OAuth2 del nodo de n8n exige 14 scopes, mientras que la clave de servicio usa solo los 3 de Contactos. Por eso cumple mejor el principio de mínimo privilegio.

## Prueba realizada

Se envió un email real a la casilla de soporte ("Tasación departamento Fisherton"). Resultado:

- El contacto se creó una sola vez en HubSpot.
- La respuesta quedó como borrador en Gmail, sin enviar.
- Llegó el log a Slack con el canal, el worker y el estado del CRM.
- La sesión quedó registrada en Airtable con el email como identificador.
