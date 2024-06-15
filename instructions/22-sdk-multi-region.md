---
lab:
  title: "Conectarse a una cuenta de escritura de varias regiones con el SDK de Azure Cosmos\_DB for NoSQL"
  module: Module 9 - Design and implement a replication strategy for Azure Cosmos DB for NoSQL
---

# Conectarse a una cuenta de escritura de varias regiones con el SDK de Azure Cosmos DB for NoSQL

La clase **CosmosClientBuilder** es una clase fluida diseñada para compilar el cliente del SDK para conectarse al contenedor y realizar operaciones. Con el generador, puede configurar una región de aplicación preferida para las operaciones de escritura si la cuenta de Azure Cosmos DB for NoSQL ya está configurada para escrituras en varias regiones.

En este laboratorio, configurará una cuenta de Azure Cosmos DB for NoSQL con varias regiones y habilitará escrituras en varias regiones. Después, usará el SDK para realizar operaciones en una región específica.

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

1. En el panel **Replicar datos globalmente**, agregue al menos una región adicional a la cuenta.

1. Aún dentro del panel **Replicar datos globalmente**, habilite **Escrituras en varias regiones** y, a continuación, **Guarde** los cambios.

1. Espere a que se complete la tarea de replicación antes de continuar con esta tarea.

    > &#128221; Esta operación puede tardar aproximadamente entre 5 a 10 minutos.

1. Observe al menos una de las regiones adicionales que ha creado. Usará este valor de región más adelante en este ejercicio.

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

1. En la hoja de recursos, vaya al panel **Claves**.

1. Este panel contiene los detalles de conexión y las credenciales necesarias para conectarse a la cuenta desde el SDK. Específicamente:

    1. Observe el campo **URI**. Usará este valor de **endpoint** más adelante en este ejercicio.

    1. Observe el campo **PRIMARY KEY**. Usará este valor **key** más adelante en este ejercicio.

1. Vuelva a **Visual Studio Code**.

## Conexión a la cuenta de Azure Cosmos DB for NoSQL desde el SDK

Con las credenciales de la cuenta recién creada, se conectará con las clases del SDK y creará una nueva instancia de contenedor y una base de datos. A continuación, usará Data Explorer para validar que las instancias existan en Azure Portal.

1. En el panel **Explorador**, vaya a la carpeta **22-sdk-multi-region**.

1. Abra el menú contextual de la carpeta **22-sdk-multi-region** y, a continuación, seleccione **Abrir en el terminal integrado** para abrir una nueva instancia de terminal.

    > &#128221; Este comando abrirá el terminal con el directorio inicial ya establecido en la carpeta **22-sdk-multi-region**.

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

## Configuración de la región de escritura para el SDK

El método fluido **WithApplicationRegion** se usa para configurar la región preferida para las operaciones posteriores mediante la clase generadora.

1. Cree una nueva instancia de la clase [CosmosClientBuilder][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.fluent.cosmosclientbuilder] denominada **builder** pasando las variables **endpoint** y **key** como parámetros de constructor:

    ```
    CosmosClientBuilder builder = new (endpoint, key);
    ```

1. Cree una variable denominada **region** de tipo **string** con el nombre de la región adicional que creó anteriormente en el laboratorio. Por ejemplo, si creó la cuenta de Azure Cosmos DB for NoSQL en la región **Este de EE. UU.** y, a continuación, agregó **Sur de Brasil**, la variable de cadena contendrá:

    ```
    string region = "Brazil South"; 
    ```

    > &#128161; Como alternativa; puede usar la clase estática [Microsoft.Azure.Cosmos.Regions][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.regions], que incluye propiedades de cadena integradas para varias regiones de Azure.

1. Invoque el método [WithApplicationRegion][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.fluent.cosmosclientbuilder.withapplicationregion] con un parámetro de **region** y el método [Build][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.fluent.cosmosclientbuilder.build] fluidamente en la variable **builder** que almacena el resultado en una variable denominada **client** de tipo **CosmosClient** que se encapsula dentro de una instrucción Using:

    ```
    using CosmosClient client = builder
        .WithApplicationRegion(region)
        .Build();
    ```

1. Use el método **GetContainer** de la variable **client** para recuperar el contenedor existente con el nombre de la base de datos (*cosmicworks*) y el nombre del contenedor (*products*):

    ```
    Container container = client.GetContainer("cosmicworks", "products");
    ```

1. Cree dos variables de **string** denominadas **id** y **categoryId** mediante la generación de un nuevo valor **Guid** y, a continuación, almacene el resultado como una cadena:

    ```
    string id = $"{Guid.NewGuid()}";
    string categoryId = $"{Guid.NewGuid()}";
    ```

1. Cree una variable denominada **item** de tipo **Product** pasando en la variable **id** un valor de cadena de **Polished Bike Frame** y la variable **categoryId** como parámetros de constructor:

    ```
    Product item = new (id, "Polished Bike Frame", categoryId);
    ```

1. Invoque de forma asincrónica el método **CreateItemAsync\<\>** de la variable **container** pasando la variable **item** como parámetro y almacenando el resultado en una variable denominada **response**:

    ```
    var response = await container.CreateItemAsync<Product>(item);
    ```

1. Invoque el método estático **Console.WriteLine** para imprimir el código de estado HTTP de la respuesta y solicitar la carga (en unidades de solicitud):

    ```
    Console.WriteLine($"Status Code:\t{response.StatusCode}");
    Console.WriteLine($"Charge (RU):\t{response.RequestCharge:0.00}");
    ```

1. Una vez que haya terminado, el archivo de código debería incluir:

    ```
    using Microsoft.Azure.Cosmos;
    using Microsoft.Azure.Cosmos.Fluent;

    string endpoint = "<cosmos-endpoint>";
    string key = "<cosmos-key>";    

    CosmosClientBuilder builder = new (endpoint, key);            
    
    string region = "West Europe";
    
    using CosmosClient client = builder
        .WithApplicationRegion(region)
        .Build();
    
    Container container = client.GetContainer("cosmicworks", "products");
    
    string id = $"{Guid.NewGuid()}";
    string categoryId = $"{Guid.NewGuid()}";
    Product item = new (id, "Polished Bike Frame", categoryId);
    
    var response = await container.CreateItemAsync<Product>(item);
    
    Console.WriteLine($"Status Code:\t{response.StatusCode}");
    Console.WriteLine($"Charge (RU):\t{response.RequestCharge:0.00}");
    ```

1. **Guarde** el archivo de código **script.cs**.

1. En **Visual Studio Code**, abra el menú contextual de la carpeta **22-sdk-multi-region** y, a continuación, seleccione **Abrir en el terminal integrado** para abrir una nueva instancia de terminal.

1. Compile y ejecute el proyecto mediante el comando **[dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run]**:

    ```
    dotnet run
    ```

1. Observe la salida del terminal. El código de estado HTTP y el cargo de solicitud (en RU) deben imprimirse en la consola.

1. Cierre el terminal integrado.

1. Cierre **Visual Studio Code**.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.fluent.cosmosclientbuilder]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.fluent.cosmosclientbuilder
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.fluent.cosmosclientbuilder.build]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.fluent.cosmosclientbuilder.build
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.fluent.cosmosclientbuilder.withapplicationregion]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.fluent.cosmosclientbuilder.withapplicationregion
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.regions]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.regions
[docs.microsoft.com/dotnet/core/tools/dotnet-build]: https://docs.microsoft.com/dotnet/core/tools/dotnet-build
[docs.microsoft.com/dotnet/core/tools/dotnet-run]: https://docs.microsoft.com/dotnet/core/tools/dotnet-run
