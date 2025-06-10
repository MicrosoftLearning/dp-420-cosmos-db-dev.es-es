---
lab:
  title: "Procesamiento por lotes de varias operaciones de punto junto con el SDK de Azure\_Cosmos\_DB for NoSQL"
  module: Module 4 - Access and manage data with the Azure Cosmos DB for NoSQL SDKs
---

# Procesamiento por lotes de varias operaciones de punto junto con el SDK de Azure Cosmos DB for NoSQL

Las clases [TransactionalBatch][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.transactionalbatch] y [TransactionalBatchResponse][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.transactionalbatchresponse] juntas son la clave para redactar y descomponer operaciones en un solo paso lógico. Con estas clases, puede escribir el código para realizar varias operaciones y, a continuación, determinar si se completaron correctamente en el lado servidor.

En este laboratorio, usará el SDK para realizar dos operaciones de doble elemento en las que intentará crear dos elementos como una sola unidad lógica.

## Preparación del entorno de desarrollo

Si aún no ha clonado el repositorio de código de laboratorio para **DP-420** al entorno en el que está trabajando en este laboratorio, siga estos pasos para hacerlo. De lo contrario, abra la carpeta clonada anteriormente en **Visual Studio Code**.

1. Inicie **Visual Studio Code**.

    > &#128221; Si aún no está familiarizado con la interfaz de Visual Studio Code, revise la [Documentación de introducción][code.visualstudio.com/docs/getstarted]

1. Abra la paleta de comandos y ejecute **Git: Clonar** para clonar el repositorio de GitHub ``https://github.com/microsoftlearning/dp-420-cosmos-db-dev`` en una carpeta local de su elección.

    > &#128161; Puedes usar el método abreviado de teclado **CTRL+MAYÚS+P** para abrir la paleta de comandos.

1. Una vez clonado el repositorio, abra la carpeta local que seleccionó en **Visual Studio Code**.

## Creación de una cuenta de Azure Cosmos DB for NoSQL y configuración del proyecto del SDK

1. Vaya a Azure Portal (``portal.azure.com``) desde una nueva ventana o pestaña del explorador web.

1. Inicia sesión en el portal con las credenciales de Microsoft asociadas a tu suscripción.

1. Selecciona **+ Crear un recurso**, busca *Cosmos DB* y, a continuación, crea un nuevo recurso de cuenta de **Azure Cosmos DB for NoSQL** con la siguiente configuración, dejando todas las opciones restantes en sus valores predeterminados:

    | **Configuración** | **Valor** |
    | ---: | :--- |
    | **Tipo de carga de trabajo** | **Aprendizaje** |
    | **Suscripción** | *Tu suscripción a Azure existente* |
    | **Grupo de recursos** | *Selecciona un grupo de recursos ya existente o crea un nuevo* |
    | **Nombre de cuenta** | *Escribe un nombre único global*. |
    | **Ubicación** | *Selecciona cualquier región disponible* |
    | **Capacity mode (Modo de capacidad)** | *Rendimiento aprovisionado* |
    | **Aplicación de descuento por nivel Gratis** | *No aplicar* |

    > &#128221; Es posible que los entornos de laboratorio tengan restricciones que te impidan crear un nuevo grupo de recursos. Si es así, usa el grupo de recursos existente creado previamente.

1. Espera a que se complete la tarea de implementación antes de continuar con esta tarea.

1. Ve al recurso de cuenta de **Azure Cosmos DB** recién creado y, después, ve al panel **Claves**.

1. Este panel contiene los detalles de conexión y las credenciales necesarias para conectarte a la cuenta desde el SDK. Específicamente:

    1. Observe el campo **URI**. Usará este valor de **endpoint** más adelante en este ejercicio.

    1. Observe el campo **PRIMARY KEY**. Usará este valor **key** más adelante en este ejercicio.

1. Vuelva a **Visual Studio Code**.

1. En el panel **Explorer**, vaya a la carpeta **07-sdk-batch**.

1. Abra el archivo de código **script.cs** dentro de la carpeta **07-sdk-batch**.

    > &#128221; La biblioteca de **[Microsoft.Azure.Cosmos][nuget.org/packages/microsoft.azure.cosmos/3.22.1]** ya se ha importado previamente desde NuGet.

1. Busque la variable de **string** denominada **endpoint**. Establezca su valor en el valor **endpoint** de la cuenta de Azure Cosmos DB que creó anteriormente.
  
    ```
    string endpoint = "<cosmos-endpoint>";
    ```

    > &#128221; Por ejemplo, si el punto de conexión es: **https&shy;://dp420.documents.azure.com:443/**, entonces la instrucción C# sería: **string endpoint = "https&shy;://dp420.documents.azure.com:443/";**.

1. Busque la variable **string** denominada **key**. Establezca su valor en la clave, **key**, de la cuenta de Azure Cosmos DB que creó anteriormente.

    ```
    string key = "<cosmos-key>";
    ```

    > &#128221; Por ejemplo, si la clave es: **fDR2ci9QgkkvERTQ==**, la instrucción C# sería: **string key = "fDR2ci9QgkdkvERTQ==";**.

1. **Guarde** el archivo de código **script.cs**.

1. Abra el menú contextual de la carpeta **07-sdk-batch** y, a continuación, seleccione **Abrir en terminal integrado** para abrir una nueva instancia de terminal.

    > &#128221; Este comando abrirá el terminal con el directorio inicial ya establecido en la carpeta **07-sdk-batch**.

1. Agregue el paquete [Microsoft.Azure.Cosmos][nuget.org/packages/microsoft.azure.cosmos/3.22.1] desde NuGet mediante el comando siguiente:

    ```
    dotnet add package Microsoft.Azure.Cosmos --version 3.49.0
    ```

1. Compile el proyecto con el comando [dotnet build][docs.microsoft.com/dotnet/core/tools/dotnet-build]:

    ```
    dotnet build
    ```

1. Cierre el terminal integrado.

## Crear un lote transaccional

En primer lugar, vamos a crear un sencillo lote transaccional que hace dos productos ficticios. Este lote insertará un sillín desgastado y un manillar oxidado en el contenedor con el mismo identificador de categoría "accesorios usados". Ambos elementos tienen la misma clave de partición lógica, lo que garantiza que tendremos una operación por lotes correcta.

1. Vuelva a la pestaña editor del archivo de código **script.cs**.

1. Cree una variable **Product** denominada **sillín** con un identificador único de **0120**, un nombre de **sillín desgastado** y un identificador de categoría de **9603ca6c-9e28-4a02-9194-51cdb7fea816**:

    ```
    Product saddle = new("0120", "Worn Saddle", "9603ca6c-9e28-4a02-9194-51cdb7fea816");
    ```

1. Cree una variable de **Producto** denominada **manillar** con un identificador único de **012A**, un nombre de **Manillar oxidado** y un identificador de categoría de **9603ca6c-9e28-4a02-9194-51cdb7fea816**:

    ```
    Product handlebar = new("012A", "Rusty Handlebar", "9603ca6c-9e28-4a02-9194-51cdb7fea816");
    ```

1. Cree una variable de tipo **PartitionKey** denominada **partitionKey** pasando **9603ca6c-9e28-4a02-9194-51cdb7fea816** como parámetro de constructor:

    ```
    PartitionKey partitionKey = new ("9603ca6c-9e28-4a02-9194-51cdb7fea816");
    ```

1. Invoque el método [CreateTransactionalBatch][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.createtransactionalbatch] de la variable **contenedor** pasando en la variable **partitionkey** como parámetro de método y usando la sintaxis fluida para invocar los métodos genéricos [CreateItem<>][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.transactionalbatch.createitem] pasar en las variables de **sillín** y **manillar** como elementos para crear en operaciones individuales y almacenar el resultado en una variable denominada **lote** de tipo **TransactionalBatch**:

    ```
    TransactionalBatch batch = container.CreateTransactionalBatch(partitionKey)
        .CreateItem<Product>(saddle)
        .CreateItem<Product>(handlebar);
    ```

1. Dentro de una instrucción using, invoque de forma asincrónica el método**ExecuteAsync** de la variable **batch** y almacene el resultado en una variable de tipo **TransactionalBatchResponse** con nombre **response**:

    ```
    using TransactionalBatchResponse response = await batch.ExecuteAsync();
    ```

1. Invoque el método **Console.WriteLine** estático para generar el valor de la propiedad **StatusCode** de la variable **response**:

    ```
    Console.WriteLine($"Status:\t{response.StatusCode}");
    ```

1. Una vez que haya terminado, el archivo de código debería incluir:
  
    ```
    using System;
    using Microsoft.Azure.Cosmos;
    
    string endpoint = "<cosmos-endpoint>";
    string key = "<cosmos-key>";
    
    CosmosClient client = new CosmosClient(endpoint, key);
        
    Database database = await client.CreateDatabaseIfNotExistsAsync("cosmicworks");
    Container container = await database.CreateContainerIfNotExistsAsync("products", "/categoryId", 400);

    Product saddle = new("0120", "Worn Saddle", "9603ca6c-9e28-4a02-9194-51cdb7fea816");
    Product handlebar = new("012A", "Rusty Handlebar", "9603ca6c-9e28-4a02-9194-51cdb7fea816");
    
    PartitionKey partitionKey = new ("9603ca6c-9e28-4a02-9194-51cdb7fea816");
    
    TransactionalBatch batch = container.CreateTransactionalBatch(partitionKey)
        .CreateItem<Product>(saddle)
        .CreateItem<Product>(handlebar);
    
    using TransactionalBatchResponse response = await batch.ExecuteAsync();
    
    Console.WriteLine($"Status:\t{response.StatusCode}");
    ```

1. **Guarde** el archivo de código **script.cs**.

1. En **Visual Studio Code**, abra el menú contextual de la carpeta **07-sdk-batch** y, a continuación, seleccione **Abrir en terminal integrado** para abrir una nueva instancia de terminal.

1. Compile y ejecute el proyecto mediante el comando **[dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run]**:

    ```
    dotnet run
    ```

1. Observe la salida del terminal. El código de estado debe ser **OK**.

1. Cierre el terminal integrado.

## Crear un lote transaccional errante

Ahora, vamos a crear un lote transaccional que producirá un error a propósito. Este lote intentará insertar dos elementos que tengan claves de partición lógica diferentes. Crearemos una luz estroboscópica parpadeante en la categoría "accesorios usados" y un nuevo casco en la categoría "accesorios impecables". Por definición, debe ser una solicitud incorrecta y devolver un error al realizar esta transacción.

1. Vuelva a la pestaña editor del archivo de código **script.cs**.

1. Elimine las siguientes líneas de código:

    ```
    Product saddle = new("0120", "Worn Saddle", "9603ca6c-9e28-4a02-9194-51cdb7fea816");
    Product handlebar = new("012A", "Rusty Handlebar", "9603ca6c-9e28-4a02-9194-51cdb7fea816");
    
    PartitionKey partitionKey = new ("9603ca6c-9e28-4a02-9194-51cdb7fea816");
    
    TransactionalBatch batch = container.CreateTransactionalBatch(partitionKey)
        .CreateItem<Product>(saddle)
        .CreateItem<Product>(handlebar);
    
    using TransactionalBatchResponse response = await batch.ExecuteAsync();
    
    Console.WriteLine($"Status:\t{response.StatusCode}");
    ```

1. Cree una variable **Producto** denominada **luz** con un identificador único de **012B**, un nombre de **Luz estroboscópica parpadeante** y un identificador de categoría de **9603ca6c-9e28-4a02-9194-51cdb7fea816**:

    ```
    Product light = new("012B", "Flickering Strobe Light", "9603ca6c-9e28-4a02-9194-51cdb7fea816");
    ```

1. Cree una variable **Producto** denominada **casco** con un identificador único de **012C**, un nombre de **Casco nuevo** y un identificador de categoría de **0feee2e4-687a-4d69-b64e-be36afc33e74**:

    ```
    Product helmet = new("012C", "New Helmet", "0feee2e4-687a-4d69-b64e-be36afc33e74");
    ```

1. Cree una variable de tipo **PartitionKey** denominada **partitionKey** pasando **9603ca6c-9e28-4a02-9194-51cdb7fea816** como parámetro de constructor:

    ```
    PartitionKey partitionKey = new ("9603ca6c-9e28-4a02-9194-51cdb7fea816");
    ```

1. Invoque el método **CreateTransactionalBatch** de la variable **contenedor** pasando la variable **partitionkey** como parámetro de método y usando la sintaxis fluida para invocar los métodos genéricos **CreateItem<>** pasar las variables **luz** y **casco** como elementos para crear en operaciones individuales y almacenar el resultado en una variable denominada **lote** de tipo **TransactionalBatch**:

    ```
    TransactionalBatch batch = container.CreateTransactionalBatch(partitionKey)
        .CreateItem<Product>(light)
        .CreateItem<Product>(helmet);
    ```

1. Dentro de una instrucción using, invoque de forma asincrónica el método**ExecuteAsync** de la variable **batch** y almacene el resultado en una variable de tipo **TransactionalBatchResponse** con nombre **response**:

    ```
    using TransactionalBatchResponse response = await batch.ExecuteAsync();
    ```

1. Invoque el método **Console.WriteLine** estático para generar el valor de la propiedad **StatusCode** de la variable **response**:

    ```
    Console.WriteLine($"Status:\t{response.StatusCode}");
    ```

1. Una vez que haya terminado, el archivo de código debería incluir:
  
    ```
    using System;
    using Microsoft.Azure.Cosmos;
    
    string endpoint = "<cosmos-endpoint>";
    string key = "<cosmos-key>";
    
    CosmosClient client = new CosmosClient(endpoint, key);
        
    Database database = await client.CreateDatabaseIfNotExistsAsync("cosmicworks");
    Container container = await database.CreateContainerIfNotExistsAsync("products", "/categoryId", 400);

    Product light = new("012B", "Flickering Strobe Light", "9603ca6c-9e28-4a02-9194-51cdb7fea816");
    Product helmet = new("012C", "New Helmet", "0feee2e4-687a-4d69-b64e-be36afc33e74");
    
    PartitionKey partitionKey = new ("9603ca6c-9e28-4a02-9194-51cdb7fea816");
    
    TransactionalBatch batch = container.CreateTransactionalBatch(partitionKey)
        .CreateItem<Product>(light)
        .CreateItem<Product>(helmet);
    
    using TransactionalBatchResponse response = await batch.ExecuteAsync();
    
    Console.WriteLine($"Status:\t{response.StatusCode}");
    ```

1. **Guarde** el archivo de código **script.cs**.

1. En **Visual Studio Code**, abra el menú contextual de la carpeta **07-sdk-batch** y, a continuación, seleccione **Abrir en terminal integrado** para abrir una nueva instancia de terminal.

1. Compile y ejecute el proyecto mediante el comando **[dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run]**:

    ```
    dotnet run
    ```

1. Observe la salida del terminal. El código de estado debe ser una **solicitud incorrecta** o un **conflicto**. Esto se ha producido porque todos los elementos de la transacción no comparten el mismo valor de clave de partición que el lote transaccional.

1. Cierre el terminal integrado.

1. Cierra **Visual Studio Code**.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.createtransactionalbatch]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.createtransactionalbatch
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.transactionalbatch]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.transactionalbatch
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.transactionalbatch.createitem]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.transactionalbatch.createitem
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.transactionalbatchresponse]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.transactionalbatchresponse
[docs.microsoft.com/dotnet/core/tools/dotnet-build]: https://docs.microsoft.com/dotnet/core/tools/dotnet-build
[docs.microsoft.com/dotnet/core/tools/dotnet-run]: https://docs.microsoft.com/dotnet/core/tools/dotnet-run
[nuget.org/packages/microsoft.azure.cosmos/3.22.1]: https://www.nuget.org/packages/Microsoft.Azure.Cosmos/3.22.1
