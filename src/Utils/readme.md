# Directorio `src/Utils`

Este directorio funciona como la "caja de herramientas" de la librería Baileys. Contiene una colección de funciones y clases de utilidad (`helpers`) que proporcionan lógica reutilizable para diversas tareas a lo largo del proyecto.

## Propósito

El objetivo principal de este directorio es evitar la duplicación de código y mantener los módulos principales (como `Socket` y `Signal`) más limpios y enfocados en su responsabilidad principal. La lógica que es genérica o que se necesita en múltiples lugares se extrae y se coloca aquí.

Los archivos dentro de `Utils` están organizados por dominio, de modo que cada archivo agrupa funciones relacionadas. Por ejemplo:
- `auth-utils.ts` contiene utilidades para el manejo de la autenticación.
- `crypto.ts` proporciona funciones criptográficas comunes.
- `messages-media.ts` se encarga de la lógica para manejar medios en los mensajes.

Al migrar el proyecto, esta carpeta será una fuente importante de algoritmos y lógica de negocio específicos que deberán ser replicados. Es una buena práctica mantener una separación similar en el nuevo proyecto en Go para conservar la modularidad y la claridad del código.
