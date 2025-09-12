[🏠 Volver al Índice](../../src/navigation.md)

---

# 📋 Documentación de Eventos

Esta carpeta contiene la documentación para cada evento que puede ser invocado por la aplicación. Cada archivo describe un evento, su propósito, y el flujo de datos asociado.

## 🔄 Flujos Principales

### [📈 Flujo General del Proyecto](../project-flow.md)
Diagrama de flujo completo de la aplicación Baileys, desde la inicialización hasta la interacción con WhatsApp.

## 📱 Eventos de Mensajería

### [📩 Recibir un Mensaje](./receiving-message.md)
Describe el flujo completo cuando Baileys recibe un mensaje de WhatsApp desde el servidor.
- Transporte WebSocket
- Descifrado con Noise y WABinary  
- Procesamiento y cifrado con Signal
- Emisión del evento `messages.upsert`

### [📤 Enviar un Mensaje](./sending-message.md)
Explica el proceso cuando un cliente envía un mensaje usando `sendMessage`.
- Preparación del mensaje en `messages-send.ts`
- Cifrado con SignalRepository
- Codificación binaria y transmisión

## 🔐 Eventos de Autenticación

### [🔑 Creación/Restauración de Sesión](./session-creation.md)
Documenta el flujo de conexión a WhatsApp para nuevas sesiones y restauración de sesiones existentes.
- Handshake de Noise
- Autenticación con QR/código de emparejamiento
- Establecimiento de conexión autenticada

---

## 🧭 Navegación Rápida

| Evento | Descripción | Módulos Involucrados |
|---------|-------------|---------------------|
| [📩 Recibir](./receiving-message.md) | Procesamiento de mensajes entrantes | Socket, WABinary, Signal |
| [📤 Enviar](./sending-message.md) | Envío de mensajes salientes | Socket, Signal, WABinary |  
| [🔑 Sesión](./session-creation.md) | Autenticación y conexión | Socket, Signal, Defaults |

---

📌 **Tip**: Cada evento incluye diagramas de flujo actualizados que se visualizan correctamente en VS Code.
