# Contenido del Directorio `src/WAM`

Este documento detalla el contenido de los archivos que se encuentran dentro del directorio `src/WAM`, responsable de la codificación de las métricas de analítica de WhatsApp.

## Archivos

### `encode.ts`

Este es el archivo principal del módulo y contiene la lógica para **serializar** o **codificar** los eventos de analítica en un buffer binario.

**Funciones Clave:**

- **`encodeWAM(binaryInfo)`**: Es la función principal y punto de entrada. Orquesta todo el proceso de codificación:
  1.  Llama a `encodeWAMHeader` para escribir la cabecera del buffer de WAM.
  2.  Llama a `encodeEvents` para codificar la secuencia de eventos.
  3.  Ensambla los fragmentos de buffer en un único buffer final.

- **`encodeEvents(binaryInfo)`**: Itera sobre cada evento de analítica y sus propiedades, y utiliza `serializeData` para convertir cada pieza de información a formato binario.

- **`serializeData(key, value, flag)`**: Es el motor de la serialización. Toma una clave (ID numérico), un valor y un "flag" (que contiene metadatos). Determina el tipo del valor (string, integer, float, boolean) y elige la forma más eficiente de representarlo en binario. Por ejemplo, un entero pequeño se guarda en 1 byte, mientras que uno grande puede usar 4 u 8 bytes.

### `BinaryInfo.ts`

Este archivo define la clase `BinaryInfo`. Un objeto de esta clase actúa como un "contenedor" para el conjunto de eventos de analítica que se van a enviar. Almacena la lista de eventos, los atributos globales, la versión del protocolo y el número de secuencia. El objeto `BinaryInfo` se pasa a la función `encodeWAM` para ser procesado.

### `constants.ts`

Contiene las constantes utilizadas en el proceso de codificación. Las más importantes son:

- **`WEB_EVENTS`**: Una lista masiva de todos los posibles eventos de analítica que se pueden enviar. Cada evento tiene un `name` (nombre legible), un `id` (código numérico) y una lista de `props` (propiedades) que puede tener.
- **`WEB_GLOBALS`**: Una lista de todos los posibles atributos "globales" que se pueden adjuntar a los eventos.
- **`FLAG_*`**: Constantes de bits (`flags`) que se utilizan en la cabecera de cada campo serializado para indicar metadatos, como el tipo de datos que contiene.

### `index.ts`

Un archivo barril que exporta las funciones y clases principales del directorio para que sean utilizadas por otros módulos (principalmente el `Socket`).
