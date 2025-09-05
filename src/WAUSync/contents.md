# Contenido del Directorio `src/WAUSync`

Este documento detalla el contenido de los archivos y subdirectorios dentro de `src/WAUSync`, que gestiona el protocolo de sincronización de datos de usuario.

## Archivos

- **`USyncQuery.ts`**:
  Define la clase `USyncQuery`, que funciona como un **constructor (Builder)** para crear solicitudes de sincronización.
  - **Lógica de Construcción**: Permite construir una consulta de manera fluida, añadiendo los protocolos de datos que se desean sincronizar (ej. `withContactProtocol()`, `withDeviceProtocol()`). También permite configurar el modo (`mode`) y el contexto (`context`) de la consulta.
  - **Lógica de Parseo**: Contiene el método `parseUSyncQueryResult(result)`, que toma la respuesta del servidor (`BinaryNode`) y la procesa. Itera sobre los protocolos que se solicitaron y utiliza el `parser` específico de cada uno para extraer y estructurar los datos de la respuesta.

- **`USyncUser.ts`**:
  Define la clase `USyncUser`. Esta clase probablemente se utiliza para construir la parte de la consulta de sincronización que especifica para qué usuario se están solicitando los datos y cuál es el último estado conocido (`version`) para cada protocolo, permitiendo así una sincronización diferencial.

- **`index.ts`**:
  Un archivo barril que re-exporta las clases y tipos principales del directorio.

## Subdirectorios

### `Protocols/`

Este directorio contiene la implementación de los diferentes "protocolos" de sincronización, donde cada uno se especializa en un tipo de dato. Cada archivo de protocolo define una clase que implementa una interfaz común (`USyncQueryProtocol`), la cual seguramente incluye:
- Un `name`: El nombre del protocolo (ej. "contact", "device").
- Un `version`: El último estado conocido para ese protocolo.
- Un `parser`: Una función que sabe cómo interpretar la sección de la respuesta del servidor correspondiente a ese protocolo.

**Archivos Principales dentro de `Protocols/`:**

- **`USyncContactProtocol.ts`**: Lógica para sincronizar la lista de contactos.
- **`USyncDeviceProtocol.ts`**: Lógica para sincronizar la lista de dispositivos vinculados a la cuenta.
- **`USyncDisappearingModeProtocol.ts`**: Lógica para sincronizar la configuración de los mensajes temporales.
- **`USyncStatusProtocol.ts`**: Lógica para sincronizar los estados de los contactos.
- **`UsyncBotProfileProtocol.ts`**: Lógica para sincronizar perfiles de bots.
- **`UsyncLIDProtocol.ts`**: Lógica para sincronizar datos relacionados con "LID" (probablemente "Login ID", identificadores de sesión).
- **`index.ts`**: Exporta todas las clases de protocolo para que `USyncQuery.ts` pueda importarlas fácilmente.
