---
lab:
  title: "04: Procesamiento por lotes de varias operaciones de punto junto con el SDK de Azure\_Cosmos\_DB for NoSQL"
  module: Perform cross-document transactional operations with the Azure Cosmos DB for NoSQL
---

# Procesamiento por lotes de varias operaciones de punto junto con el SDK de Azure Cosmos DB for NoSQL

La clase `TransactionalBatch` del SDK de JavaScript para Azure Cosmos DB proporciona funcionalidad para redactar y ejecutar operaciones por lotes dentro de la misma clave de partición lógica. Con esta característica, puedes realizar varias operaciones en una sola transacción y asegurarte de que se completan todas o ninguna de las operaciones.

En este laboratorio, usarás el SDK de JavaScript para realizar operaciones por lotes de doble elemento en las que intentarás crear dos elementos como una sola unidad lógica.

## Preparación del entorno de desarrollo

Si aún no has clonado el repositorio de código del laboratorio de **Compilación de copilotos con Azure Cosmos DB** y configurado el entorno local, consulta las instrucciones de [Configuración del entorno de laboratorio local](00-setup-lab-environment.md) para hacerlo.

## Creación de una cuenta de Azure Cosmos DB for NoSQL

Si ya has creado una cuenta de Azure Cosmos DB for NoSQL para los laboratorios de **Compilación de copilotos de Azure Cosmos DB** en este sitio, puedes usarla para este laboratorio y pasar a la [sección siguiente](#import-the-azurecosmos-library). De lo contrario, consulta las instrucciones de [Configuración de Azure Cosmos DB](../../common/instructions/00-setup-cosmos-db.md) para crear una cuenta de Azure Cosmos DB for NoSQL que usarás en todos los módulos de laboratorio y concede a tu identidad de usuario acceso para administrar los datos de la cuenta mediante la asignación al rol **Colaborador de datos integrado de Cosmos DB**.

## Importa la biblioteca @azure/cosmos

La biblioteca **@azure/cosmos** está disponible en **npm** para facilitar su instalación en los proyectos JavaScript.

1. En **Visual Studio Code**, en el panel **Explorer**, ve a la carpeta **javascript/04-sdk-batch**.

1. Abre el menú contextual de la carpeta **javascript/04-sdk-batch** y, a continuación, selecciona **Open in Integrated Terminal** para abrir una nueva instancia de terminal.

    > &#128221; Este comando abrirá el terminal con el directorio inicial ya establecido en la carpeta **javascript/04-sdk-batch**.

1. Inicia un nuevo proyecto Node.js:

    ```bash
    npm init -y
    ```

1. Instala el paquete [@azure/cosmos][npmjs.com/package/@azure/cosmos] ejecutando el siguiente comando:

    ```bash
    npm install @azure/cosmos
    ```

1. Instala la biblioteca [@azure/identity][npmjs.com/package/@azure/identity], que permite utilizar la autenticación de Azure para conectarse al espacio de trabajo de Azure Cosmos DB, mediante el siguiente comando:

    ```bash
    npm install @azure/identity
    ```

## Usa la biblioteca @azure/cosmos

Una vez importada la biblioteca de Azure Cosmos DB del SDK de Azure para JavaScript, puedes utilizar inmediatamente sus clases para conectarte a una cuenta de Azure Cosmos DB for NoSQL. La clase **CosmosClient** es la clase principal que se usa para establecer la conexión inicial a una cuenta de Azure Cosmos DB for NoSQL.

1. En **Visual Studio Code**, en el panel **Explorer**, ve a la carpeta **javascript/04-sdk-batch**.

1. Abre el archivo JavaScript vacío llamado **script.js**.

1. Agrega las siguientes instrucciones `require` para importar las bibliotecas **@azure/cosmos** y **@azure/identity**:

    ```javascript
    const { CosmosClient, BulkOperationType } = require("@azure/cosmos");
    const { DefaultAzureCredential  } = require("@azure/identity");
    process.env.NODE_TLS_REJECT_UNAUTHORIZED = 0
    ```

1. Agrega las variables denominadas **endpoint** y **credential** y establece el valor de **endpoint** en el **punto de conexión** de la cuenta de Azure Cosmos DB que creaste anteriormente. La variable **credential** debe establecerse en una nueva instancia de la clase **DefaultAzureCredential**:

    ```javascript
    const endpoint = "<cosmos-endpoint>";
    const credential = new DefaultAzureCredential();
    ```

    > &#128221; Por ejemplo, si el punto final es **https://dp420.documents.azure.com:443/**, la instrucción sería **const endpoint = "https://dp420.documents.azure.com:443/";**.

1. Agrega una nueva variable denominada **client** e inicialízala como una nueva instancia de la clase **CosmosClient** mediante las variables **endpoint** y **credential**:

    ```javascript
    const client = new CosmosClient({ endpoint, aadCredentials: credential });
    ```

1. Agrega el siguiente código para crear una base de datos y un contenedor si aún no existen:

    ```javascript
    async function main() {
        // Create database
        const { database } = await client.databases.createIfNotExists({ id: "cosmicworks" });
        
        // Create container
        const { container } = await database.containers.createIfNotExists({
            id: "products",
            partitionKey: { paths: ["/categoryId"] },
            throughput: 400
        });
    }
    
    main().catch((error) => console.error(error));
    ```

1. Tu archivo **script.js** debería tener ahora este aspecto:

    ```javascript
    const { CosmosClient, BulkOperationType } = require("@azure/cosmos");
    const { DefaultAzureCredential  } = require("@azure/identity");
    process.env.NODE_TLS_REJECT_UNAUTHORIZED = 0

    const endpoint = "<cosmos-endpoint>";
    const credential = new DefaultAzureCredential();

    const client = new CosmosClient({ endpoint, aadCredentials: credential });

    async function main() {
        // Create database
        const { database } = await client.databases.createIfNotExists({ id: "cosmicworks" });
        
        // Create container
        const { container } = await database.containers.createIfNotExists({
            id: "products",
            partitionKey: { paths: ["/categoryId"] },
            throughput: 400
        });
    }
    
    main().catch((error) => console.error(error));
    ```

1. **Guarda** el archivo **script.js**.

1. Antes de ejecutar el script, debes iniciar sesión en Azure mediante el comando `az login`. En la ventana de terminal, ejecuta lo siguiente:

    ```bash
    az login
    ```

1. Ejecuta el script para crear la base de datos y el contenedor:

    ```bash
    node script.js
    ```

1. Cambia a la ventana del explorador web.

1. En el recurso de cuenta de **Azure Cosmos DB**, ve al panel de **Data Explorer**.

1. En **Data Explorer**, expande el nodo de la base de datos **cosmicworks** y, a continuación, observa el nodo contenedor **products** nuevo dentro del árbol de navegación de la **API NoSQL**.

## Creación de un lote transaccional

En primer lugar, vamos a crear un sencillo lote transaccional que agrega dos productos ficticios al contenedor. Este lote insertará un sillín usado y un manillar oxidado en el contenedor con el mismo identificador de categoría "used accessories". Ambos elementos tienen la misma clave de partición lógica, lo que garantiza que tendremos una operación por lotes correcta.

1. Vuelve a **Visual Studio Code**. Si aún no está abierto, abre el archivo de código **script.js** dentro de la carpeta **javascript/04-sdk-batch**.

1. Define los dos productos que se van a insertar en el lote transaccional:

    ```javascript
    const saddle = { id: "0120", name: "Worn Saddle", categoryId: "9603ca6c-9e28-4a02-9194-51cdb7fea816" };
    const handlebar = { id: "012A", name: "Rusty Handlebar", categoryId: "9603ca6c-9e28-4a02-9194-51cdb7fea816" };
    ```

1. Crea un lote transaccional para la misma clave de partición lógica y agrega los elementos:

    ```javascript
    const partitionKey = "9603ca6c-9e28-4a02-9194-51cdb7fea816";
    const batch = container.items.batch(partitionKey)
        .create(saddle)
        .create(handlebar);
    ```

1. Ejecuta el lote e imprime el estado de la operación:

    ```javascript
    const response = await batch.execute();
    console.log(`Status: ${response.statusCode}`);
    ```

1. Una vez que hayas terminado, el archivo de código debería incluir:
  
    ```javascript
    const { CosmosClient, BulkOperationType } = require("@azure/cosmos");
    const { DefaultAzureCredential  } = require("@azure/identity");
    process.env.NODE_TLS_REJECT_UNAUTHORIZED = 0

    const endpoint = "<cosmos-endpoint>";
    const credential = new DefaultAzureCredential();

    const client = new CosmosClient({ endpoint, aadCredentials: credential });

    async function main() {
        // Create database
        const { database } = await client.databases.createIfNotExists({ id: "cosmicworks" });
            
        // Create container
        const { container } = await database.containers.createIfNotExists({
            id: "products",
            partitionKey: { paths: ["/categoryId"] },
            throughput: 400
        });
    
        const saddle = { id: "0120", name: "Worn Saddle", categoryId: "9603ca6c-9e28-4a02-9194-51cdb7fea816" };
        const handlebar = { id: "012A", name: "Rusty Handlebar", categoryId: "9603ca6c-9e28-4a02-9194-51cdb7fea816" };
    
        const partitionKey = "9603ca6c-9e28-4a02-9194-51cdb7fea816";
        const batch = [
            { operationType: BulkOperationType.Create, resourceBody: saddle },
            { operationType: BulkOperationType.Create, resourceBody: handlebar },
        ];
    
        try {
            const response = await container.items.batch(batch, partitionKey);
    
            response.result.forEach((operationResult, index) => {
                const { statusCode, requestCharge, resourceBody } = operationResult;
                console.log(`Operation ${index + 1}: Status code: ${statusCode}, Request charge: ${requestCharge}, Resource: ${JSON.stringify(resourceBody)}`);
            });
        } catch (error) {
            if (error.code === 400) {
                console.error("Bad Request: Check the structure of the batch.");
            } else if (error.code === 409) {
                console.error("Conflict: One of the items already exists.");
            } else if (error.code === 429) {
                console.error("Too Many Requests: Throttling limit reached.");
            } else {
                console.error(`Batch operation failed. Error code: ${error.code}, message: ${error.message}`);
            }
        }
    }
    
    main().catch((error) => console.error(error));
    ```

1. Vuelve a **guardar** y ejecutar el script:

    ```bash
    node script.js
    ```

1. La salida debe indicar un código de estado correcto para cada operación.

## Creación de un lote transaccional errante

Ahora, vamos a crear un lote transaccional que producirá un error a propósito. Este lote intentará insertar dos elementos que tengan claves de partición lógica diferentes. Crearemos una luz estroboscópica parpadeante en la categoría "used accessories" y un nuevo casco en la categoría "pristine accessories". Por definición, debe ser una solicitud incorrecta y devolver un error al realizar esta transacción.

1. Vuelve a la pestaña del editor del archivo de código **script.js**.

1. Elimina las siguientes líneas de código:

    ```javascript
    const saddle = { id: "0120", name: "Worn Saddle", categoryId: "9603ca6c-9e28-4a02-9194-51cdb7fea816" };
    const handlebar = { id: "012A", name: "Rusty Handlebar", categoryId: "9603ca6c-9e28-4a02-9194-51cdb7fea816" };

    const partitionKey = "9603ca6c-9e28-4a02-9194-51cdb7fea816";
    const batch = [
        { operationType: BulkOperationType.Create, resourceBody: saddle },
        { operationType: BulkOperationType.Create, resourceBody: handlebar },
    ];
    ```

1. Modifica el script para crear una nueva **luz estroboscópica parpadeante** y un **nuevo casco** con diferentes valores de clave de partición.

    ```javascript
    const light = { id: "012B", name: "Flickering Strobe Light", categoryId: "9603ca6c-9e28-4a02-9194-51cdb7fea816" };
    const helmet = { id: "012C", name: "New Helmet", categoryId: "0feee2e4-687a-4d69-b64e-be36afc33e74" };
    ```

1. Crea una variable de cadena denominada **partition_key** con un valor de **9603ca6c-9e28-4a02-9194-51cdb7fea816**:

    ```javascript
    const partitionKey = "9603ca6c-9e28-4a02-9194-51cdb7fea816";
    ```

1. Crea un nuevo lote con los elementos **light** y **helmet**:

    ```javascript
    const partitionKey = "9603ca6c-9e28-4a02-9194-51cdb7fea816";
    const batch = [
        { operationType: BulkOperationType.Create, resourceBody: light },
        { operationType: BulkOperationType.Create, resourceBody: helmet },
    ];
    ```

1. Una vez que hayas terminado, el archivo de código debería incluir:

    ```javascript
    const { CosmosClient, BulkOperationType } = require("@azure/cosmos");
    const { DefaultAzureCredential  } = require("@azure/identity");
    process.env.NODE_TLS_REJECT_UNAUTHORIZED = 0

    const endpoint = "<cosmos-endpoint>";
    const credential = new DefaultAzureCredential();

    const client = new CosmosClient({ endpoint, aadCredentials: credential });

    async function main() {
        // Create database
        const { database } = await client.databases.createIfNotExists({ id: "cosmicworks" });
            
        // Create container
        const { container } = await database.containers.createIfNotExists({
            id: "products",
            partitionKey: { paths: ["/categoryId"] },
            throughput: 400
        });
    
        const light = { id: "012B", name: "Flickering Strobe Light", categoryId: "9603ca6c-9e28-4a02-9194-51cdb7fea816" };
        const helmet = { id: "012C", name: "New Helmet", categoryId: "0feee2e4-687a-4d69-b64e-be36afc33e74" };
    
        const partitionKey = "9603ca6c-9e28-4a02-9194-51cdb7fea816";
        const batch = [
            { operationType: BulkOperationType.Create, resourceBody: light },
            { operationType: BulkOperationType.Create, resourceBody: helmet },
        ];
    
        try {
            const response = await container.items.batch(batch, partitionKey);
    
            response.result.forEach((operationResult, index) => {
                const { statusCode, requestCharge, resourceBody } = operationResult;
                console.log(`Operation ${index + 1}: Status code: ${statusCode}, Request charge: ${requestCharge}, Resource: ${JSON.stringify(resourceBody)}`);
            });
        } catch (error) {
            if (error.code === 400) {
                console.error("Bad Request: Check the structure of the batch.");
            } else if (error.code === 409) {
                console.error("Conflict: One of the items already exists.");
            } else if (error.code === 429) {
                console.error("Too Many Requests: Throttling limit reached.");
            } else {
                console.error(`Batch operation failed. Error code: ${error.code}, message: ${error.message}`);
            }
        }
    }
    
    main().catch((error) => console.error(error));
    ```

1. Vuelve a **guardar** y ejecutar el script:

    ```bash
    node script.js
    ```

1. Observa la salida del terminal. El código de estado de los elementos debe ser **424** para **dependencia con error** o **400** para **solicitud incorrecta**. Esto se ha producido porque todos los elementos de la transacción no comparten el mismo valor de clave de partición que el lote transaccional.

1. Cierra el terminal integrado.

1. Cierra **Visual Studio Code**.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[npmjs.com/package/@azure/cosmos]: https://www.npmjs.com/package/@azure/cosmos
[npmjs.com/package/@azure/identity]: https://www.npmjs.com/package/@azure/identity
