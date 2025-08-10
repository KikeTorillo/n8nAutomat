# CLAUDE.md

Este archivo proporciona orientación a Claude Code (claude.ai/code) cuando trabaja con código en este repositorio.

**REGLA IMPORTANTE: Claude debe SIEMPRE responder en español cuando trabaje en este proyecto, sin excepciones.**

## Descripción del Proyecto

Este es un proyecto de bot de WhatsApp basado en n8n con integración de IA para gestión de calendario y procesamiento de documentos. El sistema utiliza Evolution API para conectividad de WhatsApp, DeepSeek AI para procesamiento de lenguaje natural, integración con Google Calendar para programación de citas, y Google Sheets para gestión de datos.

## Arquitectura

### Componentes Principales

**Servicios Docker (docker-compose.yml):**
- `postgres`: Base de datos PostgreSQL (puerto 5432) - almacena workflows de n8n, datos de Evolution API e historial de conversaciones
- `n8n`: Servidor de automatización de workflows (puerto 5678) - orquesta la lógica del bot de WhatsApp
- `evolution_api`: Servicio API de WhatsApp (puerto 8081) - maneja envío/recepción de mensajes de WhatsApp
- `pgadmin`: Interfaz de administración de base de datos (puerto 5050)

**Persistencia de Datos:**
Todos los datos de servicios se almacenan en el directorio `./data/`:
- `data/postgres/` - Archivos de base de datos
- `data/n8n/` - Workflows y credenciales de n8n
- `data/evolution/` - Datos de sesión de WhatsApp e instancias
- `data/pgadmin/` - Configuraciones de pgAdmin

### Arquitectura de Workflows

El proyecto contiene múltiples patrones de workflows de n8n:

**Patrón 1: DeepSeek Oficial (Agente IA Avanzado)**
- Webhook recibe mensajes de WhatsApp
- Edit Fields extrae: remitente, tipo_mensaje, conversación, sesión_id
- Agente IA con modelo DeepSeek Chat procesa mensajes usando framework LangChain
- Postgres Chat Memory proporciona contexto de conversación
- Google Calendar Tool (agenda_cita) maneja operaciones de calendario
- HTTP Request envía respuestas de vuelta vía Evolution API

**Patrón 2: Solicitud HTTP Personalizada (Manejo Manual de Herramientas)**
- Webhook y extracción de campos similar
- Llamada directa a API DeepSeek con consulta de historial de conversación
- Código JavaScript personalizado procesa respuestas de IA y detecta patrones de uso de herramientas
- Creación manual de eventos de calendario cuando se detecta uso de herramientas
- Operaciones separadas de base de datos para guardar mensajes de usuario e IA

**Patrón 3: Asistente de Sheets (Procesamiento de Documentos)**
- Chat Trigger recibe comandos
- Google Docs lee automáticamente documentos
- Nodo Code procesa y estructura datos usando regex
- AI Agent crea hojas de Google Sheets y llena datos
- Flujo híbrido: Code para extracción + IA para automatización

## Comandos de Desarrollo

### Gestión de Contenedores
```bash
npm start              # Iniciar todos los servicios
npm stop              # Detener todos los servicios
npm restart           # Reiniciar todos los servicios
npm run dev           # Construir e iniciar con últimos cambios
npm run dev:fresh     # Reconstrucción limpia con volúmenes frescos
```

### Monitoreo y Depuración
```bash
npm run logs          # Ver todos los logs de contenedores
npm run logs:n8n      # Ver solo logs de n8n
npm run logs:evolution # Ver logs de Evolution API
npm run logs:postgres # Ver logs de PostgreSQL
npm run status        # Verificar estado de contenedores
```

### Operaciones de Base de Datos
```bash
npm run db:connect    # Conectar a PostgreSQL vía psql
npm run backup:db     # Crear respaldo de base de datos
```

### Comandos de Limpieza
```bash
npm run clean         # Detener contenedores y eliminar volúmenes
npm run clean:data    # Reinicio completo incluyendo directorios de datos
npm run fresh:clean   # Reconstrucción limpia completa
```

## Esquema de Base de Datos

**Bases de Datos:**
- `n8n_db`: Datos de workflows y configuraciones de n8n
- `evolution_db`: Datos de WhatsApp de Evolution API
- `chat_memories_db`: Historial de conversaciones para contexto de IA

**Tablas Clave:**
- `conversations`: Almacena historial de chat con columnas: session_id, role, content, created_at
- n8n crea automáticamente tablas para ejecución de workflows y memoria de chat

## Configuración y Credenciales

### Variables de Entorno (en docker-compose.yml)
- PostgreSQL: `POSTGRES_USER=admin`, `POSTGRES_PASSWORD=StrongPassword123!`
- n8n: `N8N_BASIC_AUTH_USER=admin`, `N8N_BASIC_AUTH_PASSWORD=n8nPassword!`
- Evolution API: `AUTHENTICATION_API_KEY=EvoAPIKey123!`
- URL Webhook: Actualmente configurada para túnel ngrok (actualizar para producción)

### Credenciales Requeridas en n8n
- Cuenta API de DeepSeek (con clave API)
- Cuenta OAuth2 API de Google Calendar
- Cuenta OAuth2 API de Google Docs
- Cuenta OAuth2 API de Google Sheets
- Cuenta Postgres (para memoria de chat y operaciones manuales de base de datos)

### APIs de Google Cloud Requeridas
Habilitar estas APIs en Google Cloud Console para el proyecto:
- **API de Google Calendar** - para integración de calendario
- **API de Google Drive** - para herramientas de hojas de datos y operaciones de archivos
- **API de Google Sheets** - para operaciones de hojas de cálculo
- **API de Google Docs** - para procesamiento de documentos

**Problema Común**: Si obtienes errores "403 Forbidden", probablemente la API requerida no está habilitada. Habilitarla vía Google Cloud Console y esperar 2-3 minutos para la propagación.

## Trabajando con Workflows

### Importando Workflows
Los workflows de n8n se almacenan como archivos JSON en la raíz del proyecto:
- `Deepseek oficial.json` - Workflow de agente IA basado en LangChain
- `Example - Custom HTTP Request with Tools.json` - Workflow de manejo manual de herramientas
- `Asistente de sheet - FINAL.json` - Workflow de procesamiento de documentos y Google Sheets

### Componentes Clave de Workflows
- **Nodos Webhook**: Usan ruta `ae59cdbc-961a-4f31-b9e5-701b82d25f1c`
- **Extracción de campos**: Extrae datos de mensajes de WhatsApp (remitente, contenido, sesión)
- **Procesamiento IA**: Agentes LangChain o llamadas directas a API DeepSeek
- **Integración de calendario**: Operaciones de Google Calendar para gestión de citas
- **Procesamiento de documentos**: Extracción automática de datos de Google Docs usando regex
- **Manejo de respuestas**: Envía respuestas formateadas de vuelta a usuarios de WhatsApp

### Flujo de Mensajes de WhatsApp
1. Evolution API recibe mensaje de WhatsApp
2. Webhook activa workflow de n8n
3. Datos de mensaje extraídos y procesados
4. IA determina intención (crear/modificar/cancelar citas, procesar documentos)
5. Operaciones de calendario o sheets realizadas si es necesario
6. Respuesta enviada de vuelta al usuario de WhatsApp
7. Historial de conversación guardado en PostgreSQL

### Flujo de Procesamiento de Documentos
1. Chat Trigger recibe comando de procesamiento
2. Google Docs lee automáticamente el documento especificado
3. Nodo Code extrae datos usando patrones regex específicos
4. AI Agent crea nueva hoja de Google Sheets
5. Datos estructurados se llenan automáticamente en la hoja
6. Confirmación enviada al usuario

## Puntos de Integración

### Configuración de IA DeepSeek
- Modelo: `deepseek-chat`
- Prompt del sistema configurado para gestión de citas de calendario en español
- Zona horaria: `America/Mexico_City`
- Memoria de conversación mantenida vía PostgreSQL

### Integración de Google Calendar
- ID de Calendario: `arellanestorillo@gmail.com`
- Soporta: creación, modificación, eliminación y consulta de eventos
- Manejo automático de zona horaria para Ciudad de México

### Integración de Google Sheets
- Procesamiento híbrido: extracción con Code + automatización con IA
- Mapeo directo de datos extraídos a columnas de hoja
- Nombres de columnas sin caracteres especiales (titulo, categoria, duracion)
- Creación automática de hojas y llenado de datos

### Configuración de Evolution API
- Clave API: `EvoAPIKey123!` (cambiar en producción)
- URL del Servidor: `http://localhost:8081`
- Soporta: envío de mensajes de texto, gestión de sesiones de WhatsApp

## Notas de Estructura de Archivos

- Archivos JSON de workflows en raíz contienen definiciones completas de workflows de n8n
- `init-data.sh` crea bases de datos PostgreSQL requeridas en primera ejecución
- Directorio `data/` está en .gitignore - contiene datos de tiempo de ejecución sensibles
- Todas las credenciales y claves API se almacenan en sistema de credenciales de n8n (no en archivos)

**RECORDATORIO: Todos los prompts, mensajes del sistema y respuestas deben estar en español para mantener consistencia con el proyecto.**