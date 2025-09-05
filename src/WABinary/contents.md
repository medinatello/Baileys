# Contenido del Directorio `src/WABinary`

Este documento detalla el contenido de los archivos que se encuentran dentro del directorio `src/WABinary`. Este módulo es el motor de serialización y deserialización del protocolo de bajo nivel de WhatsApp.

## Archivos Principales

### `decode.ts`

Este archivo es responsable de **deserializar** un buffer de bytes recibido del servidor de WhatsApp y convertirlo en un objeto `BinaryNode` estructurado y legible.

**Funciones y Lógica Clave:**
- **`decodeBinaryNode(buff)`**: La función principal. Primero, descomprime el buffer si es necesario (usando `zlib`) y luego llama al decodificador principal.
- **`decodeDecompressedBinaryNode(...)`**: El núcleo del parser. Es una máquina de estados que lee el buffer byte a byte para reconstruir el nodo.
  - Lee la etiqueta del nodo, los atributos (pares clave-valor) y el contenido.
  - Maneja de forma recursiva el contenido si este es una lista de otros nodos.
  - Utiliza diccionarios de "tokens" (ver `constants.ts`) para decodificar eficientemente cadenas comunes que se representan con uno o dos bytes.
  - Contiene lógica para decodificar formatos especiales como JIDs, y cadenas empaquetadas en `hex` o `nibble`.

### `encode.ts`

Este archivo realiza la operación inversa a `decode.ts`. Es responsable de **serializar** un objeto `BinaryNode` (que representa un comando o mensaje) en un buffer de bytes que puede ser enviado al servidor de WhatsApp.

**Funciones y Lógica Clave:**
- **`encodeBinaryNode(node)`**: La función principal que inicia el proceso de codificación.
- **`encodeBinaryNodeInner(...)`**: El núcleo del codificador. De forma recursiva, convierte la estructura del nodo en una secuencia de bytes.
  - Escribe la etiqueta, los atributos y el contenido del nodo en el buffer.
  - **`writeString(str)`**: Es la función más compleja. Antes de escribir una cadena, intenta optimizarla de varias maneras:
    1.  **Tokenización**: Si la cadena es común (ej. "message", "notification"), la reemplaza con un token de uno o dos bytes.
    2.  **Empaquetado**: Si la cadena es de tipo `hex` o `nibble`, la empaqueta para que dos caracteres quepan en un solo byte.
    3.  **Codificación de JID**: Si es un ID de WhatsApp, utiliza un formato especial y más corto.
    4.  **Raw**: Si no se puede optimizar, la escribe como una cadena UTF-8 normal, precedida por su longitud.

## Archivos de Soporte

- **`constants.ts`**:
  Contiene las constantes utilizadas por el codificador y el decodificador. Lo más importante aquí son los **diccionarios de tokens**:
  - `SINGLE_BYTE_TOKENS`: Un array de cadenas comunes. El índice del array es el byte que representa la cadena.
  - `DOUBLE_BYTE_TOKENS`: Un diccionario de diccionarios para cadenas que necesitan dos bytes.
  - `TAGS`: Un enum con los valores numéricos de todas las etiquetas especiales (ej. `LIST_8`, `BINARY_20`, `JID_PAIR`).

- **`jid-utils.ts`**:
  Proporciona funciones de utilidad para manejar los identificadores de WhatsApp (JIDs).
  - `jidDecode(jid)`: Parsea una cadena JID (ej. `12345@s.whatsapp.net`) y la descompone en sus partes (usuario, servidor, dispositivo).
  - `jidEncode(...)`: Realiza la operación inversa.

- **`types.ts`**:
  Define las interfaces de TypeScript para este módulo, siendo la más importante `BinaryNode`, que define la estructura de un nodo con `tag`, `attrs` y `content`.

- **`index.ts`**:
  Un archivo barril que re-exporta todas las funciones, tipos y constantes importantes del directorio para que sean fácilmente accesibles desde otros módulos.

- **`generic-utils.ts`**:
  Contiene utilidades genéricas utilizadas dentro de este módulo.
