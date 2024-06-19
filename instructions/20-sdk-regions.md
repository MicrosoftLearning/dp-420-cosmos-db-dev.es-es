---
lab:
  title: "Conexión a diferentes regiones con el SDK de Azure\_Cosmos\_DB for NoSQL"
  module: Module 9 - Design and implement a replication strategy for Azure Cosmos DB for NoSQL
---

# Conexión a diferentes regiones con el SDK de Azure Cosmos DB for NoSQL

Al habilitar la redundancia geográfica para una cuenta de Azure Cosmos DB for NoSQL, puede usar el SDK para leer datos de regiones en cualquier orden que configure. Esta técnica es beneficiosa al distribuir las solicitudes de lectura en todas las regiones de lectura disponibles.

En este laboratorio, configurará la clase CosmosClient para conectarse a regiones de lectura en un orden de reserva que configure manualmente.

## Preparación del entorno de desarrollo

Si aún no ha clonado el repositorio de código de laboratorio para **DP-420** al entorno en el que está trabajando en este laboratorio, siga estos pasos para hacerlo. De lo contrario, abra la carpeta clonada anteriormente en **Visual Studio Code**.

1. Inicie **Visual Studio Code**.

    > &#128221; Si aún no está familiarizado con la interfaz de Visual Studio Code, revise la [Documentación de introducción][code.visualstudio.com/docs/getstarted]

1. Abra la paleta de comandos y ejecute **Git: Clonar** para clonar el repositorio de GitHub ``https://github.com/microsoftlearning/dp-420-cosmos-db-dev`` en una carpeta local de su elección.

    > &#128161; Puede usar el método abreviado de teclado **CTRL+MAYÚS+P** para abrir la paleta de comandos.

1. Una vez clonado el repositorio, abra la carpeta local que seleccionó en **Visual Studio Code**.

## Creación de una cuenta de Azure Cosmos DB for NoSQL

Azure Cosmos DB es un servicio de base de datos NoSQL basado en la nube que admite varias API. Al aprovisionar una cuenta de Azure Cosmos DB por primera vez, seleccionará cuál de las API quiere que admita la cuenta (por ejemplo, **API de Mongo** o **API de NoSQL**). Una vez que la cuenta de Azure Cosmos DB for NoSQL haya terminado de aprovisionar, puede recuperar el punto de conexión y la clave y usarlos para conectarse a la cuenta de Azure Cosmos DB for NoSQL mediante el SDK de Azure para .NET o cualquier otro SDK que prefiera.

1. Vaya a Azure Portal (``portal.azure.com``) desde una nueva ventana o pestaña del explorador web.

1. Inicie sesión en el portal con las credenciales de Microsoft asociadas a su suscripción.

1. Seleccione **+ Crear un recurso**, busque *Cosmos DB* y, a continuación, cree un nuevo recurso de cuenta de **Azure Cosmos DB for NoSQL** con la siguiente configuración, dejando todas las opciones restantes en sus valores predeterminados:

    | **Configuración** | **Valor** |
    | ---: | :--- |
    | **Suscripción** | *Su suscripción de Azure existente* |
    | **Grupo de recursos** | *Seleccione un grupo de recursos ya existente o cree un nuevo* |
    | **Account Name** | *Escriba un nombre único global*. |
    | **Ubicación** | *seleccione cualquier región disponible* |
    | **Capacity mode (Modo de capacidad)** | *Rendimiento aprovisionado* |
    | **Aplicación de descuento por nivel Gratis** | *No aplicar* |

    > &#128221; Es posible que los entornos de laboratorio tengan restricciones que le impidan crear un nuevo grupo de recursos. Si es así, use el grupo de recursos existente creado previamente.

1. Espere a que se complete la tarea de implementación antes de continuar con esta tarea.

1. Vaya al recurso de cuenta de **Azure Cosmos DB** recién creado y vaya al panel **Replicar datos globalmente**.

1. En el panel **Replicar datos globalmente**, agregue dos regiones de lectura adicionales a la cuenta y, a continuación, proceda a **guardar** los cambios.

1. Espere a que se complete la tarea de replicación antes de continuar con esta tarea.

    > &#128221; Esta operación puede tardar aproximadamente entre 5 a 10 minutos.

1. Registre los nombres de la región de **escritura** (primaria) y las dos regiones de **lectura**. Usará estos nombres de región más adelante en este ejercicio.

    > &#128221; Por ejemplo, si la región primaria es **Norte de Europa** y las dos regiones secundarias de lectura son **Este de EE. UU. 2**y **Norte de Sudáfrica**, registrará los tres nombres tal como están.

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

    1. Observe el campo **URI**. Usará este valor de **endpoint** más adelante en este ejercicio.

    1. Observe el campo **PRIMARY KEY**. Usará este valor **key** más adelante en este ejercicio.

1. Vuelva a **Visual Studio Code**.

## Conexión a la cuenta de Azure Cosmos DB for NoSQL desde el SDK

Con las credenciales de la cuenta recién creada, se conectará con las clases del SDK y accederá a la base de datos y a la instancia de contenedor desde una región diferente.

1. En el panel **Explorer**, vaya a la carpeta **20-sdk-bulk**.

1. Abra el menú contextual de la carpeta **20-sdk-regions** y, a continuación, seleccione **Abrir en terminal integrado** para abrir una nueva instancia de terminal.

    > &#128221; Este comando abrirá el terminal con el directorio inicial ya establecido en la carpeta **20-sdk-regions**.

1. Compile el proyecto con el comando [dotnet build][docs.microsoft.com/dotnet/core/tools/dotnet-build]:

    ```
    dotnet build
    ```

    > &#128221; Es posible que vea una advertencia del compilador de que las variables **endpoint** y **key** están actualmente sin usar. Puede omitir esta advertencia de forma segura, ya que usará estas variables en esta tarea.

1. Cierre el terminal integrado.

1. Abra el archivo de código **script.cs** dentro de la carpeta **20-sdk-regions**.

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

## Configuración del SDK de .NET con una lista de regiones preferidas

La clase **CosmosClientOptions** incluye una propiedad para configurar la lista de regiones a las que desea conectarse con el SDK. La lista se ordena por prioridad de conmutación por error, intentando conectarse a cada región en el orden que configure.

1. Cree una nueva variable de tipo genérico **List\<string\>** que contenga una lista de las regiones que configuró con su cuenta, empezando por la tercera región y finalizando con la primera región (principal). Por ejemplo, si creó la cuenta de Azure Cosmos DB for NoSQL en la región **Oeste de EE. UU**, después, agregó **Norte de Sudáfrica** y, por último, agregó **Este de Asia**, la variable de lista contendrá:

    ```
    List<string> regions = new()
    {
        "East Asia",
        "South Africa North",
        "West US"
    };
    ```

    > &#128161; Como alternativa; puede usar la clase estática [Microsoft.Azure.Cosmos.Regions][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.regions], que incluye propiedades de cadena integradas para varias regiones de Azure.

1. Cree una nueva instancia de la clase **CosmosClientOptions** denominada **options** con la propiedad [ApplicationPreferredRegions][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclientoptions.applicationpreferredregions] establecida en la variable **regions**:

    ```
    CosmosClientOptions options = new () 
    { 
        ApplicationPreferredRegions = regions
    };
    ```

1. Cree una nueva instancia de la clase **CosmosClient** denominada **client** pasando las variables **endpoint**, **key** y **options** como parámetros constructor:

    ```
    using CosmosClient client = new (endpoint, key, options); 
    ```

1. Use el método **GetContainer** de la variable **client** para recuperar el contenedor existente con el nombre de la base de datos (*cosmicworks*) y el nombre del contenedor (*products*):

    ```
    Container container = client.GetContainer("cosmicworks", "products");
    ```

1. Use el método [ReadItemAsync][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.readitemasync] de la variable **container** para recuperar un elemento específico del servidor y almacenar el resultado en una variable denominada **response** del tipo [ItemResponse][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.itemresponse] que admite un valor NULL:

    ```
    ItemResponse<dynamic> response = await container.ReadItemAsync<dynamic>(
        "7d9273d9-5d91-404c-bb2d-126abb6e4833",
        new PartitionKey("78d204a2-7d64-4f4a-ac29-9bfc437ae959")
    );
    ```

1. Invoque el método static **Console.WriteLine** para imprimir el identificador de elemento actual y los datos de diagnóstico JSON:

    ```
    Console.WriteLine($"Item Id:\t{response.Resource.Id}");
    Console.WriteLine($"Response Diagnostics JSON");
    Console.WriteLine($"{response.Diagnostics}");
    ```

1. Una vez que haya terminado, el archivo de código debería incluir:
  
    ```
    using Microsoft.Azure.Cosmos;

    string endpoint = "<cosmos-endpoint>";
    string key = "<cosmos-key>";

    List<string> regions = new()
    {
        "<read-region-2>",
        "<read-region-1>",
        "<write-region>"
    };
    
    CosmosClientOptions options = new () 
    { 
        ApplicationPreferredRegions = regions
    };
    
    using CosmosClient client = new(endpoint, key, options);
    
    Container container = client.GetContainer("cosmicworks", "products");
    
    ItemResponse<dynamic> response = await container.ReadItemAsync<dynamic>(
        "7d9273d9-5d91-404c-bb2d-126abb6e4833",
        new PartitionKey("78d204a2-7d64-4f4a-ac29-9bfc437ae959")
    );
    
    Console.WriteLine($"Item Id:\t{response.Resource.Id}");
    Console.WriteLine("Response Diagnostics JSON");
    Console.WriteLine($"{response.Diagnostics}");
    ```

1. **Guarde** el archivo de código **script.cs**.

1. En **Visual Studio Code**, abra el menú contextual de la carpeta **20-sdk-regions** y, a continuación, seleccione **Abrir en terminal integrado** para abrir una nueva instancia de terminal.

1. Compile y ejecute el proyecto mediante el comando **[dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run]**:

    ```
    dotnet run
    ```

1. Observe la salida del terminal. El nombre del contenedor y los datos de diagnóstico JSON deben imprimirse en la salida de la consola.

1. Revise los datos de diagnóstico JSON. Busque una propiedad denominada **HttpResponseStats** y una propiedad secundaria denominada **RequestUri**. El valor de esta propiedad debe ser un URI que incluya el nombre y la región que configuró anteriormente en este laboratorio.

    > &#128221; Por ejemplo, si el nombre de la cuenta es: **dp420** y la primera región que configuró es **Este de Asia**, el valor de la propiedad JSON sería: **dp420-eastasia.documents.azure.com/dbs/cosmicworks/colls/products**.

1. Cierre el terminal integrado.

1. Cierre **Visual Studio Code**.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.readitemasync]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.readitemasync
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.itemresponse]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.itemresponse
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclientoptions.applicationpreferredregions]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclientoptions.applicationpreferredregions
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.regions]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.regions
[docs.microsoft.com/dotnet/core/tools/dotnet-build]: https://docs.microsoft.com/dotnet/core/tools/dotnet-build
[docs.microsoft.com/dotnet/core/tools/dotnet-run]: https://docs.microsoft.com/dotnet/core/tools/dotnet-run
