---
title: 02 - Configuración del SDK de JavaScript de Azure Cosmos DB para el desarrollo sin conexión
lab:
  title: 02 - Configuración del SDK de JavaScript de Azure Cosmos DB para el desarrollo sin conexión
  module: Configure the Azure Cosmos DB for NoSQL SDK
layout: default
nav_order: 5
parent: JavaScript SDK labs
---

# Configuración del SDK de JavaScript de Azure Cosmos DB para el desarrollo sin conexión

El emulador de Azure Cosmos DB es una herramienta local que emula el servicio Azure Cosmos DB para desarrollo y pruebas. El emulador es compatible con la API NoSQL y puede utilizarse en lugar del servicio en la nube al desarrollar código con el SDK de Azure para JavaScript.

En este laboratorio, te conectarás al emulador Azure Cosmos DB desde el SDK de Azure para JavaScript.

## Preparación del entorno de desarrollo

Si aún no has clonado el repositorio de código del laboratorio para **Compilación de copilotos con Azure Cosmos DB** y configurado el entorno local, consulta las instrucciones de [Configuración del entorno de laboratorio local](00-setup-lab-environment.md) para hacerlo.

## Iniciar el emulador de Azure Cosmos DB

Si usas un entorno de laboratorio hospedado, ya deberías tener instalado el emulador. Si no es así, consulte las [instrucciones de instalación](https://docs.microsoft.com/azure/cosmos-db/local-emulator) para instalar el emulador de Azure Cosmos DB. Una vez que el emulador se haya iniciado, puedes recuperar la cadena de conexión y utilizarla para conectarte al emulador mediante el SDK de Azure para JavaScript.

> &#128161; Opcionalmente, puedes instalar el [nuevo emulador de Azure Cosmos DB basado en Linux (en versión preliminar)](https://learn.microsoft.com/azure/cosmos-db/emulator-linux), que está disponible como contenedor Docker. Admite la ejecución en una amplia variedad de procesadores y sistemas operativos.

1. Inicie el **emulador de Azure Cosmos DB**.

    > 💡 Si usas Windows, el emulador de Azure Cosmos DB está anclado tanto a la barra de tareas de Windows como al menú Inicio. Si el emulador no se inicia desde los iconos anclados, intenta abrirlo haciendo doble clic en el archivo **C:\Program Files\Azure Cosmos DB Emulator\CosmosDB.Emulator.exe**.

1. Espera a que el emulador abra automáticamente el explorador predeterminado y ve a la página de aterrizaje **https://localhost:8081/_explorer/index.html**.

1. En el panel **Inicio rápido**, anota la **cadena de conexión principal**. Necesitarás esta cadena de conexión más adelante.

> &#128221; A veces, la página de aterrizaje no se carga correctamente, aunque el emulador se esté ejecutando. Si esto sucede, puedes usar la cadena de conexión conocida para conectarte al emulador. La cadena de conexión conocida es:`AccountEndpoint=https://localhost:8081/;AccountKey=C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==`

## Importa la biblioteca @azure/cosmos

La biblioteca **@azure/cosmos** está disponible en **npm** para facilitar su instalación en los proyectos JavaScript.

1. En **Visual Studio Code**, en el panel **Explorer**, navega hasta la carpeta **javascript/02-sdk-offline**.

1. Abre el menú contextual de la carpeta **javascript/02-sdk-offline** y selecciona **Open in Integrated Terminal** para abrir una nueva instancia de terminal.

    > 💡 Este comando abrirá el terminal con el directorio de inicio ya configurado en la carpeta **javascript/02-sdk-offline**.

1. Inicia un nuevo proyecto Node.js:

    ```bash
    npm init -y
    ```

1. Instale el paquete [@azure/cosmos][npmjs.com/package/@azure/cosmos] ejecutando el siguiente comando:

    ```bash
    npm install @azure/cosmos
    ```

## Conéctate al emulador desde el SDK de JavaScript

1. En **Visual Studio Code**, en el panel **Explorer**, navega hasta la carpeta **javascript/02-sdk-offline**.

1. Abre el archivo JavaScript en blanco llamado **script.js**.

1. Agrega el código siguiente para conectarte al emulador, crea una base de datos e imprime su id.:

    ```javascript
    const { CosmosClient } = require("@azure/cosmos");
    process.env.NODE_TLS_REJECT_UNAUTHORIZED = 0
    
    // Connection string for the Azure Cosmos DB Emulator
    const endpoint = "https://127.0.0.1:8081/";
    const key = "C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==";
    
    // Initialize the Cosmos client
    const client = new CosmosClient({ endpoint, key });
    
    async function main() {
        // Create a database
        const databaseName = "cosmicworks";
        const { database } = await client.databases.createIfNotExists({ id: databaseName });
    
        // Print the database ID
        console.log(`New Database: Id: ${database.id}`);
    }
    
    main().catch((error) => console.error(error));
    ```

1. **Guarda** el archivo **script.js**.

## Ejecute el script.

1. Usa la misma ventana de terminal en **Visual Studio Code** que usaste para instalar la biblioteca para este laboratorio. Si lo cierras, abre el menú contextual de la carpeta **javascript/02-sdk-offline** y selecciona **Open in Integrated Terminal** para abrir una nueva instancia de terminal.

1. Ejecute el script mediante el comando `node`:

    ```bash
    node script.js
    ```

1. El script crea una base de datos denominada `cosmicworks` en el emulador. Debería ver un resultado similar al siguiente:

    ```text
    New Database: Id: cosmicworks
    ```

## Creación y visualización de un nuevo contenedor

Puedes ampliar el script para crear un contenedor dentro de la base de datos.

### Código actualizado

1. Modifica el archivo `script.js` para **reemplazar** la siguiente línea al final del archivo (`main().catch((error) => console.error(error));`) y crear un contenedor:

```javascript
async function createContainer() {
    const containerName = "products";
    const partitionKeyPath = "/categoryId";
    const throughput = 400;

    const { container } = await client.database("cosmicworks").containers.createIfNotExists({
        id: containerName,
        partitionKey: { paths: [partitionKeyPath] },
        throughput: throughput
    });

    // Print the container ID
    console.log(`New Container: Id: ${container.id}`);
}

main()
    .then(createContainer)
    .catch((error) => console.error(error));
```

El archivo `script.js` ahora debería presentar un aspecto similar a este:

```javascript
const { CosmosClient } = require("@azure/cosmos");
process.env.NODE_TLS_REJECT_UNAUTHORIZED = 0

// Connection string for the Azure Cosmos DB Emulator
const endpoint = "https://127.0.0.1:8081/";
const key = "C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==";

// Initialize the Cosmos client
const client = new CosmosClient({ endpoint, key });

async function main() {
    // Create a database
    const databaseName = "cosmicworks";
    const { database } = await client.databases.createIfNotExists({ id: databaseName });

    // Print the database ID
    console.log(`New Database: Id: ${database.id}`);
}

async function createContainer() {
    const containerName = "products";
    const partitionKeyPath = "/categoryId";
    const throughput = 400;

    const { container } = await client.database("cosmicworks").containers.createIfNotExists({
        id: containerName,
        partitionKey: { paths: [partitionKeyPath] },
        throughput: throughput
    });

    // Print the container ID
    console.log(`New Container: Id: ${container.id}`);
}

main()
    .then(createContainer)
    .catch((error) => console.error(error));
```

### Ejecución del script actualizado

1. Ejecuta el script actualizado con el siguiente comando:

    ```bash
    node script.js
    ```

1. El script crea un contenedor denominado `products` en el emulador. Debería ver un resultado similar al siguiente:

    ```text
    New Database: Id: cosmicworks
    New Container: Id: products
    ```

### Verificación de los resultados

1. Cambia al navegador donde está abierto la instancia de Data Explorer del emulador.

1. Actualiza la **API de NoSQL** para observar la nueva base de datos **cosmicworks** y el contenedor de **products**.

## Parada del emulador de Azure Cosmos DB

Es importante detener el emulador cuando hayas terminado de usarlo para liberar los recursos el sistema. Sigue los pasos siguientes en base a tu sistema operativo:

### En macOS o Linux:

Si has iniciado el emulador en una ventana de terminal, sigue estos pasos:

1. Busca la ventana de terminal donde se ejecuta el emulador.

1. Presiona `Ctrl + C` para finalizar el proceso del emulador.

Como alternativa, si necesitas detener manualmente el proceso del emulador:

1. Abra una nueva ventana de terminal.

1. Para encontrar el proceso del emulador, usa el comando siguiente:

    ```bash
    ps aux | grep CosmosDB.Emulator
    ```

Identifica el **PID** (id. de proceso) del proceso del emulador en la salida. Usa el comando kill para finalizar el proceso del emulador:

```bash
kill <PID>
```

### En Windows:

1. Busca el icono del emulador de Azure Cosmos DB en la bandeja del sistema de Windows (cerca del reloj de la barra de tareas).

1. Haz clic con el botón derecho en el icono del emulador para abrir el menú contextual.

1. Selecciona **Salir** para apagar el emulador.

> 💡 Todas las instancias del emulador pueden tardar un minuto en salir.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[npmjs.com/package/@azure/cosmos]: https://www.npmjs.com/package/@azure/cosmos
