AgendaBot — Documentación Técnica
1. Arquitectura general

El bot funciona bajo el siguiente flujo:

Telegram → n8n (Telegram Trigger)
         → Consulta SESSIONS (Google Sheets)
         → IF: ¿el usuario tiene una sesión activa?
             ├── NO  → Router principal (Switch, menú 0–8)
             └── SÍ  → Switch de pasos del wizard (paso_actual 0–6)

Esta arquitectura resuelve el reto central del proyecto: n8n no mantiene memoria nativa entre ejecuciones de webhook — cada mensaje de Telegram dispara una ejecución completamente independiente. Para simular una conversación con estado (el wizard de 6 pasos), se usa la hoja SESSIONS como almacenamiento externo de:

pantalla_actual: en qué pantalla/flujo está el usuario (menu_agenda, agendar_cita, etc.)
paso_actual: número de paso dentro del wizard (0–6)
datos_parciales: un JSON en texto plano que va acumulando las respuestas del usuario en cada paso (fecha, hora, nombre, motivo, canal)
2. Componentes técnicos clave
2.1 Entorno de ejecución
n8n Community Edition corriendo vía npx n8n dentro de un GitHub Codespace (Node.js), sin usar n8n Cloud de pago, cumpliendo la restricción del Artículo 2.
ngrok con un dominio fijo gratuito (*.ngrok-free.dev) para exponer el puerto 5678 con HTTPS público estable — necesario porque el reenvío de puertos nativo de Codespaces resultó inestable para recibir webhooks de Telegram de forma confiable.
Variables de entorno necesarias en cada arranque:
bash
  export WEBHOOK_URL="https://<tu-dominio>.ngrok-free.dev"
  export N8N_EDITOR_BASE_URL="https://<tu-dominio>.ngrok-free.dev"
2.2 Router principal (menú 0–8)

Un nodo Switch (modo Rules) con 9 reglas, comparando {{ $json.message.text }} contra cada número del 0 al 8. Cada salida conecta a un nodo Telegram "Send a text message" con el mensaje correspondiente del Artículo 6.

2.3 Wizard de agendamiento (6 pasos)

Cada paso sigue el mismo patrón:

Edit Fields (Set): construye el nuevo JSON de datos_parciales, mezclando lo ya guardado con la nueva respuesta:
   {{ JSON.stringify({...JSON.parse($json.datos_parciales || '{}'), <campo>: $('TelegramTrigger').item.json.message.text}) }}
Google Sheets – Update Row: actualiza la fila de SESSIONS del usuario (matching por telegram_user), avanzando paso_actual y guardando datos_parciales.
Telegram – Send Message: pregunta el siguiente dato del wizard.

Al llegar al paso 6 (confirmación), un segundo Switch interpreta la respuesta (1/2/3) y, si es "1", arma el registro final y lo guarda en la hoja CITAS con un id_cita generado (CITA-XXXX).

3. Lecciones técnicas / troubleshooting relevante
"Always Output Data": necesario en los nodos de Google Sheets que buscan filas, para que el flujo no se detenga en seco cuando la búsqueda no encuentra coincidencias (comportamiento por defecto de n8n).
Referencias entre nodos ($('NombreDeNodo')): al trabajar con múltiples ramas de Switch, los datos de $json cambian según el nodo inmediatamente anterior en esa rama específica — es necesario referenciar explícitamente el nodo fuente correcto (por ejemplo, $('Get row(s) in sheet') para leer datos_parciales actualizado, en vez de asumir que viene en el $json genérico).
Numeración de salidas de un Switch: las salidas se numeran en el orden en que se crean las reglas (0, 1, 2...), no según el valor que se está comparando — un error común fue conectar nodos a la salida equivocada.
Conversión de tipos: activar "Convert types where required" en los nodos Switch evita errores cuando se compara un número (ej. paso_actual) contra un valor de texto.
4. Pruebas realizadas

Ver evidencias en la carpeta /evidencias. Se realizaron pruebas funcionales del flujo completo: navegación del menú principal, submenú de Agenda, y el wizard completo de agendamiento de cita de punta a punta, incluyendo el guardado final en la hoja CITAS.
