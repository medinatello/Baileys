# Contenido del Directorio `src/Defaults`

Este documento detalla el contenido de los archivos que se encuentran dentro del directorio `src/Defaults`.

## Archivos

### `baileys-version.json`

Este archivo JSON contiene la versión de la "aplicación" que Baileys simula ser. Es un array de números que se envía a los servidores de WhatsApp como parte del proceso de identificación del cliente.

**Contenido de Ejemplo:**
```json
{
	"version": [2, 3000, 1023223821]
}
```

### `index.ts`

Este archivo es el núcleo del directorio. Importa la versión desde `baileys-version.json` y define y exporta una gran cantidad de constantes y un objeto de configuración por defecto que se utilizan en toda la aplicación.

#### Constantes Principales Exportadas

- **`UNAUTHORIZED_CODES`**: Un array de códigos de estado HTTP (`[401, 403, 419]`) que indican que la sesión ya no es válida y se debe cerrar la sesión.
- **`DEFAULT_ORIGIN`**: La URL de origen por defecto, `https://web.whatsapp.com`.
- **`WA_DEFAULT_EPHEMERAL`**: La duración por defecto de los mensajes efímeros, configurada en 7 días (en segundos).
- **`NOISE_MODE`**, **`DICT_VERSION`**, **`KEY_BUNDLE_TYPE`**, **`NOISE_WA_HEADER`**: Constantes técnicas relacionadas con el protocolo de cifrado Noise, que se utiliza para establecer la conexión segura inicial con WhatsApp.
- **`URL_REGEX`**: Una expresión regular para detectar URLs en los mensajes de texto.
- **`PROCESSABLE_HISTORY_TYPES`**: Un array que define los tipos de sincronización de historial que la librería puede procesar.
- **`MEDIA_PATH_MAP`**: Un objeto que mapea cada tipo de medio (imagen, video, etc.) a la ruta del endpoint del servidor de WhatsApp correspondiente para su subida.
- **`MEDIA_HKDF_KEY_MAPPING`**: Mapea los tipos de medios a las "claves de derivación de claves basadas en HMAC" (HKDF), utilizadas en el proceso de cifrado de medios.
- **`MIN_PREKEY_COUNT`**, **`INITIAL_PREKEY_COUNT`**: Constantes que definen el número mínimo y el número inicial de "pre-claves" necesarias para el protocolo Signal.
- **`DEFAULT_CACHE_TTLS`**: Tiempos de vida (TTL) por defecto para diferentes tipos de cachés internas, como la de reintento de mensajes o la de información de dispositivos.

#### Objeto de Configuración Principal

- **`DEFAULT_CONNECTION_CONFIG`**:
  Este es un objeto de tipo `SocketConfig` que contiene la configuración por defecto para una nueva conexión de socket. Es el objeto más importante de este archivo. Si un usuario no proporciona su propia configuración al crear un socket, se utilizan estos valores.

  **Miembros Clave de `DEFAULT_CONNECTION_CONFIG`:**
  - `version`: La versión obtenida de `baileys-version.json`.
  - `browser`: Simula un navegador por defecto (ej. Chrome en Ubuntu) para la identificación del cliente.
  - `waWebSocketUrl`: La URL del servidor WebSocket de WhatsApp.
  - `connectTimeoutMs`: Tiempo de espera para la conexión (20 segundos).
  - `keepAliveIntervalMs`: Intervalo para mantener la conexión activa (30 segundos).
  - `logger`: Una instancia del logger `pino` por defecto.
  - `auth`: Estado de autenticación, indefinido por defecto.
  - `syncFullHistory`: Booleano que indica si se debe sincronizar todo el historial de chats al conectar.
  - `getMessage`: Una función (por defecto vacía) para obtener mensajes de un almacén personalizado.
  - `makeSignalRepository`: Una función para crear el repositorio de claves de Signal.
