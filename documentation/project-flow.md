# Diagrama de Flujo del Proyecto

Este documento contiene un diagrama de Mermaid que ilustra el flujo general de la aplicación Baileys, desde la inicialización hasta la interacción con WhatsApp.

```mermaid
graph TD
    subgraph "Fase de Inicialización"
        A[Cliente inicia la aplicación] --> B(Llama a `makeWASocket` para crear una instancia de socket);
        B --> C{¿Existe sesión guardada?};
        C -- Sí --> D[Carga de credenciales desde `authState`];
        C -- No --> E[Se prepara para nueva conexión];
    end

    subgraph "Fase de Conexión y Autenticación"
        D --> F(Conexión al WebSocket de WhatsApp);
        E --> F;
        F --> G{Evento `connection.update`};
        G -- Conexión abierta --> H{¿Necesita autenticación?};
        H -- Sí (Nueva sesión) --> I{Mostrar QR o Código de Emparejamiento};
        I --> J[Usuario escanea/introduce el código];
        J --> K(Autenticación exitosa);
        H -- No (Sesión restaurada) --> K;
        K --> L[Evento `creds.update`: Guarda la nueva sesión/credenciales];
    end

    subgraph "Fase de Operación"
        L --> M(Conexión establecida y autenticada);
        M --> N[Escucha de eventos del socket: `messages.upsert`, `groups.update`, etc.];

        subgraph "Flujo de Mensaje Entrante"
            O[Servidor de WhatsApp envía evento] --> N;
            N -- `messages.upsert` --> P[Procesa y desencripta el mensaje];
            P --> Q[El cliente recibe el mensaje a través del listener];
        end

        subgraph "Flujo de Mensaje Saliente"
            R[Cliente llama a `sock.sendMessage()`] --> S[Prepara y encripta el mensaje];
            S --> T[Envía el mensaje a través del WebSocket];
            T --> U[Servidor de WhatsApp procesa y entrega el mensaje];
        end
    end

    subgraph "Fase de Desconexión"
        V[Ocurre un error o cierre de conexión] --> W{Evento `connection.update` con estado 'close'};
        W --> X{¿Fue un cierre de sesión (`loggedOut`)?};
        X -- Sí --> Y[Fin de la conexión, requiere nuevo QR];
        X -- No (Error de red, etc.) --> Z[Intenta reconectar automáticamente];
        Z --> F;
    end
```

## Explicación del Flujo

1.  **Inicialización**: El proceso comienza cuando una aplicación cliente crea una instancia de Baileys usando `makeWASocket`. La librería comprueba si existen credenciales de una sesión anterior. Si es así, las carga; de lo contrario, se prepara para una nueva autenticación.

2.  **Conexión y Autenticación**:
    *   Se establece una conexión WebSocket con los servidores de WhatsApp.
    *   Si es una nueva sesión, se genera un código QR o un código de emparejamiento que el usuario debe escanear con su teléfono.
    *   Una vez autenticado, Baileys recibe las credenciales de sesión. Es fundamental guardarlas (`creds.update`) para poder restaurar la sesión en el futuro sin necesidad de volver a escanear el código.

3.  **Operación**:
    *   Con la conexión activa, la librería se pone en modo de escucha de eventos. Los eventos más importantes son `messages.upsert` (para nuevos mensajes) y `connection.update` (para cambios en el estado de la conexión).
    *   **Mensaje Entrante**: Cuando llega un mensaje, el evento `messages.upsert` se dispara. Baileys desencripta el contenido y lo entrega a la aplicación cliente.
    *   **Mensaje Saliente**: La aplicación cliente utiliza funciones como `sock.sendMessage()` para enviar mensajes. Baileys se encarga de encriptar el contenido y enviarlo a través del WebSocket.

4.  **Desconexión**:
    *   Si la conexión se cierra, el evento `connection.update` lo notifica.
    *   La librería analiza la causa. Si el usuario cerró la sesión (`DisconnectReason.loggedOut`), el proceso termina.
    *   Si fue por un error de red u otra causa recuperable, Baileys intentará reconectarse automáticamente para mantener la sesión activa.
