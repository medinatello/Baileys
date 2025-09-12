# Contenido del Directorio `src/Utils`

Este documento describe los archivos de utilidad que se encuentran dentro del directorio `src/Utils`. Cada archivo agrupa un conjunto de funciones auxiliares relacionadas por dominio.

## Archivos

- **`auth-utils.ts`**:
  Contiene funciones relacionadas con el manejo de la autenticación y el estado de la sesión. Incluye lógica para generar nodos de registro y de inicio de sesión, y la famosa `useMultiFileAuthState`, que es una utilidad clave para persistir el estado de autenticación en múltiples archivos JSON.

- **`crypto.ts`**:
  Proporciona funciones criptográficas genéricas que se utilizan en varias partes del sistema, como encriptación/desencriptación AES, funciones hash (SHA256), y otras operaciones de bajo nivel.

- **`messages.ts`** y **`messages-media.ts`**:
  Estos archivos contienen una gran cantidad de lógica para construir, procesar y manipular mensajes. `messages-media.ts` se especializa en el manejo de archivos multimedia: funciones para descargar medios de un mensaje, preparar medios para su subida, generar miniaturas, etc.

- **`decode-wa-message.ts`**:
  Contiene la lógica para decodificar un mensaje de WhatsApp (`WAMessage`) una vez que ha sido descifrado, extrayendo su contenido y metadatos a un formato más fácil de usar.

- **`process-message.ts`**:
  Contiene la lógica para procesar un mensaje recibido. Probablemente es llamado por el `Socket` después de que un mensaje es decodificado y desencriptado, para realizar acciones adicionales o estandarizarlo antes de emitir el evento `messages.upsert`.

- **`noise-handler.ts`**:
  Implementa el manejador para el protocolo de cifrado Noise. Este es utilizado durante la fase inicial de conexión (handshake) para establecer un canal de comunicación seguro con el servidor de WhatsApp, antes de que la lógica de Signal tome el control.

- **`signal.ts`**:
  Contiene utilidades específicas para el protocolo Signal, como la generación de claves públicas o la preparación de datos para el `SignalRepository`.

- **`validate-connection.ts`**:
  Contiene lógica para validar el estado de la conexión, probablemente utilizada por el `Socket` para generar los nodos de login y registro.

- **`event-buffer.ts`**:
  Implementa `makeEventBuffer`, una utilidad muy importante que permite almacenar eventos en un búfer y liberarlos (`flush`) más tarde. Esto es crucial durante el inicio de sesión para asegurar que los eventos no se emitan hasta que la conexión esté completamente establecida y sincronizada.

- **`baileys-event-stream.ts`**:
  Define una utilidad para gestionar un flujo de eventos de Baileys, posiblemente para un manejo más avanzado o reactivo.

- **`chat-utils.ts`** e **`history.ts`**:
  Contienen funciones para manipular chats y para procesar el historial de mensajes que se recibe al sincronizar.

- **`link-preview.ts`**:
  Contiene la lógica para generar vistas previas de enlaces, interactuando con librerías externas como `link-preview-js`.

- **`logger.ts`**:
  Configura e instancia el logger `pino` que se utiliza en todo el proyecto.

- **`lt-hash.ts`**:
  Implementa la lógica para "LTHash", un tipo de hash utilizado por WhatsApp para sincronizar el historial de mutaciones (cambios en chats, etc.) de manera eficiente.

- **`make-mutex.ts`**:
  Proporciona una utilidad para crear un "mutex", que es un mecanismo para evitar condiciones de carrera al acceder a recursos compartidos de forma asíncrona.

- **`index.ts`**:
  Como en otros directorios, es un **archivo barril** que re-exporta todas las utilidades de los otros archivos, permitiendo importaciones limpias desde un único lugar.
