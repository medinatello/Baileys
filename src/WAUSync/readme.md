# Directorio `src/WAUSync`

Este directorio implementa la lógica para el protocolo **USync (User Sync)** de WhatsApp.

## Propósito

USync es un protocolo que WhatsApp utiliza para sincronizar de manera eficiente el estado de varios tipos de datos del usuario entre el cliente y el servidor. En lugar de descargar toda la información cada vez, USync permite al cliente solicitar solo los cambios que han ocurrido desde la última sincronización.

Las responsabilidades de este directorio incluyen:

1.  **Construir Consultas de Sincronización**: Crear las solicitudes (`queries`) que se envían al servidor para pedir actualizaciones de datos. Estas consultas especifican qué tipo de datos se quieren sincronizar (contactos, dispositivos, etc.) y desde qué "versión" o estado se parte.
2.  **Manejar Protocolos Específicos**: La sincronización de cada tipo de dato (contactos, listas de bloqueo, configuraciones) sigue un "protocolo" ligeramente diferente. El subdirectorio `Protocols/` organiza la lógica para cada uno de estos tipos.
3.  **Procesar las Respuestas**: Interpretar las respuestas del servidor para actualizar el estado local de la aplicación.

Este módulo es importante para mantener la aplicación sincronizada con los datos más recientes del usuario sin tener que realizar descargas masivas de información, lo que mejora la eficiencia y la velocidad.
