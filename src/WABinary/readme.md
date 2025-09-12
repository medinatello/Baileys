# Directorio `src/WABinary`

Este directorio contiene una de las partes más técnicas y de más bajo nivel de Baileys: la lógica para **codificar y decodificar el formato binario de WhatsApp**.

## Propósito

WhatsApp no se comunica utilizando formatos legibles como JSON o XML directamente sobre el WebSocket. En su lugar, utiliza un formato binario altamente optimizado para reducir el tamaño de los datos y mejorar la eficiencia. Este formato representa los mensajes y comandos como una estructura de "nodos" con etiquetas, atributos y contenido, similar a XML pero en formato binario.

La responsabilidad de este directorio es:

1.  **Codificar (Encode)**: Tomar un objeto `BinaryNode` (la representación en memoria de un mensaje o comando) y convertirlo en un `Buffer` de bytes que pueda ser enviado a través del WebSocket.
2.  **Decodificar (Decode)**: Tomar un `Buffer` de bytes recibido del WebSocket y convertirlo de nuevo en un objeto `BinaryNode` que el resto de la aplicación pueda entender y procesar.

Además, incluye utilidades para manejar los identificadores de WhatsApp (JIDs) y constantes relacionadas con el protocolo binario.

Comprender este directorio es fundamental para entender cómo se serializan y deserializan los datos que se envían y reciben de los servidores de WhatsApp. La migración a Go requerirá una reimplementación cuidadosa de estos algoritmos de codificación y decodificación.
