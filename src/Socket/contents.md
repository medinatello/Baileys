# Contenido del Directorio `src/Socket`

Este documento detalla el contenido de los archivos y subdirectorios dentro de `src/Socket`, el cual constituye el núcleo de la librería.

## Archivos Principales

### `index.ts`

Es el punto de entrada público para crear un socket. Su única responsabilidad es:
1.  Recibir la configuración proporcionada por el usuario (`UserFacingSocketConfig`).
2.  Fusionarla con la configuración por defecto (`DEFAULT_CONNECTION_CONFIG`).
3.  Llamar a la siguiente capa de la "fábrica" de sockets, `makeCommunitiesSocket`, para continuar con el proceso de construcción.

**Miembros Principales:**
- **`makeWASocket(config)`**: La función principal exportada para inicializar una conexión.

### `socket.ts`

Este es el archivo más importante del directorio y, posiblemente, de todo el proyecto. Contiene la función `makeSocket`, que implementa la lógica fundamental de la conexión con WhatsApp.

**Funciones y Miembros Clave:**
- **`makeSocket(config)`**:
  - **Inicialización**: Configura el cliente WebSocket, el manejador del protocolo de cifrado Noise para el handshake, y el `SignalRepository` para el cifrado de mensajes.
  - **Conexión**: Implementa `validateConnection` para realizar el handshake criptográfico con los servidores de WhatsApp y registrarse o iniciar sesión.
  - **Comunicación**: Define funciones de bajo nivel como `sendRawMessage` y `sendNode` para enviar datos, y `query` para enviar una solicitud y esperar su respuesta.
  - **Recepción de Mensajes**: `onMessageReceived` es el manejador principal de datos entrantes. Descodifica los frames, y emite eventos internos (`ws.emit`) basados en las etiquetas de los mensajes, permitiendo que diferentes partes del código reaccionen a respuestas específicas.
  - **Gestión de Estado**: Maneja el ciclo de vida de la conexión, incluyendo la generación de códigos QR (`pair-device`), la confirmación del emparejamiento (`pair-success`), el inicio de sesión exitoso (`success`), y el cierre de la conexión (`end`).
  - **Keep-Alive**: Implementa un `setInterval` para enviar pings periódicos y asegurar que la conexión no se pierda.
  - **API Pública**: Devuelve un objeto con todas las funciones y propiedades que las capas superiores utilizarán para interactuar con el socket (ej. `query`, `logout`, `ev`, `authState`).

## Archivos de Funcionalidades Específicas

Estos archivos parecen encapsular la lógica para diferentes tipos de interacciones o "features" de WhatsApp, probablemente para mantener el archivo `socket.ts` más limpio. La función `makeSocket` proporciona las primitivas de bajo nivel, y estas capas superiores las utilizan para implementar funcionalidades concretas.

- **`messages-recv.ts`**:
  Probablemente contiene la lógica para procesar los mensajes entrantes una vez que son descifrados. Escucha los eventos de `socket.ts` y realiza acciones como guardar mensajes en la store, actualizar chats, etc.
- **`messages-send.ts`**:
  Contiene la lógica de alto nivel para enviar mensajes. Proporciona funciones como `sendMessage` que toman un objeto de mensaje amigable, lo preparan, lo pasan al `SignalRepository` para ser cifrado, y luego usan `query` o `sendNode` de `socket.ts` para enviarlo.
- **`groups.ts`**:
  Contiene la lógica para interactuar con grupos (crear, modificar, obtener metadatos, etc.).
- **`chats.ts`**:
  Contiene la lógica para modificar chats (archivar, silenciar, etc.).
- **`business.ts`**:
  Maneja las funcionalidades específicas de las cuentas de WhatsApp Business.
- **`communities.ts`**:
  Maneja la lógica relacionada con las Comunidades de WhatsApp. La función `makeCommunitiesSocket` que se llama desde `index.ts` probablemente añade esta capa de funcionalidad al socket base.
- **`newsletter.ts`**:
  Maneja la lógica para las newsletters o canales.
- **`usync.ts`**:
  Probablemente implementa la lógica para el protocolo `USync` de WhatsApp, para sincronizar datos del estado de la aplicación.
- **`mex.ts`**:
  El nombre es ambiguo, pero podría estar relacionado con "Message Exchange" o alguna funcionalidad específica del protocolo.

## Subdirectorios

### `Client/`

Este directorio contiene la implementación del cliente WebSocket de bajo nivel.

- **`websocket.ts`**:
  Define la clase `WebSocketClient`, que es una envoltura (`wrapper`) sobre la librería `ws`. Se encarga de la lógica directa de conexión al WebSocket, manejo de eventos de `open`, `close`, `message`, y errores a nivel de transporte.
- **`types.ts`**:
  Define los tipos de TypeScript específicos para el cliente WebSocket.
- **`index.ts`**:
  Exporta los componentes del cliente.
