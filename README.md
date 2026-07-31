Proyecto_IA1_JuanMontoya
AgendaBot Services — Bot de agendamiento automatizado con Telegram + n8n + Google Sheets
Bot conversacional para agendar citas, gestionar tareas y recordatorios, construido con tecnologías gratuitas y sin dependencia de plataformas de pago, según los lineamientos del proyecto.
Stack tecnológico
Telegram — interfaz conversacional (bot @Agendabot_juan_bot)
n8n (Community Edition, self-hosted) — motor de automatización, corriendo sobre un GitHub Codespace (Node.js/npx), sin usar n8n Cloud de pago
Google Sheets — base de datos (AgendaBot_DB)
ngrok — túnel HTTPS con dominio fijo para exponer el webhook de n8n públicamente
Router principal: recibe cualquier mensaje de Telegram y enruta según la opción numérica elegida (Menú principal 0–8), usando un nodo Switch.
Menú principal y menú de Ayuda (Artículo 6 y 7 del documento).
Submenú de Agenda (Artículo 9).
Wizard completo de "Agendar nueva cita" (Artículo 10), de 6 pasos:
Fecha
Hora
Nombre
Motivo
Canal de atención (presencial / virtual / llamada)
Confirmación y guardado final en la hoja CITAS
Persistencia de sesión por usuario en la hoja SESSIONS (Google Sheets), permitiendo que n8n "recuerde" en qué paso del wizard va cada usuario entre un mensaje y otro — la parte técnica más compleja del proyecto, dado que n8n no mantiene estado nativo entre ejecuciones de webhook.
Guardado final de la cita en la hoja CITAS, con id_cita generado automáticamente, estado: confirmada y creado_por (el chat_id del usuario).
├── README.md
├── docs/
│   └── AgendaBot.md       # Documentación técnica detallada
├── workflows/
│   └── backup_workflows.json   # Export del workflow de n8n
└── evidencias/
    └── (capturas de pantalla de pruebas)
Modelo de datos (Google Sheets — AgendaBot_DB)

Implementadas: CITAS, SESSIONS. Diseñadas en el documento pero no implementadas: TAREAS, HABITOS, LISTAS, ITEMS_LISTA, USUARIOS, LOGS.

Autor
Juan Montoya
