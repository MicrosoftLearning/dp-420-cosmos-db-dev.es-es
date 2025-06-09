---
lab:
  title: Configuración de una directiva de índice de un contenedor de Azure Cosmos DB para NoSQL mediante el SDK
  module: Module 6 - Define and implement an indexing strategy for Azure Cosmos DB for NoSQL
---

# Configuración de una directiva de índice de un contenedor de Azure Cosmos DB para NoSQL mediante el SDK

Las directivas de indexación se pueden administrar desde cualquiera de los SDK de Azure Cosmos DB. El SDK de .NET incluye específicamente un conjunto de clases que se pueden usar para diseñar e insertar una nueva directiva de indexación en un contenedor de Azure Cosmos DB para NoSQL.

En este laboratorio, creará una directiva de indexación personalizada para un contenedor mediante el SDK de .NET

## Preparación del entorno de desarrollo

Si aún no ha clonado el repositorio de código de laboratorio para **DP-420** al entorno en el que está trabajando en este laboratorio, siga estos pasos para hacerlo. De lo contrario, abra la carpeta clonada anteriormente en **Visual Studio Code**.

1. Inicia **Visual Studio Code**.

    > &#128221; Si aún no estás familiarizado con la interfaz de Visual Studio Code, consulta la [Guía de introducción para Visual Studio Code][code.visualstudio.com/docs/getstarted]

1. Abre la paleta de comandos y ejecuta **Git: Clonar** para clonar el repositorio de GitHub ``https://github.com/microsoftlearning/dp-420-cosmos-db-dev`` en una carpeta local de tu elección.

    > &#128161; Puedes usar el método abreviado de teclado **CTRL+MAYÚS+P** para abrir la paleta de comandos.

1. Una vez clonado el repositorio, abra la carpeta local que seleccionó en **Visual Studio Code**.

## Creación de una cuenta de Azure Cosmos DB for NoSQL

Azure Cosmos DB es un servicio de base de datos NoSQL basado en la nube que admite varias API. Al aprovisionar una cuenta de Azure Cosmos DB por primera vez, seleccionará cuál de las API quiere que admita la cuenta (por ejemplo, **API de Mongo** o **API de NoSQL**). Una vez que la cuenta de Azure Cosmos DB for NoSQL haya terminado de aprovisionar, puede recuperar el punto de conexión y la clave y usarlos para conectarse a la cuenta de Azure Cosmos DB for NoSQL mediante el SDK de Azure para .NET o cualquier otro SDK que prefiera.

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

## Creación de una nueva directiva de indexación mediante el SDK de .NET

El SDK de .NET contiene un conjunto de clases relacionadas con la clase principal [Microsoft.Azure.Cosmos.IndexingPolicy ][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.indexingpolicy] para crear nuevas directivas de indexación en el código.

1. En el panel **Explorador**, vaya a la carpeta **12-custom-index-policy**.

1. Abra el archivo de código **script.cs**.

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

1. Cree una nueva variable de tipo [IndexingPolicy][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.indexingpolicy] denominada **directiva** con el constructor vacío predeterminado:

    ```
    IndexingPolicy policy = new ();
    ```

1. Establezca la propiedad [IndexingMode][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.indexingpolicy.indexingmode] de la variable de **directiva** en un valor de [IndexingMode.Consistent][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.indexingmode#fields]:

    ```
    policy.IndexingMode = IndexingMode.Consistent;
    ```

1. Agregue un nuevo objeto de tipo [ExcludedPath][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.excludedpath] con su propiedad [Path][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.excludedpath.path] establecida en un valor de **/*** a la propiedad de colección [ExcludedPaths][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.indexingpolicy.excludedpaths] de la variable de **directiva**:

    ```
    policy.ExcludedPaths.Add(
        new ExcludedPath{ Path = "/*" }
    );
    ```

1. Agregue un nuevo objeto de tipo [IncludedPath][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.includedpath] con su propiedad [Path][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.includedpath.path] establecida en un valor de **/name/?** a la propiedad de colección [IncludedPaths][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.indexingpolicy.includedpaths] en la variable de **directiva**:

    ```
    policy.IncludedPaths.Add(
        new IncludedPath{ Path = "/name/?" }
    );
    ```

1. Cree una variable de tipo [ContainerProperties][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.containerproperties] denominada **opciones** pasando los valores ``products`` y ``/categoryId`` como parámetros de constructor:

    ```
    ContainerProperties options = new ("products", "/category/name");
    ```

1. Asigne la variable de **directiva** a la propiedad [IndexingPolicy][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.containerproperties.indexingpolicy] de la variable **opciones**:

    ```
    options.IndexingPolicy = policy;
    ```

1. Invoque de forma asincrónica el método [CreateContainerIfNotExistsAsync][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.database.createcontainerifnotexistsasync] de la variable de **base de datos** pasando la variable **opciones** como parámetro de constructor y almacenando el resultado en una variable de tipo [Contenedor][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container] denominado **contenedor**:

    ```
    Container container = await database.CreateContainerIfNotExistsAsync(options);
    ```

1. Use el método estático **Console.WriteLine** integrado para imprimir la propiedad [Id][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.id] de la clase Container con un encabezado titulado **Contendor creado**:

    ```
    Console.WriteLine($"Container Created [{container.Id}]");
    ```

1. Una vez que haya terminado, el archivo de código debería incluir:
  
    ```
    using System;
    using Microsoft.Azure.Cosmos;

    string endpoint = "<cosmos-endpoint>";

    string key = "<cosmos-key>";

    CosmosClient client = new CosmosClient(endpoint, key);

    Database database = await client.CreateDatabaseIfNotExistsAsync("cosmicworks");
    
    IndexingPolicy policy = new ();
    policy.IndexingMode = IndexingMode.Consistent;
    policy.ExcludedPaths.Add(
        new ExcludedPath{ Path = "/*" }
    );
    policy.IncludedPaths.Add(
        new IncludedPath{ Path = "/name/?" }
    );

    ContainerProperties options = new ("products", "/category/name");
    options.IndexingPolicy = policy;

    Container container = await database.CreateContainerIfNotExistsAsync(options);
    Console.WriteLine($"Container Created [{container.Id}]");
    ```

1. **Guarde** el archivo **script.cs**.

1. En **Visual Studio Code**, abra el menú contextual de la carpeta **12-custom-index-policy** y, a continuación, seleccione **Abrir en terminal integrado** para abrir una nueva instancia de terminal.

1. Compile y ejecute el proyecto mediante el comando [dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run]:

    ```
    dotnet run
    ```

1. El script generará ahora el nombre del contenedor recién creado:

    ```
    Container Created [products]
    ```

1. Cierre el terminal integrado.

1. Vuelva al explorador web.

## Observe una directiva de indexación creada por el SDK de .NET mediante el Explorador de datos

Al igual que con cualquier otra directiva de indexación, puede usar el Explorador de datos para ver las directivas que insertó mediante los SDK de .NET. Ahora usará el portal para revisar la directiva que creó en este laboratorio desde el código.

1. En el recurso de cuenta de **Azure Cosmos DB**, vaya al panel del **Explorador de datos**.

1. En **Data Explorer**, expande el nodo de la base de datos **cosmicworks** y, a continuación, observa el nodo contenedor **products** nuevo dentro del árbol de navegación de la **API NOSQL**.

1. Dentro del nodo contenedor **products** en el árbol de navegación de la **API NOSQL**, selecciona **Configuración**.

1. Observe la directiva de indexación dentro de la sección **Directiva de indexación**:

    ```
    {
      "indexingMode": "consistent",
      "automatic": true,
      "includedPaths": [
        {
          "path": "/name/?"
        }
      ],
      "excludedPaths": [
        {
          "path": "/*"
        },
        {
          "path": "/\"_etag\"/?"
        }
      ]
    }
    ```

    > &#128221; Esta es la representación JSON de la directiva de indexación que creó mediante el SDK de .NET en este laboratorio.

1. Cierre la ventana o pestaña del explorador web.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.id]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.id
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.containerproperties]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.containerproperties
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.containerproperties.indexingpolicy]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.containerproperties.indexingpolicy
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.database.createcontainerifnotexistsasync]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.database.createcontainerifnotexistsasync
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.excludedpath]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.excludedpath
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.excludedpath.path]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.excludedpath.path
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.includedpath]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.includedpath
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.includedpath.path]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.includedpath.path
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.indexingmode#fields]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.indexingmode#fields
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.indexingpolicy]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.indexingpolicy
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.indexingpolicy.excludedpaths]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.indexingpolicy.excludedpaths
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.indexingpolicy.includedpaths]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.indexingpolicy.includedpaths
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.indexingpolicy.indexingmode]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.indexingpolicy.indexingmode
[docs.microsoft.com/dotnet/core/tools/dotnet-run]: https://docs.microsoft.com/dotnet/core/tools/dotnet-run
[nuget.org/packages/cosmicworks]: https://www.nuget.org/packages/cosmicworks/
