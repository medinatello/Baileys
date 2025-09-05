# Directorio `src/Signal`

Este directorio es uno de los más críticos del proyecto, ya que contiene toda la lógica relacionada con el **Protocolo Signal**, que es el responsable del cifrado de extremo a extremo (E2EE) en WhatsApp.

La correcta implementación de este protocolo es lo que permite a Baileys enviar y recibir mensajes de forma segura, garantizando que solo el emisor y el receptor puedan leer el contenido.

## Contenido

- **`libsignal.ts`**: Actúa como un adaptador o un puente (`wrapper`) hacia la librería de Signal (`libsignal-node`). Define una interfaz unificada para que el resto de la aplicación interactúe con el protocolo sin tener que conocer los detalles de bajo nivel de la librería subyacente.
- **`Group/`**: Este subdirectorio contiene la lógica específica para manejar el cifrado en los chats grupales. El cifrado en grupos es más complejo que en los chats individuales, ya que implica la gestión de claves para múltiples participantes (Sender Keys). Aquí se implementa el `GroupCipher` para construir y gestionar las sesiones de cifrado de grupo.

Al migrar este proyecto, la comprensión de este directorio será fundamental para replicar la capacidad de cifrar y descifrar mensajes correctamente.
