---
lab:
  title: Conectarse a Azure Cosmos DB for NoSQL con el SDK
  module: Module 3 - Connect to Azure Cosmos DB for NoSQL with the SDK
---

# Conectarse a Azure Cosmos DB for NoSQL con el SDK

El SDK de Azure para .NET es un conjunto de bibliotecas que proporciona una interfaz de desarrollador coherente para interactuar con muchos servicios de Azure. El SDK de Azure para .NET se compila en la especificación de .NET Standard 2.0, lo que garantiza que se pueda usar en .NET Framework (4.6.1 o superior), .NET Core (2.1 o superior) y aplicaciones de .NET (5 o superior).

En este laboratorio, se conectará a una cuenta de Azure Cosmos DB para NoSQL mediante el SDK de Azure para .NET.

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
    | **Limitar la cantidad total de rendimiento que se puede aprovisionar en esta cuenta** | *Desactivado* |

    > &#128221; Es posible que los entornos de laboratorio tengan restricciones que le impidan crear un nuevo grupo de recursos. Si es así, usa el grupo de recursos existente creado previamente.

1. Espera a que se complete la tarea de implementación antes de continuar con esta tarea.

1. Ve al recurso de cuenta de **Azure Cosmos DB** recién creado y, después, ve al panel **Claves**.

1. Este panel contiene los detalles de conexión y las credenciales necesarias para conectarte a la cuenta desde el SDK. Específicamente:

    1. Observe el campo **URI**. Usará este valor de **endpoint** más adelante en este ejercicio.

    1. Observe el campo **PRIMARY KEY**. Usará este valor de **clave** más adelante en este ejercicio.

1. Mantenga abierta la pestaña del explorador, ya que volveremos a ella más adelante.

## Visualización de la biblioteca Microsoft.Azure.Cosmos en NuGet

El sitio web de NuGet contiene un índice que permite búsquedas de paquetes que están disponibles para importar en las aplicaciones .NET. Para importar paquetes de versión preliminar como **Microsoft.Azure.Cosmos**, puede usar el sitio web de NuGet para obtener las versiones y comandos adecuados para importar el paquete en las aplicaciones.

1. En la nueva pestaña del explorador, vaya al sitio web de NuGet (``nuget.org``).

1. Revise la descripción de NuGet, el administrador de paquetes para .NET y sus funcionalidades.

1. Busque la biblioteca **Microsoft.Azure.Cosmos** en NuGet.org.

1. Seleccione la pestaña **CLI de .NET** para observar el comando necesario para importar la versión más reciente de esta biblioteca en un proyecto de .NET.

    > &#128161; No es necesario grabar este comando. Usará una versión específica de la biblioteca más adelante en este ejercicio.

1. Cierre la ventana o pestaña del explorador web.

## Importación de la biblioteca Microsoft.Azure.Cosmos en un proyecto de .NET

La CLI de .NET incluye un comando [add package][docs.microsoft.com/dotnet/core/tools/dotnet-add-package] para importar paquetes desde una fuente de paquetes preconfigurada. Una instalación de .NET usa NuGet como fuente de paquetes predeterminada.

1. En **Visual Studio Code**, en el panel **Explorador**, vaya a la carpeta **04-sdk-connect**.

1. Abra el menú contextual de la carpeta **04-sdk-connect** y, a continuación, seleccione **Abrir en terminal integrado** para abrir una nueva instancia de terminal.

    > &#128221; Este comando abrirá el terminal con el directorio inicial ya establecido en la carpeta **04-sdk-connect**.

1. Agregue el paquete [Microsoft.Azure.Cosmos][nuget.org/packages/microsoft.azure.cosmos] desde NuGet mediante el comando siguiente:

    ```
    dotnet add package Microsoft.Azure.Cosmos --version 3.49.0
    ```

1. Cierre el terminal integrado.

## Uso de la biblioteca Microsoft.Azure.Cosmos

Una vez importada la biblioteca de Azure Cosmos DB del SDK de Azure para .NET, puede usar inmediatamente sus clases en el espacio de nombres [Microsoft.Azure.Cosmos][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos] para conectarse a una cuenta de Azure Cosmos DB para NoSQL. La clase [CosmosClient][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclient] es la clase principal que se usa para establecer la conexión inicial a una cuenta de Azure Cosmos DB para NoSQL.

1. En **Visual Studio Code**, en el panel **Explorador**, vaya a la carpeta **04-sdk-connect**.

1. Abra el archivo de código **script.cs** vacío.

1. Agregue los bloques using para los espacios de nombres **System** y **System.Linq** integrados:

    ```
    using System;
    using System.Linq;
    ```

1. Agregue un bloque using para el espacio de nombres [Microsoft.Azure.Cosmos][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos]:

    ```
    using Microsoft.Azure.Cosmos;
    ```

1. Agregue una variable **string** denominada **endpoint** con su valor establecido en el **punto de conexión** de la cuenta de Azure Cosmos DB que creó anteriormente.
  
    ```
    string endpoint = "<cosmos-endpoint>";
    ```

    > &#128221; Por ejemplo, si el punto de conexión es: **https&shy;://dp420.documents.azure.com:443/**, la instrucción C# sería: **string endpoint = "https&shy;://dp420.documents.azure.com:443/";**.

1. Agregue una variable **string** denominada **key** con su valor establecido en la **clave** de la cuenta de Azure Cosmos DB que creó anteriormente.

    ```
    string key = "<cosmos-key>";
    ```

    > &#128221; Por ejemplo, si la clave es: **fDR2ci9QgkkvERTQ==**, la instrucción C# sería: **string key = "fDR2ci9QgkdkvERTQ==";**.

1. Agregue una nueva variable denominada **client** de tipo [CosmosClient][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclient] mediante las variables **endpoint** y **key** en el constructor:
  
    ```
    CosmosClient client = new (endpoint, key);
    ```

1. Agregue una nueva variable denominada **account** de tipo [AccountProperties][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.accountproperties] mediante el resultado asincrónico de invocar el método [ReadAccountAsync][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclient.readaccountasync] de la variable **client**:

    ```
    AccountProperties account = await client.ReadAccountAsync();
    ```

1. Use el método estático **Console.WriteLine** integrado para imprimir la propiedad [Id][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.accountproperties.id] de la clase AccountProperties con un encabezado titulado **Nombre de cuenta**:

    ```
    Console.WriteLine($"Account Name:\t{account.Id}");
    ```

1. Use el método estático **Console.WriteLine** integrado para consultar la propiedad [WritableRegions][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.accountproperties.writableregions] de la clase AccountProperties y, a continuación, imprima la propiedad [Name][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.accountregion.name] del primer resultado con un encabezado titulado **Región primaria**:

    ```
    Console.WriteLine($"Primary Region:\t{account.WritableRegions.FirstOrDefault()?.Name}");
    ```

1. Una vez que haya terminado, el archivo de código debería incluir:
  
    ```
    using System;
    using System.Linq;
    
    using Microsoft.Azure.Cosmos;

    string endpoint = "<cosmos-endpoint>";
    string key = "<cosmos-key>";

    CosmosClient client = new (endpoint, key);

    AccountProperties account = await client.ReadAccountAsync();

    Console.WriteLine($"Account Name:\t{account.Id}");
    Console.WriteLine($"Primary Region:\t{account.WritableRegions.FirstOrDefault()?.Name}");
    ```

1. **Guarde** el archivo de código **script.cs**.

## Prueba del script

Ahora que el código de .NET para conectarse a la cuenta de Azure Cosmos DB para NoSQL está completo, puede probar el script. Este script imprimirá el nombre de la cuenta y el nombre de la primera región grabable. Al crear la cuenta, especificó una ubicación y debería esperar ver ese mismo valor de ubicación impreso como resultado de este script.

1. En **Visual Studio Code**, abra el menú contextual de la carpeta **04-sdk-connect** y, a continuación, seleccione **Abrir en terminal integrado** para abrir una nueva instancia de terminal.

1. Compile y ejecute el proyecto mediante el comando [dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run]:

    ```
    dotnet run
    ```

1. El script ahora generará el nombre de la cuenta y la primera región grabable. Por ejemplo, si llamó a la cuenta **dp420** y la primera región grabable era **Oeste de EE. UU. 2**, el script generaría:

    ```
    Account Name:   dp420
    Primary Region: West US 2
    ```

1. Cierre el terminal integrado.

1. Cierra **Visual Studio Code**.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.accountproperties]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.accountproperties
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.accountproperties.id]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.accountproperties.id
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.accountproperties.writableregions]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.accountproperties.writableregions
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.accountregion.name]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.accountregion.name
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclient]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclient
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclient.readaccountasync]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclient.readaccountasync
[docs.microsoft.com/dotnet/core/tools/dotnet-add-package]: https://docs.microsoft.com/dotnet/core/tools/dotnet-add-package
[docs.microsoft.com/dotnet/core/tools/dotnet-run]: https://docs.microsoft.com/dotnet/core/tools/dotnet-run
[nuget.org/packages/microsoft.azure.cosmos]: https://www.nuget.org/packages/Microsoft.Azure.Cosmos
