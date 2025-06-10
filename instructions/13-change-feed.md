---
lab:
  title: "Procesamiento de eventos de una fuente de cambios con el SDK de Azure Cosmos\_DB for NoSQL"
  module: Module 7 - Integrate Azure Cosmos DB for NoSQL with Azure services
---

# Procesamiento de eventos de una fuente de cambios con el SDK de Azure Cosmos DB for NoSQL

La fuente de cambios de Azure Cosmos DB para NoSQL es la clave para crear aplicaciones complementarias controladas por eventos de la plataforma. El SDK de .NET para Azure Cosmos DB para NoSQL se incluye con un conjunto de clases para compilar las aplicaciones que se integran con la fuente de cambios y escuchar notificaciones sobre las operaciones dentro de los contenedores.

En este laboratorio, usará la funcionalidad del procesador de fuente de cambios en el SDK de .NET para crear una aplicación que reciba una notificación con una operación de creación o actualización se realiza en un elemento del contenedor especificado.

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

    1. Observe el campo **PRIMARY KEY**. Usará el valor **key** más adelante en este ejercicio.

    1. Observe el campo **CADENA DE CONEXIÓN PRINCIPAL**. Usará este valor de **cadena de conexión** más adelante en este ejercicio.

1. En el menú de recursos, seleccione **Explorador de datos**.

1. En el panel **Data Explorer**, expanda **Nuevo contenedor** y, a continuación, seleccione **Nueva base de datos**.

1. En la ventana emergente **Nueva base de datos**, escriba los valores siguientes para cada configuración y, después, seleccione **Aceptar**:

    | **Configuración** | **Valor** |
    | --: | :-- |
    | **Id. de base de datos** | *``cosmicworks``* |
    | **Aprovisionar rendimiento** | enabled |
    | **Rendimiento de base de datos** | **Manual** |
    | **RU/s necesarios de base de datos** | ``1000`` |

1. De nuevo en el panel **Data Explorer**, observe el nodo de base de datos **cosmicworks** dentro de la jerarquía.

1. En el panel **Data Explorer**, seleccione **Nuevo contenedor**.

1. En la ventana emergente **Nuevo contenedor**, escriba los valores siguientes para cada configuración y, después, seleccione **Aceptar**:

    | **Configuración** | **Valor** |
    | --: | :-- |
    | **Id. de base de datos** | *Usar existente* &vert; *cosmicworks* |
    | **Id. de contenedor** | *``products``* |
    | **Clave de partición** | *``/category/name``* |

1. De nuevo en el panel **Data Explorer**, expanda el nodo de base de datos **cosmicworks** y observe el nodo contenedor **productos** dentro de la jerarquía.

1. En el panel **Data Explorer**, vuelva a seleccionar **Nuevo contenedor**.

1. En la ventana emergente **Nuevo contenedor**, escriba los valores siguientes para cada configuración y, después, seleccione **Aceptar**:

    | **Configuración** | **Valor** |
    | --: | :-- |
    | **Id. de base de datos** | *Usar existente* &vert; *cosmicworks* |
    | **Id. de contenedor** | *``productslease``* |
    | **Clave de partición** | *``/partitionKey``* |

1. De nuevo en el panel **Data Explorer**, expanda el nodo de base de datos **cosmosworks** y observe el nodo de contenedor **productslease** dentro de la jerarquía.

1. Vuelva a **Visual Studio Code**.

## Implementación del procesador de fuente de cambios en el SDK de .NET

La clase **Microsoft.Azure.Cosmos.Container** se incluye con una serie de métodos para compilar el procesador de fuente de cambios fluidamente. Para empezar, necesita una referencia al contenedor supervisado, el contenedor de concesión y un delegado en C\# (para controlar cada lote de cambios).

1. En el panel **Explorador**, vaya a la carpeta **13-change-feed**.

1. Abra el archivo de código **product.cs**.

1. Observe la clase **Product** y sus propiedades correspondientes. En concreto, este laboratorio usará las propiedades **id** y **name**.

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

1. Use el método **GetContainer** de la variable **client** para recuperar el contenedor existente con el nombre de la base de datos (*cosmicworks*) y el nombre del contenedor (*products*) y almacenar el resultado en una variable denominada **sourceContainer** de tipo **Container**:

    ```
    Container sourceContainer = client.GetContainer("cosmicworks", "products");
    ```

1. Use el método **GetContainer** de la variable **client** para recuperar el contenedor existente con el nombre de la base de datos (*cosmicworks*) y el nombre del contenedor (*productslease*) y almacenar el resultado en una variable denominada **leaseContainer** de tipo **Container**:

    ```
    Container leaseContainer = client.GetContainer("cosmicworks", "productslease");
    ```

1. Cree una nueva variable de delegado denominada **handleChanges** de tipo [ChangesHandler<>][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.changefeedhandler-1] con una función anónima asincrónica vacía que tenga dos parámetros de entrada:

    1. Parámetro denominado **changes** de tipo **IReadOnlyCollection\<Product\>**.
    
    1. Parámetro denominado **cancellationToken** de tipo **CancellationToken**.

    ```
    ChangesHandler<Product> handleChanges = async (
        IReadOnlyCollection<Product> changes,
        CancellationToken cancellationToken
    ) => {
    };
    ```

1. Dentro de la función anónima, use el método estático **Console.WriteLine** integrado para imprimir la cadena sin formato **START\tHandling batch of changes...**:

    ```
    Console.WriteLine($"START\tHandling batch of changes...");
    ```

1. Aún dentro de la función anónima, cree un bucle foreach que recorra en iteración la variable **changes** mediante la variable **product** para representar una instancia de tipo **Product**:

    ```
    foreach(Product product in changes)
    {
    }
    ```

1. Dentro del bucle foreach de la función anónima, use el método estático **Console.WriteLineAsync** integrado para imprimir las propiedades **id** y**name** de la variable **product**:

    ```
    await Console.Out.WriteLineAsync($"Detected Operation:\t[{product.id}]\t{product.name}");
    ```

1. Fuera del bucle foreach y la función anónima, cree una nueva variable denominada **builder** que almacene el resultado de invocar [GetChangeFeedProcessorBuilder<>][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.getchangefeedprocessorbuilder] en la variable **sourceContainer** mediante los parámetros siguientes:

    | **Parámetro** | **Valor** |
    | ---: | :--- |
    | **processorName** | *productsProcessor* |
    | **onChangesDelegate** | *handleChanges* |

    ```
    var builder = sourceContainer.GetChangeFeedProcessorBuilder<Product>(
        processorName: "productsProcessor",
        onChangesDelegate: handleChanges
    );
    ```

1. Invoque el método [WithInstanceName][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessorbuilder.withinstancename] con un parámetro de **consoleApp**, el método [WithLeaseContainer][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessorbuilder.withleasecontainer] con un parámetro de **leaseContainer** y el método [Build][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessorbuilder.build] fluidamente en la variable **builder** que almacene el resultado en una variable denominada **processor** de tipo [ChangeFeedProcessor][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessor]:

    ```
    ChangeFeedProcessor processor = builder
        .WithInstanceName("consoleApp")
        .WithLeaseContainer(leaseContainer)
        .Build();
    ```

1. Invoque asincrónicamente [StartAsync][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessor.startasync] de la variable **processor**:

    ```
    await processor.StartAsync();
    ```

1. Use los métodos estáticos **Console.WriteLine** y **Console.ReadKey** integrados para imprimir la salida en la consola y hacer que la aplicación espere una pulsación de tecla:

    ```
    Console.WriteLine($"RUN\tListening for changes...");
    Console.WriteLine("Press any key to stop");
    Console.ReadKey();  
    ```

1. Invoque de forma asincrónica [stopAsync][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessor.stopasync] de la variable **processor**:

    ```
    await processor.StopAsync();
    ```

1. Una vez que haya terminado, el archivo de código debería incluir:
  
    ```
    using Microsoft.Azure.Cosmos;
    using static Microsoft.Azure.Cosmos.Container;

    string endpoint = "<cosmos-endpoint>";
    string key = "<cosmos-key>";

    CosmosClient client = new CosmosClient(endpoint, key);
    
    Container sourceContainer = client.GetContainer("cosmicworks", "products");
    Container leaseContainer = client.GetContainer("cosmicworks", "productslease");
    
    ChangesHandler<Product> handleChanges = async (
        IReadOnlyCollection<Product> changes,
        CancellationToken cancellationToken
    ) => {
        Console.WriteLine($"START\tHandling batch of changes...");
        foreach(Product product in changes)
        {
            await Console.Out.WriteLineAsync($"Detected Operation:\t[{product.id}]\t{product.name}");
        }
    };
    
    var builder = sourceContainer.GetChangeFeedProcessorBuilder<Product>(
            processorName: "productsProcessor",
            onChangesDelegate: handleChanges
        );
    
    ChangeFeedProcessor processor = builder
        .WithInstanceName("consoleApp")
        .WithLeaseContainer(leaseContainer)
        .Build();
    
    await processor.StartAsync();
    
    Console.WriteLine($"RUN\tListening for changes...");
    Console.WriteLine("Press any key to stop");
    Console.ReadKey();
    
    await processor.StopAsync();
    ```

1. **Guarde** el archivo **script.cs**.

1. En **Visual Studio Code**, abra el menú contextual de la carpeta **13-change-feed** y, a continuación, seleccione **Abrir en terminal integrado** para abrir una nueva instancia de terminal.

1. Compile y ejecute el proyecto mediante el comando [dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run]:

    ```
    dotnet run
    ```

1. Deje abiertos **Visual Studio Code** y el terminal.

    > &#128221; Usará otra herramienta para generar elementos en el contenedor de Azure Cosmos DB for NoSQL. Una vez que genere los elementos, volverá a este terminal para observar la salida. No cierre el terminal prematuramente.

## Inicialización de la cuenta de Azure Cosmos DB for NoSQL con datos de ejemplo

Usará una utilidad de línea de comandos que crea una base de datos de **cosmicworks** y un contenedor de **productos**. A continuación, la herramienta creará un conjunto de elementos que observará mediante el procesador de fuente de cambios que se ejecuta en la ventana del terminal.

1. En **Visual Studio Code**, abra el menú **Terminal** y seleccione **Dividir terminal** para abrir un nuevo terminal en paralelo con la instancia existente.

1. Instale la herramienta de línea de comandos [cosmicworks][nuget.org/packages/cosmicworks] para su uso global en la máquina.

    ```
    dotnet tool install --global CosmicWorks --version 2.3.1
    ```

    > &#128161; Este comando puede tardar un par de minutos en completarse. Este comando generará el mensaje de advertencia (*Tool "cosmicworks" ya está instalado) si ya ha instalado la versión más reciente de esta herramienta en el pasado.

1. Ejecute cosmicworks para inicializar la cuenta de Azure Cosmos DB con las siguientes opciones de línea de comandos:

    | **Opción** | **Valor** |
    | ---: | :--- |
    | **-c** | *El valor de la cadena de conexión que ha comprobado anteriormente en este laboratorio* |
    | **--number-of-employees** | *El comando cosmicworks rellena la base de datos tanto con empleados como con contenedores de productos con 1000 y 200 elementos, respectivamente, a menos que se especifique lo contrario.* |

    ```powershell
    cosmicworks -c "connection-string" --number-of-employees 0 --disable-hierarchical-partition-keys
    ```

    > &#128221; Por ejemplo, si el punto de conexión es **https&shy;://dp420.documents.azure.com:443/** y la clave es **fDR2ci9QgkdkvERTQ==**, el comando sería: ``cosmicworks -c "AccountEndpoint=https://dp420.documents.azure.com:443/;AccountKey=fDR2ci9QgkdkvERTQ==" --number-of-employees 0 --disable-hierarchical-partition-keys``

1. Espere a que el comando **cosmicworks** termine de poblar la cuenta con una base de datos, un contenedor y elementos.

1. Observe la salida del terminal de la aplicación .NET. El terminal genera un mensaje **Operación detectada** para cada cambio que se le envió mediante la fuente de cambios.

1. Cierre ambos terminales integrados.

1. Cierre **Visual Studio Code**.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.changefeedhandler-1]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.changefeedhandler-1
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessor]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessor
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessor.startasync]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessor.startasync
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessor.stopasync]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessor.stopasync
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessorbuilder.build]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessorbuilder.build
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessorbuilder.withinstancename]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessorbuilder.withinstancename
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessorbuilder.withleasecontainer]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessorbuilder.withleasecontainer
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.getchangefeedprocessorbuilder]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.getchangefeedprocessorbuilder
[docs.microsoft.com/dotnet/core/tools/dotnet-run]: https://docs.microsoft.com/dotnet/core/tools/dotnet-run
[nuget.org/packages/cosmicworks]: https://www.nuget.org/packages/cosmicworks
