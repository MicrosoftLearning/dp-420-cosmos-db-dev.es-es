---
lab:
  title: "Creación y actualización de documentos con el SDK de Azure\_Cosmos\_DB for NoSQL"
  module: Module 4 - Access and manage data with the Azure Cosmos DB for NoSQL SDKs
---

# Creación y actualización de documentos con el SDK de Azure Cosmos DB for NoSQL

La clase [Microsoft.Azure.Cosmos.Container][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container] incluye un conjunto de métodos miembro para crear, recuperar, actualizar y eliminar elementos dentro de un contenedor de Azure Cosmos DB para NoSQL. Juntos, estos métodos realizan algunas de las operaciones "CRUD" más comunes en varios elementos dentro de los contenedores de api de NoSQL.

En este laboratorio, utilizará el SDK para realizar operaciones CRUD cotidianas en un elemento dentro de un contenedor Azure Cosmos DB para NoSQL.

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

1. Vaya al recurso de cuenta de **Azure Cosmos DB** recién creado y vaya al panel **Claves**.

1. Este panel contiene los detalles de conexión y las credenciales necesarias para conectarse a la cuenta desde el SDK. Específicamente:

    1. Observe el campo **URI**. Usará este valor de **endpoint** más adelante en este ejercicio.

    1. Observe el campo **PRIMARY KEY**. Usará el valor **key** más adelante en este ejercicio.

1. Vuelva a **Visual Studio Code**.

## Conexión a la cuenta de Azure Cosmos DB for NoSQL desde el SDK

Con las credenciales de la cuenta recién creada, se conectará con las clases del SDK y creará una nueva instancia de contenedor y una base de datos. A continuación, usará Data Explorer para validar que las instancias existan en Azure Portal.

1. En **Visual Studio Code**, en el panel **Explorador**, busque la carpeta **06-sdk-crud**.

1. Abra el menú contextual de la carpeta **06-sdk-bulk** y, a continuación, seleccione **Abrir en terminal integrado** para abrir una nueva instancia de terminal.

    > &#128221; Este comando abrirá el terminal con el directorio inicial ya establecido en la carpeta **06-sdk-bulk**.

1. Agregue el paquete [Microsoft.Azure.Cosmos][nuget.org/packages/microsoft.azure.cosmos/3.22.1] desde NuGet mediante el comando siguiente:

    ```
    dotnet add package Microsoft.Azure.Cosmos --version 3.22.1
    ```

1. Compile el proyecto con el comando [dotnet build][docs.microsoft.com/dotnet/core/tools/dotnet-build]:

    ```
    dotnet build
    ```

1. Cierre el terminal integrado.

1. Abra el archivo de código **script.cs** dentro de la carpeta **06-sdk-crud**.

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

1. Invoque de forma asíncrona el método CreateDatabaseIfNotExistsAsync de la variable **client** pasando el nombre de la nueva base de datos (**cosmicworks**) que desea crear, y almacenando el resultado en una variable de tipo **Database**:

    ```
    Database database = await client.CreateDatabaseIfNotExistsAsync("cosmicworks");
    ```

1. Invoque de forma asincrónica el método **CreateContainerIfNotExistsAsync** de la **base de datos** que pasa el nombre del nuevo contenedor (**productos**), la partición ruta de acceso de clave (**/categoryId**) y el rendimiento (**400**) que le gustaría crear en la base de datos de **cosmosworks** y almacenar el da como resultado una variable de tipo **Container**:
  
    ```
    Container container = await database.CreateContainerIfNotExistsAsync("products", "/categoryId", 400);    
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
    ```

1. **Guarde** el archivo de código **script.cs**.

1. En **Visual Studio Code**, abra el menú contextual de la carpeta **06-sdk-crud** y, a continuación, seleccione **Abrir en terminal integrado** para abrir una nueva instancia de terminal.

1. Compile y ejecute el proyecto mediante el comando [dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run]:

    ```
    dotnet run
    ```

1. Cierre el terminal integrado.

1. Cambie a la ventana del explorador web.

1. En el recurso de cuenta de **Azure Cosmos DB**, vaya al panel del **Explorador de datos**.

1. En **Data Explorer**, expanda el nodo de la base de datos **cosmicworks** y, a continuación, observe el nodo contenedor **products** nuevo dentro del árbol de navegación de la **API NOSQL**.

## Realización de operaciones de creación y lectura de puntos en elementos con el SDK

Ahora usará el conjunto de métodos asincrónicos en la clase Microsoft.Azure.Cosmos.Container para realizar operaciones comunes en elementos dentro de un contenedor de API noSQL. Estas operaciones se realizan con el modelo de programación asincrónica de tareas en C#.

1. Vuelva a **Visual Studio Code**. Abra el archivo de código **product.cs** dentro de la carpeta **06-sdk-crud**.

    > &#128221; No cierre el editor del archivo **script.cs** .

1. Observe la clase **Producto** dentro de este archivo de código. Esta clase representa un elemento de producto que se almacenará y manipulará dentro de este contenedor.

1. Vuelva a la pestaña editor del archivo de código **script.cs**.

1. Cree un nuevo objeto de tipo **Product** denominado **saddle** con las siguientes propiedades:

    | Propiedad | Valor |
    | ---: | :--- |
    | **id** | *706cd7c6-db8b-41f9-aea2-0e0c7e8eb009* |
    | **categoryId** | *9603ca6c-9e28-4a02-9194-51cdb7fea816* |
    | **name** | *Silla de carretera* |
    | **price** | *45.99d* |
    | **etiquetas** | *{ tan, new, crisp }* |

    ```
    Product saddle = new()
    {
        id = "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009",
        categoryId = "9603ca6c-9e28-4a02-9194-51cdb7fea816",
        name = "Road Saddle",
        price = 45.99d,
        tags = new string[]
        {
            "tan",
            "new",
            "crisp"
        }
    };
    ```

1. Invoque de forma asíncrona el método genérico [CreateItemAsync<>][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.createitemasync] de la variable **container** pasando la variable **saddle** como parámetro del método y utilizando **Product** como tipo genérico:

    ```
    await container.CreateItemAsync<Product>(saddle);
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

    Product saddle = new()
    {
        id = "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009",
        categoryId = "9603ca6c-9e28-4a02-9194-51cdb7fea816",
        name = "Road Saddle",
        price = 45.99d,
        tags = new string[]
        {
            "tan",
            "new",
            "crisp"
        }
    };

    await container.CreateItemAsync<Product>(saddle);
    ```

1. **Guarde** el archivo de código **script.cs**.

1. En **Visual Studio Code**, abra el menú contextual de la carpeta **06-sdk-crud** y, a continuación, seleccione **Abrir en terminal integrado** para abrir una nueva instancia de terminal.

1. Compile y ejecute el proyecto mediante el comando **[dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run]**:

    ```
    dotnet run
    ```

1. Cierre el terminal integrado.

1. Vuelva a la pestaña editor del archivo de código **script.cs**.

1. Elimine las siguientes líneas de código:

    ```
    Product saddle = new()
    {
        id = "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009",
        categoryId = "9603ca6c-9e28-4a02-9194-51cdb7fea816",
        name = "Road Saddle",
        price = 45.99d,
        tags = new string[]
        {
            "tan",
            "new",
            "crisp"
        }
    };

    await container.CreateItemAsync<Product>(saddle);
    ```

1. Cree una variable de cadena denominada **id** con un valor de **706cd7c6-db8b-41f9-aea2-0e0c7e8eb009**:

    ```
    string id = "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009";
    ```

1. Cree una variable de cadena denominada **categoryId** con un valor de **9603ca6c-9e28-4a02-9194-51cdb7fea816**:

    ```
    string categoryId = "9603ca6c-9e28-4a02-9194-51cdb7fea816";
    ```

1. Cree una variable de tipo [PartitionKey][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.partitionkey] denominada **partitionKey** pasando la variable **categoryId** como parámetro de constructor:

    ```
    PartitionKey partitionKey = new (categoryId);
    ```

1. Invoque de forma asíncrona el método genérico [ReadItemAsync<>][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.readitemasync] de la variable **container** pasando las variables **id** y **partitionkey** como parámetros del método, utilizando **Product** como tipo genérico y almacenando el resultado en una variable llamada **saddle** de tipo **Product**:

    ```
    Product saddle = await container.ReadItemAsync<Product>(id, partitionKey);
    ```

1. Invoque el método estático **Console.WriteLine** para imprimir el objeto silla utilizando una cadena de salida formateada:

    ```
    Console.WriteLine($"[{saddle.id}]\t{saddle.name} ({saddle.price:C})");
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

    string id = "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009";

    string categoryId = "9603ca6c-9e28-4a02-9194-51cdb7fea816";
    PartitionKey partitionKey = new (categoryId);

    Product saddle = await container.ReadItemAsync<Product>(id, partitionKey);

    Console.WriteLine($"[{saddle.id}]\t{saddle.name} ({saddle.price:C})");
    ```

1. **Guarde** el archivo de código **script.cs**.

1. En **Visual Studio Code**, abra el menú contextual de la carpeta **06-sdk-crud** y, a continuación, seleccione **Abrir en terminal integrado** para abrir una nueva instancia de terminal.

1. Compile y ejecute el proyecto mediante el comando **[dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run]**:

    ```
    dotnet run
    ```

1. Observe la salida del terminal. En concreto, observe el texto de salida con formato con el identificador, el nombre y el precio del elemento.

1. Cierre el terminal integrado.

## Realización de operaciones de actualización y eliminación de puntos con el SDK

Al aprender el SDK, no es raro usar una cuenta del SDK de Azure Cosmos DB en línea o el emulador para actualizar un elemento y oscilar entre el Explorador de datos y el IDE que prefiera mientras realiza una operación y comprueba si se ha aplicado el cambio. Aquí, lo hará al actualizar y eliminar un elemento mediante el SDK.

1. Vuelva a la ventana o pestaña del explorador web.

1. En el recurso de cuenta de **Azure Cosmos DB**, vaya al panel del **Explorador de datos**.

1. En el **Explorador de datos**, expanda el nodo de la base de datos **cosmicworks** y, a continuación, expanda el nodo contenedor **products** nuevo dentro del árbol de navegación de la **API NOSQL**.

1. Seleccione el nodo **Elementos**. Seleccione el único artículo dentro del contenedor y observe los valores de las propiedades **nombre** y **precio** del artículo.

    | **Propiedad** | **Valor** |
    | ---: | :--- |
    | **Nombre** | *Silla de carretera* |
    | **Precio** | *45,99 $* |

    > &#128221; En este momento, estos valores no deben haberse cambiado desde que creó el elemento. Estos valores se cambiarán en este ejercicio.

1. Vuelva a **Visual Studio Code**. Vuelva a la pestaña editor del archivo de código **script.cs**.

1. Elimine las siguientes líneas de código:

    ```
    Console.WriteLine($"[{saddle.id}]\t{saddle.name} ({saddle.price:C})");
    ```

1. Cambie la variable **silla** estableciendo el valor de la propiedad precio en **32,55**:

    ```
    saddle.price = 32.55d;
    ```

1. Modifique de nuevo la variable **silla** estableciendo el valor de la propiedad **nombre** en **Silla de carretera LL**:

    ```
    saddle.name = "Road LL Saddle";
    ```

1. Invoque de forma asíncrona el método genérico [UpsertItemAsync<>][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.upsertitemasync] de la variable **contenedor** pasando la variable **silla** como parámetro del método y utilizando **Producto** como tipo genérico:

    ```
    await container.UpsertItemAsync<Product>(saddle);
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

    string id = "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009";

    string categoryId = "9603ca6c-9e28-4a02-9194-51cdb7fea816";
    PartitionKey partitionKey = new (categoryId);

    Product saddle = await container.ReadItemAsync<Product>(id, partitionKey);

    saddle.price = 32.55d;
    saddle.name = "Road LL Saddle";
    
    await container.UpsertItemAsync<Product>(saddle);
    ```

1. **Guarde** el archivo de código **script.cs**.

1. En **Visual Studio Code**, abra el menú contextual de la carpeta **06-sdk-crud** y, a continuación, seleccione **Abrir en terminal integrado** para abrir una nueva instancia de terminal.

1. Compile y ejecute el proyecto mediante el comando **[dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run]**:

    ```
    dotnet run
    ```

1. Cierre el terminal integrado.

1. Vuelva a la ventana o pestaña del explorador web.

1. En el recurso de cuenta de **Azure Cosmos DB**, vaya al panel del **Explorador de datos**.

1. En el **Explorador de datos**, expanda el nodo de la base de datos **cosmicworks** y, a continuación, expanda el nodo contenedor **products** nuevo dentro del árbol de navegación de la **API NOSQL**.

1. Seleccione el nodo **Elementos**. Seleccione el único artículo dentro del contenedor y observe los valores de las propiedades **nombre** y **precio** del artículo.

    | **Propiedad** | **Valor** |
    | ---: | :--- |
    | **Nombre** | *Silla de carretera LL* |
    | **Precio** | *32,55 $* |

    > &#128221; En este momento, estos valores deben haberse cambiado desde que ha observado el elemento.

1. Vuelva a **Visual Studio Code**. Vuelva a la pestaña editor del archivo de código **script.cs**.

1. Elimine las siguientes líneas de código:

    ```
    Product saddle = await container.ReadItemAsync<Product>(id, partitionKey);

    saddle.price = 32.55d;
    saddle.name = "Road LL Saddle";
    
    await container.UpsertItemAsync<Product>(saddle);
    ```

1. Invoque de forma asíncrona el método genérico [DeleteItemAsync<>][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.deleteitemasync] de la variable **contenedor** pasando las variables **id** y **partitionkey** como parámetros del método y utilizando **Producto** como tipo genérico:

    ```
    await container.DeleteItemAsync<Product>(id, partitionKey);
    ```

1. **Guarde** el archivo de código **script.cs**.

1. En **Visual Studio Code**, abra el menú contextual de la carpeta **06-sdk-crud** y, a continuación, seleccione **Abrir en terminal integrado** para abrir una nueva instancia de terminal.

1. Compile y ejecute el proyecto mediante el comando **[dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run]**:

    ```
    dotnet run
    ```

1. Cierre el terminal integrado.

1. Vuelva a la ventana o pestaña del explorador web.

1. En el recurso de cuenta de **Azure Cosmos DB**, vaya al panel del **Explorador de datos**.

1. En el **Explorador de datos**, expanda el nodo de la base de datos **cosmicworks** y, a continuación, expanda el nodo contenedor **products** nuevo dentro del árbol de navegación de la **API NOSQL**.

1. Seleccione el nodo **Elementos**. Observe que la lista de elementos ahora está vacía.

1. Cierre la ventana o pestaña del explorador web.

1. Cierre **Visual Studio Code**.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.createitemasync]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.createitemasync
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.deleteitemasync]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.deleteitemasync
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.readitemasync]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.readitemasync
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.upsertitemasync]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.upsertitemasync
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.partitionkey]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.partitionkey
[docs.microsoft.com/dotnet/core/tools/dotnet-build]: https://docs.microsoft.com/dotnet/core/tools/dotnet-build
[docs.microsoft.com/dotnet/core/tools/dotnet-run]: https://docs.microsoft.com/dotnet/core/tools/dotnet-run
[nuget.org/packages/microsoft.azure.cosmos/3.22.1]: https://www.nuget.org/packages/Microsoft.Azure.Cosmos/3.22.1
