---
lab:
  title: "Configuración de modelos de coherencia en el portal y el SDK de Azure\_Cosmos\_DB for NoSQL"
  module: Module 9 - Design and implement a replication strategy for Azure Cosmos DB for NoSQL
---

# Configuración de modelos de coherencia en el portal y el SDK de Azure Cosmos DB for NoSQL

El nivel de coherencia predeterminado para las nuevas cuentas de Azure Cosmos DB for NoSQL es la coherencia de la sesión. Esta configuración predeterminada se puede modificar para todas las solicitudes futuras. En un nivel de solicitud individual, puede ir un paso más allá y relajar el nivel de coherencia para esa solicitud específica.

En este laboratorio, configuraremos el nivel de coherencia predeterminado para una cuenta de Azure Cosmos DB for NoSQL y, a continuación, configuraremos un nivel de coherencia para una operación individual mediante el SDK.

## Preparación del entorno de desarrollo

Si aún no ha clonado el repositorio de código de laboratorio para **DP-420** al entorno en el que está trabajando en este laboratorio, siga estos pasos para hacerlo. De lo contrario, abra la carpeta clonada anteriormente en **Visual Studio Code**.

1. Inicie **Visual Studio Code**.

    > &#128221; Si aún no está familiarizado con la interfaz de Visual Studio Code, revise la [Documentación de introducción][code.visualstudio.com/docs/getstarted]

1. Abra la paleta de comandos y ejecute **Git: Clonar** para clonar el repositorio de GitHub ``https://github.com/microsoftlearning/dp-420-cosmos-db-dev`` en una carpeta local de su elección.

    > &#128161; Puede usar el método abreviado de teclado **CTRL+MAYÚS+P** para abrir la paleta de comandos.

1. Una vez clonado el repositorio, abra la carpeta local que seleccionó en **Visual Studio Code**.

## Creación de una cuenta de Azure Cosmos DB for NoSQL

Azure Cosmos DB es un servicio de base de datos NoSQL basado en la nube que admite varias API. Al aprovisionar una cuenta de Azure Cosmos DB por primera vez, seleccionará cuál de las API quiere que admita la cuenta (por ejemplo, **API de Mongo** o **API de NoSQL**). Una vez que la cuenta de Azure Cosmos DB for NoSQL haya terminado el aprovisionamiento, puede recuperar el punto de conexión y la clave, y usarlos para conectarse a la cuenta de Azure Cosmos DB for NoSQL mediante el SDK de Azure para .NET o de cualquier otro SDK que prefiera.

1. Vaya a Azure Portal (``portal.azure.com``) desde una nueva ventana o pestaña del explorador web.

1. Inicie sesión en el portal con las credenciales de Microsoft asociadas a su suscripción.

1. Seleccione **+ Crear un recurso**, busque *Cosmos DB* y, a continuación, cree un nuevo recurso de cuenta de **Azure Cosmos DB for NoSQL** con la siguiente configuración, dejando todas las opciones restantes en sus valores predeterminados:

    | **Configuración** | **Valor** |
    | ---: | :--- |
    | **Suscripción** | *Su suscripción de Azure existente* |
    | **Grupo de recursos** | *Seleccione un grupo de recursos ya existente o cree uno nuevo* |
    | **Account Name** | *Escriba un nombre único global*. |
    | **Ubicación** | *seleccione cualquier región disponible* |
    | **Capacity mode (Modo de capacidad)** | *Rendimiento aprovisionado* |
    | **Distribución global** &vert; **Replicación geográfica** | *Habilitar* |
    | **Aplicación de descuento por nivel Gratis** | *No aplicar* |

    > &#128221; Es posible que los entornos de laboratorio tengan restricciones que le impidan crear un nuevo grupo de recursos. Si es así, use el grupo de recursos existente creado previamente.

1. Espere a que se complete la tarea de implementación antes de continuar con esta tarea.

1. Vaya al recurso de cuenta de **Azure Cosmos DB** recién creado y vaya al panel **Replicar datos globalmente**.

1. En el panel **Replicar datos globalmente**, agregue dos regiones de lectura adicionales a la cuenta y, a continuación, proceda a **Guardar** los cambios.

    > &#128221; En un par de pasos, se le pedirá que cambie el nivel de coherencia a Fuerte, pero tenga en cuenta que una coherencia fuerte para las cuentas con regiones que abarcan más de 5000 millas (8000 kilómetros) está bloqueada de forma predeterminada debido a una latencia de escritura alta. Asegúrese de elegir regiones cercanas entre sí.  En un entorno de producción, para habilitar esta funcionalidad, póngase en contacto con el soporte técnico.

1. Espere a que se complete la tarea de replicación antes de continuar con esta tarea.

    > &#128221; Esta operación puede tardar aproximadamente entre 5 a 10 minutos, luego navegue hasta el panel **Coherencia predeterminada**.

1. En la hoja de recursos, vaya al panel **Coherencia predeterminada**.

1. En el panel **Coherencia predeterminada**, seleccione la opción **Fuerte** y, a continuación, seleccione **Guardar** los cambios.

1. Espere a que se conserve el cambio en el nivel de coherencia predeterminado antes de continuar con esta tarea.

1. En la hoja de recursos, vaya al panel **Data Explorer**.

1. En el panel **Data Explorer**, seleccione **Nuevo contenedor**.

1. En la ventana emergente **Nuevo contenedor**, escriba los valores siguientes para cada configuración y, después, seleccione **Aceptar**:

    | **Configuración** | **Valor** |
    | --: | :-- |
    | **Id. de base de datos** | *Crear nuevo* &vert; *``cosmicworks``* |
    | **Uso compartido del rendimiento entre contenedores** | *No seleccionar* |
    | **Id. de contenedor** | *``products``* |
    | **Clave de partición** | *``/categoryId``* |
    | **Rendimiento del contenedor** | *Manual* &vert; *400* |

1. De regreso en el panel **Data Explorer**, expanda el nodo de base de datos **cosmicworks** y observe el nodo contenedor **products** dentro de la jerarquía.

1. En **Data Explorer**, expanda el nodo de base de datos **cosmicworks**, expanda el nodo de contenedor **products** y, a continuación, seleccione **Elementos**.

1. En el panel **Data Explorer**, seleccione **Nuevo elemento** en la barra de comandos. En el editor, reemplace el elemento JSON de marcador de posición por el siguiente contenido:

    ```
    {
      "id": "7d9273d9-5d91-404c-bb2d-126abb6e4833",
      "categoryId": "78d204a2-7d64-4f4a-ac29-9bfc437ae959",
      "categoryName": "Components, Pedals",
      "sku": "PD-R563",
      "name": "ML Road Pedal",
      "price": 62.09
    }
    ```

1. Seleccione **Guardar** en la barra de comandos para agregar el elemento JSON:

1. En la pestaña **Elementos**, observe el nuevo elemento en el panel **Elementos**.

1. En la hoja de recursos, vaya al panel **Claves**.

1. Este panel contiene los detalles de conexión y las credenciales necesarias para conectarse a la cuenta desde el SDK. Específicamente:

    1. Observe el campo **URI**. Usará el valor **endpoint** más adelante en este ejercicio.

    1. Observe el campo **PRIMARY KEY**. Usará el valor **key** más adelante en este ejercicio.

1. Vuelva a **Visual Studio Code**.

## Conexión a la cuenta de Azure Cosmos DB for NoSQL desde el SDK

Con las credenciales de la cuenta recién creada, se conectará con las clases del SDK y creará una nueva instancia de contenedor y una base de datos. A continuación, usará Data Explorer para validar que las instancias existan en Azure Portal.

1. En el panel **Explorador**, vaya a la carpeta **21-sdk-consistency-model**.

1. Abra el menú contextual de la carpeta **21-sdk-consistency-model** y, a continuación, seleccione **Abrir en el terminal integrado** para abrir una nueva instancia del terminal.

    > &#128221; Este comando abrirá el terminal con el directorio inicial ya establecido en la carpeta **21-sdk-consistency-model**.

1. Compile el proyecto con el comando [dotnet build][docs.microsoft.com/dotnet/core/tools/dotnet-build]:

    ```
    dotnet build
    ```

    > &#128221; Es posible que vea una advertencia del compilador de que las variables **endpoint** y **key** están actualmente sin usar. Puede omitir esta advertencia de forma segura, ya que usará estas variables en esta tarea.

1. Cierre el terminal integrado.

1. Abra el archivo de código **product.cs**.

1. Observe el registro **Product** y sus propiedades correspondientes. En concreto, este laboratorio usará las propiedades **id**, **name**y **categoryId**.

1. De nuevo en el panel **Explorador** de **Visual Studio Code**, abra el archivo de código **script.cs**.

    > &#128221; La biblioteca **[Microsoft.Azure.Cosmos][nuget.org/packages/microsoft.azure.cosmos/3.22.1]** ya se ha importado previamente desde NuGet.

1. Busque la variable de **string** denominada **endpoint**. Establezca su valor en el **endpoint** de la cuenta de Azure Cosmos DB que creó anteriormente.
  
    ```
    string endpoint = "<cosmos-endpoint>";
    ```

    > &#128221; Por ejemplo, si el punto de conexión es: **https&shy;://dp420.documents.azure.com:443/**, entonces la instrucción C# sería: **string endpoint = "https&shy;://dp420.documents.azure.com:443/";**.

1. Busque la variable **string** denominada **key**. Establezca su valor en la clave, **key**, de la cuenta de Azure Cosmos DB que creó anteriormente.

    ```
    string key = "<cosmos-key>";
    ```

    > &#128221; Por ejemplo, si la clave es: **fDR2ci9QgkkvERTQ==**, la instrucción C# sería: **string key = "fDR2ci9QgkdkvERTQ==";**.

1. **Guarde** el archivo de código **script.cs**.

## Configuración del nivel de coherencia para una operación de punto

La clase **ItemRequestOptions** contiene propiedades de configuración por solicitud. Con esta clase, relajará el nivel de coherencia del valor predeterminado actual de fuerte a coherencia final.

1. Cree una variable de cadena denominada **id** con un valor de **7d9273d9-5d91-404c-bb2d-126abb6e4833**:

    ```
    string id = "7d9273d9-5d91-404c-bb2d-126abb6e4833";
    ```

1. Cree una variable de cadena denominada **categoryId** con un valor de **78d204a2-7d64-4f4a-ac29-9bfc437ae959**:

    ```
    string categoryId = "78d204a2-7d64-4f4a-ac29-9bfc437ae959";
    ```

1. Cree una variable de tipo **PartitionKey** denominada **partitionKey** pasando la variable **categoryId** como parámetro de constructor:

    ```
    PartitionKey partitionKey = new (categoryId);
    ```

1. Invoque de forma asincrónica el método genérico **ReadItemAsync\<\>** de la variable **container** pasando las variables **id** y **partitionkey** como parámetros de método, usando **Product** como tipo genérico y almacenando el resultado en una variable denominada **response** de tipo **ItemResponse\<Product\>**:

    ```
    ItemResponse<Product> response = await container.ReadItemAsync<Product>(id, partitionKey);
    ```

1. Invoque el método estático **Console.WriteLine** para imprimir el cargo de solicitud mediante una cadena de salida con formato:

    ```
    Console.WriteLine($"STRONG Request Charge:\t{response.RequestCharge:0.00} RUs");
    ```

1. Una vez que haya terminado, el archivo de código debería incluir:

    ```
    using Microsoft.Azure.Cosmos;

    string endpoint = "<cosmos-endpoint>";
    string key = "<cosmos-key>";

    CosmosClient client = new CosmosClient(endpoint, key);
    
    Container container = client.GetContainer("cosmicworks", "products");
    
    string id = "7d9273d9-5d91-404c-bb2d-126abb6e4833";
    
    string categoryId = "78d204a2-7d64-4f4a-ac29-9bfc437ae959";
    PartitionKey partitionKey = new (categoryId);
    
    ItemResponse<Product> response = await container.ReadItemAsync<Product>(id, partitionKey);
    
    Console.WriteLine($"STRONG Request Charge:\t{response.RequestCharge:0.00} RUs");
    ```

1. **Guarde** el archivo de código **script.cs**.

1. En **Visual Studio Code**, abra el menú contextual de la carpeta **21-sdk-consistency-model** y, a continuación, seleccione **Abrir en el terminal integrado** para abrir una nueva instancia del terminal.

1. Compile y ejecute el proyecto mediante el comando **[dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run]**:

    ```
    dotnet run
    ```

1. Observe la salida del terminal. El cargo de solicitud (en RU) debe imprimirse en la consola.

    > &#128221; El cargo de solicitud actual debe ser de **2 RU**. Esto se debe a que la coherencia fuerte requiere una lectura de al menos dos réplicas para asegurarse de que tiene la escritura más reciente.

1. Cierre el terminal integrado.

1. Vuelva a la pestaña editor del archivo de código **script.cs**.

1. Elimine las siguientes líneas de código:

    ```
    ItemResponse<Product> response = await container.ReadItemAsync<Product>(id, partitionKey);
    
    Console.WriteLine($"Request Charge:\t{response.RequestCharge:0.00} RUs");
    ```

1. Cree una nueva variable denominada **options** de tipo [ItemRequestOptions][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.itemrequestoptions] estableciendo la propiedad [ConsistencyLevel][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.itemrequestoptions.consistencylevel] en el valor de enumeración [ConsistencyLevel.Eventual][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.consistencylevel]:

    ```
    ItemRequestOptions options = new()
    { 
        ConsistencyLevel = ConsistencyLevel.Eventual 
    };
    ```

1. Invoque de forma asincrónica el método **ReadItemAsync\<\>** genérico de la variable **container** pasando las variables **id**, **partitionKey** y **options** como parámetros de método, usando **Product** como tipo genérico y almacenando el resultado en una variable denominada **response** de tipo **ItemResponse\<Product\>**:

    ```
    ItemResponse<Product> response = await container.ReadItemAsync<Product>(id, partitionKey, requestOptions: options);
    ```

1. Invoque el método estático **Console.WriteLine** para imprimir el cargo de solicitud mediante una cadena de salida con formato:

    ```
    Console.WriteLine($"EVENTUAL Request Charge:\t{response.RequestCharge:0.00} RUs");
    ```

1. Una vez que haya terminado, el archivo de código debería incluir:

    ```
    using Microsoft.Azure.Cosmos;

    string endpoint = "<cosmos-endpoint>";
    string key = "<cosmos-key>";

    CosmosClient client = new CosmosClient(endpoint, key);
    
    Container container = client.GetContainer("cosmicworks", "products");
    
    string id = "7d9273d9-5d91-404c-bb2d-126abb6e4833";
    
    string categoryId = "78d204a2-7d64-4f4a-ac29-9bfc437ae959";
    PartitionKey partitionKey = new (categoryId);

    ItemRequestOptions options = new()
    { 
        ConsistencyLevel = ConsistencyLevel.Eventual 
    };
    
    ItemResponse<Product> response = await container.ReadItemAsync<Product>(id, partitionKey, requestOptions: options);
    
    Console.WriteLine($"EVENTUAL Request Charge:\t{response.RequestCharge:0.00} RUs");
    ```

1. **Guarde** el archivo de código **script.cs**.

1. En **Visual Studio Code**, abra el menú contextual de la carpeta **21-sdk-consistency-model** y, a continuación, seleccione **Abrir en el terminal integrado** para abrir una nueva instancia del terminal.

1. Compile y ejecute el proyecto mediante el comando **[dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run]**:

    ```
    dotnet run
    ```

1. Observe la salida del terminal. El cargo de solicitud (en RU) debe imprimirse en la consola.

    > &#128221; El cargo de solicitud actual debe ser de **1 RU**. Esto se debe a que la coherencia final solo requiere una lectura de una sola réplica.

1. Cierre el terminal integrado.

1. Cierre **Visual Studio Code**.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.consistencylevel]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.consistencylevel
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.itemrequestoptions]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.itemrequestoptions
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.itemrequestoptions.consistencylevel]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.itemrequestoptions.consistencylevel
[docs.microsoft.com/dotnet/core/tools/dotnet-build]: https://docs.microsoft.com/dotnet/core/tools/dotnet-build
[docs.microsoft.com/dotnet/core/tools/dotnet-run]: https://docs.microsoft.com/dotnet/core/tools/dotnet-run
