# Contenido del Directorio `src/Signal`

Este documento detalla el contenido de los archivos y subdirectorios dentro de `src/Signal`.

## Archivos

### `libsignal.ts`

Este archivo es el corazón del módulo Signal en Baileys. No es una reimplementación del protocolo, sino una **capa de adaptación (adapter)** que conecta la lógica de gestión de estado de Baileys (autenticación y claves) con la librería `libsignal-node`.

**Funciones y Miembros Principales:**

- **`makeLibSignalRepository(auth: SignalAuthState): SignalRepository`**:
  Esta es la función principal exportada. Recibe el objeto de estado de autenticación de Baileys (`auth`) y devuelve un objeto `SignalRepository`. Este repositorio es una interfaz que abstrae todas las operaciones de cifrado y descifrado.

- **`SignalRepository` (Objeto devuelto)**:
  Contiene los métodos que el resto de la aplicación (principalmente el `Socket`) utiliza para manejar la criptografía:
  - `decryptGroupMessage`: Descifra un mensaje de grupo.
  - `processSenderKeyDistributionMessage`: Procesa un mensaje que distribuye claves de sesión para un grupo.
  - `decryptMessage`: Descifra un mensaje individual (uno a uno).
  - `encryptMessage`: Cifra un mensaje individual.
  - `encryptGroupMessage`: Cifra un mensaje de grupo.
  - `injectE2ESession`: Inicia una sesión de cifrado con un nuevo dispositivo/contacto.

- **`signalStorage(auth: SignalAuthState)`**:
  Esta función interna es fundamental. Crea un objeto `store` que la librería `libsignal` utiliza para persistir y recuperar toda la información de la sesión criptográfica (claves de sesión, pre-claves, claves de identidad, etc.). La función mapea las llamadas de `libsignal` (ej. `loadSession`, `storeSession`) a las funciones del gestor de claves de Baileys (`auth.keys.get`, `auth.keys.set`). Es el pegamento entre la librería de cifrado y el sistema de almacenamiento de Baileys.

- **`jidToSignalProtocolAddress`** y **`jidToSignalSenderKeyName`**:
  Funciones de utilidad para convertir los "JIDs" (identificadores de WhatsApp) al formato de dirección que la librería `libsignal` espera.

## Carpetas

### `Group/`

Este subdirectorio contiene la implementación específica para el manejo del cifrado en los chats grupales, que sigue una lógica diferente al de los chats individuales (utiliza "Sender Keys" en lugar de sesiones pairwise).

**Archivos Principales dentro de `Group/`:**

- **`group_cipher.ts`**:
  Probablemente contiene la clase `GroupCipher`, que es la responsable de cifrar y descifrar los mensajes de un grupo utilizando las "Sender Keys" correspondientes.
- **`group-session-builder.ts`**:
  Contiene la clase `GroupSessionBuilder`, utilizada para construir y gestionar las sesiones de cifrado de un grupo, procesando las `SenderKeyDistributionMessage`.
- **`sender-key-distribution-message.ts`**:
  Define la estructura del mensaje que se utiliza para distribuir las claves de cifrado a los miembros de un grupo.
- **`sender-key-message.ts`**:
  Define la estructura de un mensaje cifrado para un grupo.
- **`sender-key-name.ts`**, **`sender-key-record.ts`**, **`sender-key-state.ts`**:
  Estos archivos definen las estructuras de datos para nombrar, almacenar y gestionar el estado de las "Sender Keys" para cada grupo/participante.
- **`index.ts`**:
  Probablemente exporta las clases y tipos más importantes del directorio para que puedan ser utilizados por `libsignal.ts`.
- **Otros archivos (`ciphertext-message.ts`, `keyhelper.ts`, `queue-job.ts`, `sender-chain-key.ts`, `sender-message-key.ts`)**:
  Son clases y utilidades de soporte que encapsulan diferentes partes del complejo proceso de gestión de claves de grupo, como la estructura del texto cifrado, ayudantes de claves, gestión de colas, y manejo de las cadenas de claves.
