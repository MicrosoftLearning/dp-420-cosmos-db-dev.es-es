---
lab:
  title: "Movimiento de varios documentos en masa con el SDK de Azure Cosmos\_DB for NoSQL"
  module: Module 4 - Access and manage data with the Azure Cosmos DB for NoSQL SDKs
---

# Movimiento de varios documentos en masa con el SDK de Azure Cosmos DB for NoSQL

La manera más fácil de aprender a realizar una operación masiva es intentar insertar muchos documentos en una cuenta de Azure Cosmos DB for NoSQL en la nube. Con las características masivas del SDK, esto se puede hacer con alguna ayuda secundaria del espacio de nombres [System.Threading.Tasks][docs.microsoft.com/dotnet/api/system.threading.tasks].

En este laboratorio, usará la biblioteca [Bogus][nuget.org/packages/bogus/33.1.1] de NuGet para generar datos ficticios y colocarlos en una cuenta de Azure Cosmos DB.

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

    1. Observe el campo **PRIMARY KEY**. Usará este valor de **clave** más adelante en este ejercicio.

1. Aún en el recurso de cuenta de **Azure Cosmos DB** recién creado, vaya al panel **Data Explorer**.

1. En **Data Explorer**, seleccione **Nuevo contenedor** y, a continuación, cree un contenedor con la siguiente configuración, dejando todas las opciones restantes en sus valores predeterminados:

    | **Configuración** | **Valor** |
    | ---: | :--- |
    | **Id. de base de datos** | *Crear nuevo* &vert; *`cosmicworks`* |
    | **Uso compartido del rendimiento entre contenedores** | *No seleccionar* |
    | **Id. de contenedor** | *`products`* |
    | **Clave de partición** | *`/categoryId`* |
    | **Rendimiento del contenedor** | *Escalabilidad automática* &vert; *`4000`* |

1. Vuelva a **Visual Studio Code**.

1. En el panel **Explorer**, vaya a la carpeta **08-sdk-bulk**.

1. Abra el archivo de código **script.cs** dentro de la carpeta **08-sdk-bulk**.

    > &#128221; La biblioteca **[Microsoft.Azure.Cosmos][nuget.org/packages/microsoft.azure.cosmos/3.22.1]** ya se ha importado previamente desde NuGet.

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

1. Abra el menú contextual de la carpeta **08-sdk-bulk** y, a continuación, seleccione **Abrir en terminal integrado** para abrir una nueva instancia de terminal.

    > &#128221; Este comando abrirá el terminal con el directorio inicial ya establecido en la carpeta **08-sdk-bulk**.

1. Agrega el paquete [Microsoft.Azure.Cosmos][nuget.org/packages/microsoft.azure.cosmos/3.49.0] desde NuGet mediante el comando siguiente:

    ```
    dotnet add package Microsoft.Azure.Cosmos --version 3.49.0
    ```

1. Compile el proyecto con el comando [dotnet build][docs.microsoft.com/dotnet/core/tools/dotnet-build]:

    ```
    dotnet build
    ```

1. Cierre el terminal integrado.

## Inserción masiva de veinticinco mil documentos

Vamos a "ir a por todas" e intentar insertar muchos documentos para ver cómo funciona. En nuestras pruebas internas, puede tardar aproximadamente entre 1 y 2 minutos si la máquina virtual del laboratorio y la cuenta de Azure Cosmos DB for NoSQL están relativamente cerca entre sí geográficamente.

1. Vuelva a la pestaña editor del archivo de código **script.cs**.

1. Cree una nueva instancia de la clase [CosmosClientOptions][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclientoptions] denominada **opciones** con la propiedad **AllowBulkExecution** establecida en **true**:

    ```
    CosmosClientOptions options = new () 
    { 
        AllowBulkExecution = true 
    };
    ```

1. Cree una nueva instancia de la clase **CosmosClient** denominada **cliente** pasando las variables **punto de conexión**, **clave** y **opciones** como parámetros de constructor:

    ```
    CosmosClient client = new (endpoint, key, options); 
    ```

1. Use el método [GetContainer][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclient.getcontainer] de la variable **cliente** para recuperar el contenedor existente con el nombre de la base de datos (*cosmicworks*) y el nombre del contenedor (*products*):

    ```
    Container container = client.GetContainer("cosmicworks", "products");
    ```

1. Use este código de ejemplo especial para generar **25 000** productos ficticios mediante la clase **Ficticio** de la biblioteca Bogus importada de NuGet.

    ```
    List<Product> productsToInsert = new Faker<Product>()
        .StrictMode(true)
        .RuleFor(o => o.id, f => Guid.NewGuid().ToString())
        .RuleFor(o => o.name, f => f.Commerce.ProductName())
        .RuleFor(o => o.price, f => Convert.ToDouble(f.Commerce.Price(max: 1000, min: 10, decimals: 2)))
        .RuleFor(o => o.categoryId, f => f.Commerce.Department(1))
        .Generate(25000);
    ```

    > &#128161; La biblioteca [Bogus][nuget.org/packages/bogus/33.1.1] es una biblioteca de código abierto que se usa para diseñar datos ficticios para probar aplicaciones de interfaz de usuario y es excelente para aprender a desarrollar aplicaciones de importación y exportación masivas.

1. Cree una nueva **List<>** genérica de tipo **Tarea** denominada **concurrentTasks**:

    ```
    List<Task> concurrentTasks = new List<Task>();
    ```

1. Cree un bucle forEach que recorrerá en iteración la lista de productos que se generaron anteriormente en esta aplicación:

    ```
    foreach(Product product in productsToInsert)
    {
    }
    ```

1. En el bucle forEach, cree una **Tarea** para insertar de forma asincrónica un producto en Azure Cosmos DB for NoSQL, asegurándose de especificar explícitamente la clave de partición y agregar la tarea a la lista de tareas denominadas **concurrentTasks**:

    ```
    concurrentTasks.Add(
        container.CreateItemAsync(product, new PartitionKey(product.categoryId))
    );   
    ```

1. Después del bucle forEach, espere asincrónicamente el resultado de **Task.WhenAll** en la variable **concurrentTasks**:

    ```
    await Task.WhenAll(concurrentTasks);
    ```

1. Use el método estático **Console.WriteLine** integrado para imprimir un mensaje estático de **tareas masivas completadas** en la consola:

    ```
    Console.WriteLine("Bulk tasks complete");
    ```

1. Una vez que haya terminado, el archivo de código debería incluir:
  
    ```
    using System;
    using System.Collections.Generic;
    using System.Threading.Tasks;
    using Bogus;
    using Microsoft.Azure.Cosmos;
    
    string endpoint = "<cosmos-endpoint>";
    string key = "<cosmos-key>";
    
    CosmosClientOptions options = new () 
    { 
        AllowBulkExecution = true 
    };
    
    CosmosClient client = new (endpoint, key, options);  
    
    Container container = client.GetContainer("cosmicworks", "products");
    
    List<Product> productsToInsert = new Faker<Product>()
        .StrictMode(true)
        .RuleFor(o => o.id, f => Guid.NewGuid().ToString())
        .RuleFor(o => o.name, f => f.Commerce.ProductName())
        .RuleFor(o => o.price, f => Convert.ToDouble(f.Commerce.Price(max: 1000, min: 10, decimals: 2)))
        .RuleFor(o => o.categoryId, f => f.Commerce.Department(1))
        .Generate(25000);
        
    List<Task> concurrentTasks = new List<Task>();
    
    foreach(Product product in productsToInsert)
    {    
        concurrentTasks.Add(
            container.CreateItemAsync(product, new PartitionKey(product.categoryId))
        );
    }
    
    await Task.WhenAll(concurrentTasks);   

    Console.WriteLine("Bulk tasks complete");
    ```

1. **Guarde** el archivo de código **script.cs**.

1. En **Visual Studio Code**, abra el menú contextual de la carpeta **08-sdk-bulk** y, a continuación, seleccione **Abrir en terminal integrado** para abrir una nueva instancia de terminal.

1. Compile y ejecute el proyecto mediante el comando **[dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run]**:

    ```
    dotnet run
    ```

1. La aplicación debe ejecutarse de forma silenciosa, debe tardar aproximadamente uno a dos minutos en ejecutarse antes de completarse de forma silenciosa.

1. Cierre el terminal integrado.

## Observe los resultados

Ahora que ha enviado 25 000 elementos a Azure Cosmos DB, echemos un vistazo a Data Explorer.

1.Vuelva al explorador web y vaya al panel **Data Explorer**.

1. En **Data Explorer**, expanda el nodo de la base de datos **cosmicworks** y, a continuación, observe el nodo contenedor **products** dentro del árbol de navegación de la **API NOSQL**.

1. Expanda el nodo **products** y, a continuación, seleccione el nodo **Elementos**. Observe la lista de elementos dentro del contenedor.

1. Seleccione el nodo contenedor **products** dentro del árbol de navegación de la **API NoSQL** y, a continuación, seleccione **Nueva consulta SQL**.

1. Elimine el contenido del área del editor.

1. Cree una nueva consulta SQL que devuelva un recuento de todos los documentos creados mediante la operación masiva:

    ```
    SELECT COUNT(1) FROM items
    ```

1. Seleccione **Ejecutar consulta**.

1. Observe el recuento de los elementos del contenedor.

1. Cierre la ventana o pestaña del explorador web.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclient.getcontainer]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclient.getcontainer
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclientoptions]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclientoptions
[docs.microsoft.com/dotnet/api/system.threading.tasks]: https://docs.microsoft.com/dotnet/api/system.threading.tasks
[docs.microsoft.com/dotnet/core/tools/dotnet-build]: https://docs.microsoft.com/dotnet/core/tools/dotnet-build
[docs.microsoft.com/dotnet/core/tools/dotnet-run]: https://docs.microsoft.com/dotnet/core/tools/dotnet-run
[nuget.org/packages/bogus/33.1.1]: https://www.nuget.org/packages/bogus/33.1.1
