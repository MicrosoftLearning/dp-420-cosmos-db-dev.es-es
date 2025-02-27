---
title: 03 - Creación y actualización de documentos con el SDK de Azure Cosmos DB for NoSQL
lab:
  title: 03 - Creación y actualización de documentos con el SDK de Azure Cosmos DB for NoSQL
  module: Implement Azure Cosmos DB for NoSQL point operations
layout: default
nav_order: 6
parent: JavaScript SDK labs
---

# Creación y actualización de documentos con el SDK de Azure Cosmos DB for NoSQL

La biblioteca `@azure/cosmos` incluye métodos para crear, recuperar, actualizar y eliminar (CRUD) elementos dentro de un contenedor Azure Cosmos DB for NoSQL. Juntos, estos métodos realizan algunas de las operaciones "CRUD" más comunes en distintos elementos dentro de los contenedores de la API NoSQL.

En este laboratorio, utilizarás el SDK de JavaScript para realizar operaciones CRUD cotidianas en un elemento dentro de un contenedor Azure Cosmos DB for NoSQL.

## Preparación del entorno de desarrollo

Si aún no has clonado el repositorio de código del laboratorio de **Compilación de copilotos con Azure Cosmos DB** y configurado el entorno local, consulta las instrucciones de [Configuración del entorno de laboratorio local](00-setup-lab-environment.md) para hacerlo.

## Creación de una cuenta de Azure Cosmos DB for NoSQL

Si ya creaste una cuenta de Azure Cosmos DB for NoSQL para los laboratorios de **Compilación de copilotos con Azure Cosmos DB** de este sitio, puedes usarla para este laboratorio y pasar a la [siguiente sección](#import-the-azurecosmos-library). De lo contrario, consulta las instrucciones de [Configuración de Azure Cosmos DB](../../common/instructions/00-setup-cosmos-db.md) para crear una cuenta de Azure Cosmos DB for NoSQL que usarás en todos los módulos de laboratorio y concede a tu identidad de usuario acceso para administrar los datos de la cuenta mediante la asignación al rol **Colaborador de datos integrado de Cosmos DB**.

## Importa la biblioteca @azure/cosmos

La biblioteca **@azure/cosmos** está disponible en **npm** para facilitar su instalación en los proyectos JavaScript.

1. En **Visual Studio Code**, en el panel **Explorer**, navega hasta la carpeta **javascript/03-sdk-crud**.

1. Abre el menú contextual de la carpeta **javascript/03-sdk-crud** y selecciona **Open in Integrated Terminal** para abrir una nueva instancia de terminal.

    > &#128221; Este comando abrirá el terminal con el directorio de inicio ya configurado en la carpeta **javascript/03-sdk-crud**.

1. Inicia un nuevo proyecto Node.js:

    ```bash
    npm init -y
    ```

1. Instale el paquete [@azure/cosmos][npmjs.com/package/@azure/cosmos] ejecutando el siguiente comando:

    ```bash
    npm install @azure/cosmos
    ```

1. Instala la biblioteca [@azure/identity][npmjs.com/package/@azure/identity], que permite utilizar la autenticación de Azure para conectarse al espacio de trabajo de Azure Cosmos DB, mediante el siguiente comando:

    ```bash
    npm install @azure/identity
    ```

## Usa la biblioteca @azure/cosmos

Una vez importada la biblioteca de Azure Cosmos DB del SDK de Azure para JavaScript, puedes utilizar inmediatamente sus clases para conectarte a una cuenta de Azure Cosmos DB for NoSQL. La clase **CosmosClient** es la clase principal que se usa para establecer la conexión inicial a una cuenta de Azure Cosmos DB for NoSQL.

1. En **Visual Studio Code**, en el panel **Explorer**, navega hasta la carpeta **javascript/03-sdk-crud**.

1. Abre el archivo JavaScript vacío llamado **script.js**.

1. Agrega las siguientes instrucciones `require` para importar las bibliotecas **@azure/cosmos** y **@azure/identity**:

    ```javascript
    const { CosmosClient } = require("@azure/cosmos");
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
    const { CosmosClient } = require("@azure/cosmos");
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

1. Cambie a la ventana del explorador web.

1. En el recurso de cuenta de **Azure Cosmos DB**, vaya al panel del **Explorador de datos**.

1. En **Data Explorer**, expanda el nodo de la base de datos **cosmicworks** y, a continuación, observe el nodo contenedor **products** nuevo dentro del árbol de navegación de la **API NOSQL**.

## Realización de operaciones de creación y lectura de puntos en elementos con el SDK

Ahora utilizarás el conjunto de métodos de la clase **Container** para realizar operaciones comunes en elementos dentro de un contenedor de API NoSQL.

1. Vuelva a **Visual Studio Code**. Si no está abierto, abre el archivo de código **script.js** dentro de la carpeta **javascript/03-sdk-crud**.

1. Crea un nuevo artículo de producto y asígnalo a una variable llamada **saddle** con las siguientes propiedades. Asegúrate de agregar el siguiente código en la función `main`:

    | Propiedad | Valor |
    | ---: | :--- |
    | **id** | *706cd7c6-db8b-41f9-aea2-0e0c7e8eb009* |
    | **categoryId** | *9603ca6c-9e28-4a02-9194-51cdb7fea816* |
    | **name** | *Road Saddle* |
    | **price** | *45.99d* |
    | **etiquetas** | *{ tan, new, crisp }* |

    ```javascript
    const saddle = {
        id: "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009",
        categoryId: "9603ca6c-9e28-4a02-9194-51cdb7fea816",
        name: "Road Saddle",
        price: 45.99,
        tags: ["tan", "new", "crisp"]
    };
    ```

1. Invoca el método [`create`](https://learn.microsoft.com/javascript/api/%40azure/cosmos/items?view=azure-node-latest#@azure-cosmos-items-create) de la clase **items** del contenedor, pasando la variable **saddle** como parámetro de método:

    ```javascript
    const { resource: item } = await container
        .items.create(saddle);
    ```

1. Una vez que hayas terminado, el archivo de código debería incluir:
  
    ```javascript
    const { CosmosClient } = require("@azure/cosmos");
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
    
        const saddle = {
            id: "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009",
            categoryId: "9603ca6c-9e28-4a02-9194-51cdb7fea816",
            name: "Road Saddle",
            price: 45.99,
            tags: ["tan", "new", "crisp"]
        };
    
        const { resource: item } = await container
                .items.create(saddle);
    }
    
    main().catch((error) => console.error(error));
    ```

1. Vuelve a **guardar** y ejecutar el script:

    ```bash
    node script.js
    ```

1. Observa el nuevo elemento en **Data Explorer**.

1. Vuelve a **Visual Studio Code**.

1. Vuelve a la pestaña del editor del archivo de código **script.js**.

1. Elimina las siguientes líneas de código:

    ```javascript
    const saddle = {
        id: "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009",
        categoryId: "9603ca6c-9e28-4a02-9194-51cdb7fea816",
        name: "Road Saddle",
        price: 45.99,
        tags: ["tan", "new", "crisp"]
    };

    const { resource: item } = await container
            .items.create(saddle);
    ```

1. Crea una variable de cadena denominada **item_id** con un valor de **706cd7c6-db8b-41f9-aea2-0e0c7e8eb009**:

    ```javascript
    const itemId = "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009";
    ```

1. Crea una variable de cadena denominada **partition_key** con un valor de **9603ca6c-9e28-4a02-9194-51cdb7fea816**:

    ```javascript
    const partitionKey = "9603ca6c-9e28-4a02-9194-51cdb7fea816";
    ```

1. Invoca el método [`read`](https://learn.microsoft.com/javascript/api/%40azure/cosmos/item?view=azure-node-latest#@azure-cosmos-item-read) de la clase **item** del contenedor, pasando las variables **itemId** y **partitionKey** como parámetros de método:

    ```javascript
    // Read the item
    const { resource: saddle } = await container.item(itemId, partitionKey).read();
    ```

    > &#128161; El método `read` permite realizar una operación de lectura puntual en un elemento del contenedor. El método requiere los parámetros `itemId` y `partitionKey` para identificar el elemento que se va a leer. En lugar de ejecutar una consulta mediante el lenguaje de consulta SQL de Cosmos DB para buscar el único elemento, el método `read` es más eficaz y rentable para recuperar un elemento único. Las lecturas puntuales pueden leer los datos directamente y no requieren que el motor de consultas procese la solicitud.

1. Imprime el objeto saddle mediante una cadena de salida con formato:

    ```javascript
    print(f'[{saddle["id"]}]\t{saddle["name"]} ({saddle["price"]})')
    ```

1. Una vez que hayas terminado, el archivo de código debería incluir:

    ```javascript
    const { CosmosClient } = require("@azure/cosmos");
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
    
        const itemId = "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009";
        const partitionKey = "9603ca6c-9e28-4a02-9194-51cdb7fea816";
    
        // Read the item
        const { resource: saddle } = await container.item(itemId, partitionKey).read();
    
        console.log(`[${saddle.id}]\t${saddle.name} (${saddle.price})`);
    }
    
    main().catch((error) => console.error(error));
    ```

1. Vuelve a **guardar** y ejecutar el script:

    ```bash
    node script.js
    ```

1. Observa la salida del terminal. En concreto, observa el texto de salida con formato con el identificador, el nombre y el precio del elemento.

## Realización de operaciones de actualización y eliminación de puntos con el SDK

Al aprender el SDK, no es raro usar una cuenta de Azure Cosmos DB en línea o el emulador para actualizar un elemento y alternar entre Data Explorer y el IDE que prefieras mientras realizas una operación y comprueba si se ha aplicado el cambio. Aquí, lo harás al actualizar y eliminar un elemento mediante el SDK.

1. Vuelve a la ventana o pestaña del explorador web.

1. En el recurso de cuenta de **Azure Cosmos DB**, ve al panel de **Data Explorer**.

1. En **Data Explorer**, expande el nodo de la base de datos **cosmicworks** y, a continuación, expande el nodo contenedor **products** nuevo dentro del árbol de navegación de la **API de NoSQL**.

1. Selecciona el nodo **Items**. Selecciona el único artículo dentro del contenedor y observa los valores de las propiedades **name** y **price** del artículo.

    | **Propiedad** | **Valor** |
    | ---: | :--- |
    | **Name** | *Road Saddle* |
    | **Price** | *$45.99* |

    > &#128221; En este momento, estos valores no deben haberse cambiado desde que creaste el elemento. Estos valores se cambiarán en este ejercicio.

1. Vuelve a **Visual Studio Code**. Vuelve a la pestaña del editor del archivo de código **script.js**.

1. Elimina la siguiente línea de código:

    ```javascript
    console.log(`[${saddle.id}]\t${saddle.name} (${saddle.price})`);
    ```

1. Cambia la variable **saddle** estableciendo el valor de la propiedad precio en **32.55**:

    ```javascript
    // Update the item
    saddle.price = 32.55;
    ```

1. Modifica de nuevo la variable **saddle** estableciendo el valor de la propiedad **name** en **Sillín para carretera LL**:

    ```javascript
    saddle.name = "Road LL Saddle";
    ```

1. Invoca el método [`replace`](https://learn.microsoft.com/javascript/api/%40azure/cosmos/item?view=azure-node-latest#@azure-cosmos-item-replace) de la clase **item** del contenedor, pasando la variable **saddle** como parámetro de método:

    ```javascript
    await container.item(saddle.id, partitionKey).replace(saddle);
    ```

1. Una vez que hayas terminado, el archivo de código debería incluir:

    ```javascript
    const { CosmosClient } = require("@azure/cosmos");
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
    
        const itemId = "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009";
        const partitionKey = "9603ca6c-9e28-4a02-9194-51cdb7fea816";
    
        // Read the item
        const { resource: saddle } = await container.item(itemId, partitionKey).read();

        // Update the item
        saddle.price = 32.55;
        saddle.name = "Road LL Saddle";
    
        await container.item(saddle.id, partitionKey).replace(saddle);
    }
    
    main().catch((error) => console.error(error));
    ```

1. Vuelve a **guardar** y ejecutar el script:

    ```bash
    node script.js
    ```

1. Vuelve a la ventana o pestaña del explorador web.

1. En el recurso de cuenta de **Azure Cosmos DB**, vaya al panel del **Explorador de datos**.

1. En el **Explorador de datos**, expanda el nodo de la base de datos **cosmicworks** y, a continuación, expanda el nodo contenedor **products** nuevo dentro del árbol de navegación de la **API NOSQL**.

1. Seleccione el nodo **Elementos**. Seleccione el único artículo dentro del contenedor y observe los valores de las propiedades **nombre** y **precio** del artículo.

    | **Propiedad** | **Valor** |
    | ---: | :--- |
    | **Nombre** | *Sillín para carretera LL* |
    | **Precio** | *32,55 $* |

    > &#128221; En este momento, estos valores deben haberse cambiado desde que ha observado el elemento.

1. Vuelva a **Visual Studio Code**. Vuelve a la pestaña del editor del archivo de código **script.js**.

1. Elimine las siguientes líneas de código:

    ```javascript
    // Read the item
    const { resource: saddle } = await container.item(itemId, partitionKey).read();

    // Update the item
    saddle.price = 32.55;
    saddle.name = "Road LL Saddle";

    await container.item(saddle.id, partitionKey).replace(saddle);
    ```

1. Invoca el método [`delete`](https://learn.microsoft.com/javascript/api/%40azure/cosmos/item?view=azure-node-latest#@azure-cosmos-item-delete) de la clase **item** del contenedor, pasando las variables **itemId** y **partitionKey** como parámetros de método:

    ```javascript
    // Delete the item
    await container.item(itemId, partitionKey).delete();
    ```

1. Guarda y vuelve a ejecutar el script

    ```bash
    node script.js
    ```

1. Cierre el terminal integrado.

1. Vuelva a la ventana o pestaña del explorador web.

1. En el recurso de cuenta de **Azure Cosmos DB**, vaya al panel del **Explorador de datos**.

1. En el **Explorador de datos**, expanda el nodo de la base de datos **cosmicworks** y, a continuación, expanda el nodo contenedor **products** nuevo dentro del árbol de navegación de la **API NOSQL**.

1. Seleccione el nodo **Elementos**. Observe que la lista de elementos ahora está vacía.

1. Cierre la ventana o pestaña del explorador web.

1. Cierre **Visual Studio Code**.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[npmjs.com/package/@azure/cosmos]: https://www.npmjs.com/package/@azure/cosmos
[npmjs.com/package/@azure/identity]: https://www.npmjs.com/package/@azure/identity
