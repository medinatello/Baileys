# Directorio `src/Socket`

Este es el directorio más importante y complejo de Baileys. Contiene el núcleo de la lógica de comunicación, responsable de gestionar la conexión WebSocket con los servidores de WhatsApp y de orquestar todo el flujo de datos.

## Responsabilidades Principales

1.  **Gestión de la Conexión**:
    - Establece, mantiene y cierra la conexión WebSocket.
    - Maneja la autenticación inicial, ya sea mediante código QR, código de emparejamiento o restaurando una sesión existente.
    - Implementa un sistema de `keep-alive` para evitar que la conexión se cierre por inactividad.
    - Gestiona la reconexión automática en caso de fallos de red.

2.  **Procesamiento de Mensajes**:
    - Escucha los datos entrantes del WebSocket.
    - Decodifica los nodos binarios (utilizando la lógica de `WABinary`).
    - Desencapsula los mensajes y los pasa al módulo `Signal` para su descifrado.
    - Emite eventos (`messages.upsert`, `connection.update`, etc.) que la aplicación cliente puede escuchar.

3.  **Envío de Mensajes**:
    - Proporciona una API (ej. `sendMessage`) para que el cliente envíe datos.
    - Prepara, encripta (usando `Signal`) y codifica los mensajes salientes al formato binario de WhatsApp.
    - Gestiona la lógica de reintentos en caso de que un mensaje no se pueda enviar.

4.  **Orquestación de Funcionalidades**:
    - Separa la lógica de diferentes características en archivos dedicados (`groups.ts`, `chats.ts`, `business.ts`, etc.), que son coordinados desde el archivo principal `socket.ts`.

Debido a su papel central, una comprensión profunda de este directorio es esencial para la migración del proyecto. Los archivos `contents.md` y `diagram.md` ofrecen un desglose más detallado de su estructura interna.
