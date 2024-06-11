---
lab:
  title: "Paginación de resultados de consultas de producto cruzado con el SDK de Azure Cosmos\_DB for NoSQL"
  module: Module 5 - Execute queries in Azure Cosmos DB for NoSQL
---

# Paginación de resultados de consultas de producto cruzado con el SDK de Azure Cosmos DB for NoSQL

En Azure Cosmos DB, las consultas tendrán normalmente varias páginas de resultados. La paginación se realiza automáticamente en el servidor cuando Azure Cosmos DB no puede devolver todos los resultados de la consulta en una sola ejecución. En muchas aplicaciones, querrá escribir código mediante el SDK para procesar los resultados de la consulta por lotes de una manera eficaz.

En este laboratorio, creará un iterador de fuente que se puede usar en un bucle para recorrer en iteración todo el conjunto de resultados.

## Preparación del entorno de desarrollo

Si aún no ha clonado el repositorio de código de laboratorio para **DP-420** al entorno en el que está trabajando en este laboratorio, siga estos pasos para hacerlo. De lo contrario, abra la carpeta clonada anteriormente en **Visual Studio Code**.

1. Inicie **Visual Studio Code**.

    > &#128221; Si aún no está familiarizado con la interfaz de Visual Studio Code, revise la [Guía de introducción para Visual Studio Code][code.visualstudio.com/docs/getstarted]

1. Abra la paleta de comandos y ejecute **Git: Clonar** para clonar el repositorio de GitHub ``https://github.com/microsoftlearning/dp-420-cosmos-db-dev`` en una carpeta local de su elección.

    > &#128161; Puede usar el método abreviado de teclado **CTRL+MAYÚS+P** para abrir la paleta de comandos.

1. Una vez clonado el repositorio, abra la carpeta local que seleccionó en **Visual Studio Code**.

## Creación de una cuenta de Azure Cosmos DB for NoSQL

Azure Cosmos DB es un servicio de base de datos NoSQL basado en la nube que admite varias API. Al aprovisionar una cuenta de Azure Cosmos DB por primera vez, seleccionará cuál de las API que quiere que admita la cuenta (por ejemplo, **API Mongo** o **API NoSQL**). Una vez que la cuenta de Azure Cosmos DB for NoSQL haya terminado de aprovisionar, puede recuperar el punto de conexión y la clave y usarlos para conectarse a la cuenta de Azure Cosmos DB for NoSQL mediante el SDK de Azure para .NET o cualquier otro SDK que prefiera.

1. Vaya a Azure Portal (``portal.azure.com``) desde una nueva ventana o pestaña del explorador web.

1. Inicie sesión en el portal con las credenciales de Microsoft asociadas a su suscripción.

1. Seleccione **+ Crear un recurso**, busque *Cosmos DB* y, a continuación, cree un nuevo recurso de cuenta de **Azure Cosmos DB for NoSQL** con la siguiente configuración, dejando todas las opciones restantes en sus valores predeterminados:

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

1. Vaya al recurso de cuenta de **Azure Cosmos DB** recién creado y vaya al panel **Claves**.

1. Este panel contiene los detalles de conexión y las credenciales necesarias para conectarse a la cuenta desde el SDK. Específicamente:

    1. Observe el campo **URI**. Usará este valor de **punto de conexión** más adelante en este ejercicio.

    1. Observe el campo **CLAVE PRINCIPAL**. Usará este valor de **clave** más adelante en este ejercicio.

1. Vuelva a **Visual Studio Code**.

## Inicialización de la cuenta de Azure Cosmos DB for NoSQL con datos

La herramienta de línea de comandos [cosmicworks][nuget.org/packages/cosmicworks] implementa datos de ejemplo en cualquier cuenta de Azure Cosmos DB for NoSQL La herramienta es de código abierto y está disponible a través de NuGet. Instalará esta herramienta en Azure Cloud Shell y la usará para inicializar la base de datos.

1. En **Visual Studio Code**, abra el menú **Terminal** y, a continuación, seleccione **Nuevo terminal** para abrir una nueva instancia de terminal.

1. Instale la herramienta de línea de comandos [cosmicworks][nuget.org/packages/cosmicworks] para su uso global en la máquina.

    ```
    dotnet tool install cosmicworks --global --version 1.*
    ```

    > &#128161; Este comando puede tardar un par de minutos en completarse. Este comando generará el mensaje de advertencia (*Tool "cosmicworks" ya está instalado) si ya ha instalado la versión más reciente de esta herramienta en el pasado.

1. Ejecute cosmicworks para inicializar la cuenta de Azure Cosmos DB con las siguientes opciones de línea de comandos:

    | **Opción** | **Valor** |
    | ---: | :--- |
    | **--endpoint** | *El valor del punto de conexión que copió anteriormente en este laboratorio* |
    | **--key** | *El valor de clave que copió anteriormente en este laboratorio* |
    | **--datasets** | *product* |

    ```
    cosmicworks --endpoint <cosmos-endpoint> --key <cosmos-key> --datasets product
    ```

    > &#128221; Por ejemplo, si el punto de conexión es: **https&shy;://dp420.documents.azure.com:443/** y la clave es: **fDR2ci9QgkdkvERTQ==**, el comando sería: ``cosmicworks --endpoint https://dp420.documents.azure.com:443/ --key fDR2ci9QgkdkvERTQ== --datasets product``

1. Espere a que el comando **cosmicworks**termine de poblar la cuenta con una base de datos, un contenedor y elementos.

1. Cierre el terminal integrado.

## Paginar a través de pequeños conjuntos de resultados de una consulta SQL mediante el SDK

Al procesar los resultados de la consulta, debe asegurarse de que el código avanza a través de todas las páginas de resultados y comprobaciones para ver si quedan más páginas antes de realizar solicitudes posteriores.

1. En **Visual Studio Code**, en el panel **Explorador**, vaya a la carpeta **10-paginate-results-sdk**.

1. Abra el archivo de código **product.cs**.

1. Observe la clase **Product** y sus propiedades correspondientes. En concreto, este laboratorio usará las propiedades **id**, **name** y **price**.

1. De nuevo en el panel **Explorador** de **Visual Studio Code**, abra el archivo de código **script.cs**.

1. Actualice la variable existente denominada **endpoint** con su valor establecido en el **punto de conexión** de la cuenta de Azure Cosmos DB que creó anteriormente.
  
    ```
    string endpoint = "<cosmos-endpoint>";
    ```

    > &#128221; Por ejemplo, si el punto de conexión es: **https&shy;://dp420.documents.azure.com:443/**, la instrucción C# sería: **string endpoint = "https&shy;://dp420.documents.azure.com:443/";**.

1. Actualice la variable existente denominada **key** con su valor establecido en la **clave** de la cuenta de Azure Cosmos DB que creó anteriormente.

    ```
    string key = "<cosmos-key>";
    ```

    > &#128221; Por ejemplo, si la clave es: **fDR2ci9QgkkvERTQ==**, la instrucción C# sería: **string key = "fDR2ci9QgkdkvERTQ==";**.

1. Cree una variable denominada **sql** de tipo *cadena* con un valor de **SELECT p.id, p.name, p.price FROM products p**:

    ```
    string sql = "SELECT p.id, p.name, p.price FROM products p ";
    ```

1. Cree una variable de tipo [QueryDefinition][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.querydefinition] pasando la variable **sql** como parámetro al constructor:

    ```
    QueryDefinition query = new (sql);
    ```

1. Cree una variable de tipo [QueryRequestOptions][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.queryrequestoptions] con nombre **options** con el constructor vacío predeterminado:

    ```
    QueryRequestOptions options = new ();
    ```

1. Establezca la propiedad [MaxItemCount][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.queryrequestoptions.maxitemcount] de la variable **options** a un valor de **50**:

    ```
    options.MaxItemCount = 50;
    ```

1. Cree una nueva variable denominada **iterador** de tipo [FeedIterator<>][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.feediterator-1] invocando el método genérico [GetItemQueryIterator][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.getitemqueryiterator] de la clase [Container][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container] pasando las variables **query** y **options** como parámetros:

    ```
    FeedIterator<Product> iterator = container.GetItemQueryIterator<Product>(query, requestOptions: options);
    ```

1. Cree un bucle **while** que compruebe la propiedad [HasMoreResults][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.feediterator-1.hasmoreresults] de la variable **iterator**:

    ```
    while (iterator.HasMoreResults)
    {
        
    }
    ```

1. Dentro del bucle **while**, invoque asíncronamente el método [ReadNextAsync][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.feediterator-1.readnextasync] de la variable **iterator** almacenando el resultado en una variable llamada **products** de tipo genérico [FeedResponse][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.feedresponse-1] utilizando la clase **Product**:

    ```
    FeedResponse<Product> products = await iterator.ReadNextAsync();
    ```

1. Aún dentro del bucle **while**, cree un nuevo bucle **foreach** iterando sobre la variable **products** utilizando la variable **product** para representar una instancia del tipo **Product**:

    ```
    foreach (Product product in products)
    {

    }
    ```

1. Dentro del bucle **foreach**, utilice el método estático incorporado **Console.WriteLine** para dar formato a las propiedades **id**, **name** y **price** de la variable **product** e imprimirlas:

    ```
    Console.WriteLine($"[{product.id}]\t[{product.name,40}]\t[{product.price,10}]");
    ```

1. De vuelta dentro del bucle **while**, utilice el método estático incorporado **Console.WriteLine** para imprimir el mensaje *Presione cualquier tecla para obtener más resultados*:

    ```
    Console.WriteLine("Press any key to get more results");
    ```

1. Aún dentro del bucle **while**, utilice el método estático **Console.ReadKey** incorporado para escuchar la entrada de la siguiente pulsación de tecla:

    ```
    Console.ReadKey();
    ```

1. Una vez que haya terminado, el archivo de código debería incluir:
  
    ```
    using System;
    using Microsoft.Azure.Cosmos;

    string endpoint = "<cosmos-endpoint>";

    string key = "<cosmos-key>";

    CosmosClient client = new CosmosClient(endpoint, key);

    Database database = await client.CreateDatabaseIfNotExistsAsync("cosmicworks");

    Container container = await database.CreateContainerIfNotExistsAsync("products", "/categoryId");

    string sql = "SELECT p.id, p.name, p.price FROM products p ";
    QueryDefinition query = new (sql);

    QueryRequestOptions options = new ();
    options.MaxItemCount = 50;

    FeedIterator<Product> iterator = container.GetItemQueryIterator<Product>(query, requestOptions: options);

    while (iterator.HasMoreResults)
    {
        FeedResponse<Product> products = await iterator.ReadNextAsync();
        foreach (Product product in products)
        {
            Console.WriteLine($"[{product.id}]\t[{product.name,40}]\t[{product.price,10}]");
        }

        Console.WriteLine("Press any key for next page of results");
        Console.ReadKey();        
    }
    ```

1. **Guarde** el archivo **script.cs**.

1. En **Visual Studio Code**, abra el menú contextual de la carpeta **10-paginate-results-sdk** y, a continuación, seleccione **Abrir en terminal integrado** para abrir una nueva instancia de terminal.

1. Compile y ejecute el proyecto mediante el comando [dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run]:

    ```
    dotnet run
    ```

1. El script generará ahora el primer conjunto de 50 elementos que coinciden con la consulta. Presione cualquier tecla para obtener el siguiente conjunto de 50 elementos hasta que la consulta se haya iterado sobre todos los elementos coincidentes.

    > &#128161; La consulta coincidirá con cientos de elementos del contenedor de productos.

1. Cierre el terminal integrado.

1. Cierre **Visual Studio Code**.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.getitemqueryiterator]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.getitemqueryiterator
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.feediterator-1]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.feediterator-1
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.feediterator-1.hasmoreresults]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.feediterator-1.hasmoreresults
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.feediterator-1.readnextasync]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.feediterator-1.readnextasync
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.feedresponse-1]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.feedresponse-1
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.querydefinition]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.querydefinition
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.queryrequestoptions]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.queryrequestoptions
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.queryrequestoptions.maxitemcount]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.queryrequestoptions.maxitemcount
[docs.microsoft.com/dotnet/core/tools/dotnet-run]: https://docs.microsoft.com/dotnet/core/tools/dotnet-run
[nuget.org/packages/cosmicworks]: https://www.nuget.org/packages/cosmicworks/
