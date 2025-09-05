# Directorio `src/WAM`

Este directorio contiene la lógica para manejar **WAM (WhatsApp Analytics Metrics)**.

## Propósito

WAM es el sistema de analíticas y telemetría interno de WhatsApp. La aplicación oficial recopila una gran cantidad de métricas sobre el uso de la aplicación (ej. tiempo para enviar un mensaje, éxito de una llamada, uso de una función específica) y las envía a los servidores de WhatsApp.

Para que un cliente de terceros como Baileys no sea fácilmente distinguible de la aplicación oficial, debe simular el envío de estas métricas. Este directorio es el responsable de:

1.  **Definir los Eventos**: Especificar la estructura y los posibles valores de los diferentes eventos de analítica.
2.  **Codificar los Eventos**: Tomar los datos de un evento de analítica y codificarlos en un formato de buffer binario específico que los servidores de WhatsApp esperan.

Este módulo es principalmente un "codificador". No hay lógica para decodificar métricas, ya que el cliente solo las envía, no las recibe. La migración de esta funcionalidad a Go será importante para mantener la compatibilidad y reducir la probabilidad de que la conexión sea marcada como "inválida".
