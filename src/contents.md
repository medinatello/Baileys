# Contenido de src

## Carpetas

- **Defaults/**: valores por defecto utilizados por la biblioteca.
- **Signal/**: implementación relacionada con el protocolo Signal utilizado por WhatsApp.
- **Socket/**: manejo de la conexión WebSocket y eventos.
- **Types/**: definiciones de tipos TypeScript.
- **Utils/**: utilidades comunes como manejo de logs y formatos.
- **WABinary/**: funciones para procesar nodos binarios del protocolo.
- **WAM/**: analítica y métricas de WhatsApp.
- **WAUSync/**: utilidades de sincronización con WA.
- **__tests__/**: pruebas automatizadas.

## Archivos

- **index.ts**: punto de entrada del módulo.
  - `makeWASocket(config)`: crea una instancia preparada para comunicarse con WhatsApp.
  - `WASocket`: tipo que representa la instancia retornada por `makeWASocket`.
  - Exportaciones adicionales: reexporta módulos de `WAProto`, `Utils`, `Types`, `Defaults`, `WABinary`, `WAM` y `WAUSync`.
