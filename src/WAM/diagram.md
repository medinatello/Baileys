# Diagrama del Directorio `src/WAM`

Este diagrama ilustra el flujo de datos para la codificación de un evento de analítica (WAM), desde la creación de los datos del evento hasta la generación del buffer binario final.

```mermaid
graph TD
    subgraph "Lógica de la Aplicación"
        A[Creación de Evento de Analítica];
    end

    subgraph "Módulo WAM"
        B(BinaryInfo.ts) -- Define la estructura --> C[Objeto BinaryInfo];
        D(encode.ts) -- Utiliza --> E{constants.ts (IDs de Eventos y Campos)};

        C -- 1. Es pasado a --> D;
        D -- 2. Codifica a --> F[Buffer Binario de WAM];
    end

    subgraph "Módulo Socket"
        G[Socket] -- 3. Envía el buffer a WhatsApp --> H((Servidor de WhatsApp));
    end

    A --> C;
```

## Explicación del Flujo

El proceso es lineal y se centra en la codificación:

1.  **Creación de Datos del Evento**: En alguna parte de la aplicación, se decide que se debe registrar un evento de analítica (por ejemplo, "se ha enviado un mensaje de tipo imagen"). Se crea un objeto `BinaryInfo` (cuya clase se define en `BinaryInfo.ts`) y se puebla con los detalles de este evento.

2.  **Paso al Codificador**: Este objeto `BinaryInfo` se pasa a la función principal `encodeWAM` del archivo `encode.ts`.

3.  **Codificación**: La función `encodeWAM` recorre la estructura del objeto `BinaryInfo`.
    - Para cada evento y cada propiedad, busca su ID numérico correspondiente en los arrays `WEB_EVENTS` y `WEB_GLOBALS` definidos en `constants.ts`.
    - Utiliza la lógica en `serializeData` para convertir los valores del evento (cadenas, números) en su representación binaria más eficiente.
    - Ensambla todos estos bytes en un único `Buffer`.

4.  **Envío**: El `Buffer` binario resultante es devuelto al módulo `Socket`, que luego lo envuelve en un nodo `iq` (query) y lo envía a los servidores de WhatsApp.

En resumen, el módulo `WAM` actúa como un **serializador especializado**. Su única función es tomar datos de analítica estructurados y convertirlos al formato binario compacto que WhatsApp espera, utilizando un diccionario de constantes para mapear nombres de eventos y propiedades a IDs numéricos.
