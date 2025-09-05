# Índice de Documentación

Este documento sirve como índice principal para toda la documentación del proyecto. A medida que se genere la documentación, se agregarán enlaces aquí.

## Documentación General

- [Resumen del Proyecto](./project-overview.md): Una visión general de alto nivel sobre qué es Baileys, su arquitectura y las tecnologías que utiliza.
- [Diagrama de Flujo del Proyecto](./project-flow.md): Un diagrama que ilustra el flujo de trabajo general de la aplicación.

## Documentación por Directorio

*Próximamente...*

## Documentación por Evento

- [Creación o Restauración de una Sesión](./events/session-creation.md): Describe el flujo de conexión y autenticación.
- [Envío de un Mensaje](./events/sending-message.md): Describe el flujo desde la llamada a `sendMessage` hasta la transmisión del mensaje.
- [Recepción de un Mensaje](./events/receiving-message.md): Describe el flujo desde la recepción de datos del socket hasta la emisión del evento `messages.upsert`.
