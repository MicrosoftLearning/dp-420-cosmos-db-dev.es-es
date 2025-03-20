---
lab:
  title: "06: Paginación de resultados de consultas de producto cruzado con el SDK de Azure Cosmos\_DB for NoSQL"
  module: Author complex queries with the Azure Cosmos DB for NoSQL
---

# Paginación de resultados de consultas de producto cruzado con el SDK de Azure Cosmos DB for NoSQL

En Azure Cosmos DB, las consultas tendrán normalmente varias páginas de resultados. La paginación se realiza automáticamente en el servidor cuando Azure Cosmos DB no puede devolver todos los resultados de la consulta en una sola ejecución. En muchas aplicaciones, querrás escribir código mediante el SDK para procesar los resultados de la consulta por lotes de una manera eficaz.

En este laboratorio, crearás un iterador de fuente que se puede usar en un bucle para recorrer en iteración todo el conjunto de resultados.

## Preparación del entorno de desarrollo

Si aún no has clonado el repositorio de código del laboratorio de **Compilación de copilotos con Azure Cosmos DB** y configurado el entorno local, consulta las instrucciones de [Configuración del entorno de laboratorio local](00-setup-lab-environment.md) para hacerlo.

## Creación de una cuenta de Azure Cosmos DB for NoSQL

Si ya has creado una cuenta de Azure Cosmos DB for NoSQL para los laboratorios de **Compilación de copilotos de Azure Cosmos DB** en este sitio, puedes usarla para este laboratorio y pasar a la [sección siguiente](#create-azure-cosmos-db-database-and-container-with-sample-data). De lo contrario, consulta las instrucciones de [Configuración de Azure Cosmos DB](../../common/instructions/00-setup-cosmos-db.md) para crear una cuenta de Azure Cosmos DB for NoSQL que usarás en todos los módulos de laboratorio y concede a tu identidad de usuario acceso para administrar los datos de la cuenta mediante la asignación al rol **Colaborador de datos integrado de Cosmos DB**.

## Creación de un contenedor y una base de datos de Azure Cosmos DB con datos de ejemplo

Si ya has creado una base de datos de Azure Cosmos DB denominada **cosmicworks-full** y un contenedor dentro de ella denominada **products**, que se ha cargado previamente con datos de ejemplo, puedes usarla para este laboratorio y pasar a la [sección siguiente](#import-the-azurecosmos-library). De lo contrario, sigue los pasos siguientes para crear una nueva base de datos y un contenedor de ejemplo.

<details markdown=1>
<summary markdown="span"><strong>Haz clic para expandir o contraer los pasos para crear una base de datos y un contenedor con los datos de ejemplo.</strong></summary>

1. En el recurso de cuenta de **Azure Cosmos DB** recién creado, ve al panel **Data Explorer**.

1. En **Data Explorer**, selecciona **Iniciar inicio rápido** en la página principal.

1. En la ventana **Nuevo contenedor**, escribe los siguientes valores:

    - **Id. de base de datos**: `cosmicworks-full`
    - **Id. de contenedor**: `products`
    - **Clave de partición**: `/categoryId`
    - **Almacén analítico**: `Off`

1. Selecciona **Aceptar** para crear el contenedor nuevo. Este proceso tardará uno o dos minutos mientras crea los recursos y carga previamente el contenedor con datos de producto de ejemplo.

1. Mantén abierta la pestaña del explorador, ya que volveremos a ella más adelante.

1. Vuelve a **Visual Studio Code**.

</details>

## Importa la biblioteca @azure/cosmos

La biblioteca **@azure/cosmos** está disponible en **npm** para facilitar su instalación en los proyectos JavaScript.

1. En **Visual Studio Code**, en el panel **Explorer**, ve a la carpeta **javascript/06-sdk-pagination**.

1. Abre el menú contextual de la carpeta **javascript/06-sdk-pagination** y, a continuación, selecciona **Open in Integrated Terminal** para abrir una nueva instancia de terminal.

    > &#128221; Este comando abrirá el terminal con el directorio inicial ya establecido en la carpeta **javascript/06-sdk-pagination**.

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

## Iteración de los resultados de una consulta SQL mediante el SDK

Al procesar los resultados de la consulta, debes asegurarte de que el código avanza a través de todas las páginas de resultados y comprobaciones para ver si quedan más páginas antes de realizar solicitudes posteriores.

1. En **Visual Studio Code**, en el panel **Explorer**, ve a la carpeta **javascript/06-sdk-pagination**.

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

1. Crea un nuevo método denominado **paginateResults** y código para ejecutar ese método al ejecutar el script. Agregarás el código para consultar el contenedor dentro de este método:

    ```javascript
    async function paginateResults() {
        // Query the container
    }

    paginateResults().catch((error) => {
        console.error(error);
    });
    ```

1. Dentro del método **paginateResults**, agrega el código siguiente para conectarte a la base de datos y al contenedor que creaste anteriormente:

    ```javascript
    const database = client.database("cosmicworks-full");
    const container = database.container("products");
    ```

1. Crea una variable de cadena de consulta denominada `sql` con el valor de `SELECT * FROM products p`.

    ```javascript
    const sql = "SELECT * FROM products p";
    ```

1. Crea una variable denominada `options` y establécela en un objeto con la propiedad `enableCrossPartitionQuery` establecida en `true`. Esta propiedad permite enviar más de una solicitud para ejecutar la consulta en el servicio Azure Cosmos DB. Se necesita más de una solicitud si la consulta no tiene como ámbito un único valor de clave de partición. Establece la propiedad `maxItemCount` en `50` para limitar el número de elementos por página:

    ```javascript
    const options = {
        enableCrossPartitionQuery: true,
        maxItemCount: 50 // Set the maximum number of items per page
    };
    ```

1. Invoca el método [`query`](https://learn.microsoft.com/javascript/api/%40azure/cosmos/items?view=azure-node-latest#@azure-cosmos-items-query-1) con las variables `sql` y `options` como parámetros para el constructor. Este método devuelve un iterador que se puede usar para capturar la siguiente página de resultados:

    ```javascript
    const iterator = container.items.query(query, options);
    ```

1. Itera por los resultados paginados e imprime `id`, `name` y `price` de cada elemento. El método `iterator.getAsyncIterator` devuelve un iterador asincrónico que se puede usar para capturar el bucle `for await...of` para recuperar cada página de resultados. Este bucle controla automáticamente la iteración asincrónica en las páginas.

    ```javascript
    for await (const page of iterator.getAsyncIterator()) {
        page.resources.forEach(product => {
            console.log(`[${product.id}] ${product.name} $${product.price.toFixed(2)}`);
        });
    }
    ```

1. Tu archivo **script.js** debería tener ahora este aspecto:

    ```javascript
    const { CosmosClient } = require("@azure/cosmos");
    const { DefaultAzureCredential  } = require("@azure/identity");
    process.env.NODE_TLS_REJECT_UNAUTHORIZED = 0

    const endpoint = "<cosmos-endpoint>";
    const credential = new DefaultAzureCredential();

    const client = new CosmosClient({ endpoint, aadCredentials: credential });

    async function paginateResults() {
        const database = client.database("cosmicworks-full");
        const container = database.container("products");
        
        const query = "SELECT * FROM products WHERE products.price > 500";

        const options = {
            enableCrossPartitionQuery: true,
            maxItemCount: 50 // Set the maximum number of items per page
        };
        
        const iterator = container.items.query(query, options);

        for await (const page of iterator.getAsyncIterator()) {
            page.resources.forEach(product => {
                console.log(`[${product.id}] ${product.name} $${product.price.toFixed(2)}`);
            });
        }
    }
    
    paginateResults().catch((error) => {
        console.error(error);
    });
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

1. El script ahora generará páginas de 50 elementos a la vez.

    > &#128161; La consulta coincidirá con cientos de elementos del contenedor de productos.

1. Cierra el terminal integrado.

1. Cierra **Visual Studio Code**.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[npmjs.com/package/@azure/cosmos]: https://www.npmjs.com/package/@azure/cosmos
[npmjs.com/package/@azure/identity]: https://www.npmjs.com/package/@azure/identity
