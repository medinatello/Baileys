# Contenido del Directorio `src/Types`

Este documento detalla los archivos que se encuentran dentro del directorio `src/Types`. Cada archivo está dedicado a definir las interfaces y tipos de TypeScript para un dominio específico de la aplicación.

## Archivos

- **`Auth.ts`**:
  Define la estructura del estado de autenticación (`AuthenticationState`). Esto incluye las credenciales (`creds`) y las claves (`keys`) necesarias para mantener y restaurar una sesión. Es uno de los tipos más importantes del proyecto.

- **`Chat.ts`**:
  Contiene las definiciones para los chats, como `Chat`, `Conversation`, etc. Define qué propiedades tiene un chat (ej. `id`, `name`, `unreadCount`, `timestamp`).

- **`Contact.ts`**:
  Define la estructura de un objeto de contacto, incluyendo su `id`, `name`, `pushName`, etc.

- **`Message.ts`**:
  Define la estructura completa de un mensaje de WhatsApp (`WAMessage`). Este es un tipo muy complejo que incluye el contenido del mensaje, la clave (`key`), metadatos como el `timestamp`, información del remitente, etc. También define los tipos para los diferentes contenidos de mensajes (texto, imagen, video, etc.).

- **`Socket.ts`**:
  Contiene las definiciones relacionadas con la configuración del socket, principalmente la interfaz `SocketConfig`, que detalla todas las opciones posibles para configurar una nueva conexión.

- **`Events.ts`**:
  Define la estructura de los eventos que emite el socket a través del emisor de eventos (`ev`). Mapea cada nombre de evento (ej. `connection.update`, `messages.upsert`) al tipo de datos que se pasa con ese evento.

- **`GroupMetadata.ts`**:
  Define la estructura de los metadatos de un grupo, que incluye la lista de participantes (`participants`), el asunto (`subject`), la descripción (`desc`), etc.

- **`Signal.ts`**:
  Define los tipos e interfaces necesarios para interactuar con el repositorio de Signal, como `SignalRepository` y `SignalKeyStore`.

- **`State.ts`**:
  Define tipos relacionados con el estado general de la conexión y la aplicación.

- **`Call.ts`**:
  Contiene las definiciones de tipos para los eventos y datos relacionados con las llamadas de WhatsApp.

- **`Label.ts`** y **`LabelAssociation.ts`**:
  Definen tipos para las "etiquetas" (Labels), una funcionalidad de WhatsApp Business para organizar chats.

- **`Newsletter.ts`**:
  Define los tipos para la funcionalidad de Canales o Newsletters.

- **`Product.ts`**:
  Define la estructura de los productos, para la funcionalidad de catálogos de WhatsApp Business.

- **`USync.ts`**:
  Define los tipos de datos utilizados en el protocolo de sincronización `USync`.

- **`index.ts`**:
  Este es un **archivo barril (barrel file)**. No define nuevos tipos (excepto algunos genéricos como `DisconnectReason`), sino que utiliza `export * from './...'` para re-exportar todas las definiciones de los otros archivos del directorio. Esto permite a otros módulos importar cualquier tipo directamente desde `@/Types` en lugar de tener que saber en qué archivo específico está definido. Facilita la importación y el uso de los tipos en todo el proyecto.
