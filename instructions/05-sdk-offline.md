---
lab:
  title: "Configuración del SDK de Azure\_Cosmos\_DB for NoSQL para el desarrollo sin conexión"
  module: Module 3 - Connect to Azure Cosmos DB for NoSQL with the SDK
---

# Configuración del SDK de Azure Cosmos DB for NoSQL para el desarrollo sin conexión

El emulador de Azure Cosmos DB es una herramienta local que emula el servicio Azure Cosmos DB para desarrollo y pruebas. El emulador admite para NoSQL y se puede usar en lugar del servicio en la nube al desarrollar código mediante el SDK de Azure para .NET.

En este laboratorio, se conectará al emulador de Azure Cosmos DB desde el SDK de Azure para .NET.

## Preparación del entorno de desarrollo

Si aún no ha clonado el repositorio de código de laboratorio para **DP-420** al entorno en el que está trabajando en este laboratorio, siga estos pasos para hacerlo. De lo contrario, abra la carpeta clonada anteriormente en **Visual Studio Code**.

1. Inicie **Visual Studio Code**.

    > &#128221; Si aún no está familiarizado con la interfaz de Visual Studio Code, revise la [Documentación de introducción][code.visualstudio.com/docs/getstarted]

1. Abra la paleta de comandos y ejecute **Git: Clonar** para clonar el repositorio de GitHub ``https://github.com/microsoftlearning/dp-420-cosmos-db-dev`` en una carpeta local de su elección.

    > &#128161; Puede usar el método abreviado de teclado **CTRL+MAYÚS+P** para abrir la paleta de comandos.

1. Una vez clonado el repositorio, abra la carpeta local que seleccionó en **Visual Studio Code**.

## Iniciar el emulador de Azure Cosmos DB

El entorno ya debe tener el emulador preinstalado. Si no es así, consulte las [instrucciones de instalación][docs.microsoft.com/azure/cosmos-db/local-emulator] para instalar el emulador de Azure Cosmos DB. Una vez iniciado el emulador, puede recuperar la cadena de conexión y usarla para conectarse al emulador mediante el SDK de Azure para .NET o cualquier otro SDK que prefiera.

1. Inicie el **emulador de Azure Cosmos DB**.

    > &#128221; Es posible que se le pida que conceda acceso de administrador para iniciar el emulador. En el entorno de laboratorio, la cuenta de **administrador** tiene la misma contraseña que la cuenta de **Estudiante**.

    > &#128161; El emulador de Azure Cosmos DB está anclado tanto a la barra de tareas de Windows como al menú Inicio. ***Si el emulador no se inicia desde los iconos anclados, intente abrirlo haciendo doble clic en el ***archivo*** *****C:\Archivos de programa\Emulador de Azure Cosmos DB\CosmosDB.Emulator.exe**. Tenga en cuenta que el emulador tarda entre 20 y 30 segundos en iniciarse.

1. Espere a que el emulador abra automáticamente el explorador predeterminado y vaya a la página de aterrizaje de **localhost:8081/_explorer/index.html**.

1. En la página de aterrizaje **emulador de Azure Cosmos DB**, vaya al panel **inicio rápido**.

1. Este panel contiene los detalles de conexión y las credenciales necesarias para conectarse a la cuenta desde el SDK. Específicamente:

    > &#128221; Observe el campo **cadena de conexión principal**. Usará este valor de **cadena de conexión** más adelante en este ejercicio.

1. Vaya al panel **Explorador**.

1. En el **Explorador de datos**, observe que no hay ningún nodo dentro del árbol de navegación **API de NoSQL**.

1. Deje esta pestaña abierta y cambie a **Visual Studio Code**.

## Conexión al emulador desde el SDK

La biblioteca de **Microsoft.Azure.Cosmos** ya se ha instalado previamente en el script de .NET que usará en este ejercicio. Además, parte del código reutilizable ya se ha escrito para ahorrar tiempo. Tendrá que actualizar el valor de la cadena de conexión reutilizable y escribir un par de líneas de código para completar el script.

1. En **Visual Studio Code**, en el panel **explorador**, vaya a la carpeta **05-sdk-offline**.

1. Abra el archivo de código **script.cs** dentro de la carpeta **05-sdk-offline**.

1. Actualice la variable existente denominada **connectionString** con su valor establecido en la **cadena de conexión** del emulador de Azure Cosmos DB.
  
    ```
    string connectionString = "AccountEndpoint=https://localhost:8081/;AccountKey=C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==";
    ```

    > &#128221; El URI del emulador suele ser ***localhost:[puerto]*** mediante SSL con el puerto predeterminado establecido en **8081**.

    > &#128221; *C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==* es la clave predeterminada para todas las instalaciones del emulador. Esta clave se puede cambiar mediante las opciones de la línea de comandos.

1. Invoque de forma asincrónica el método [CreateDatabaseIfNotExistsAsync][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclient.createdatabaseifnotexistsasync] del **cliente** variable pasando el nombre de la nueva base de datos (**cosmicworks**) que le gustaría crear en el emulador y almacenar el resultado en una variable de tipo [Database][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.database]:

    ```
    Database database = await client.CreateDatabaseIfNotExistsAsync("cosmicworks");
    ```

1. Use el método estático **Console.WriteLine** integrado para imprimir la propiedad [Id][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.database.id] de la clase Database con un encabezado titulado **New Database**:

    ```
    Console.WriteLine($"New Database:\tId: {database.Id}");
    ```

1. Una vez que haya terminado, el archivo de código debería incluir:
  
    ```
    using System;
    using Microsoft.Azure.Cosmos;
    
    string connectionString = "AccountEndpoint=https://localhost:8081/;AccountKey=C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==";
    
    CosmosClient client = new (connectionString);
    
    Database database = await client.CreateDatabaseIfNotExistsAsync("cosmicworks");
    Console.WriteLine($"New Database:\tId: {database.Id}");
    ```

1. **Guarde** el archivo de código **script.cs**.

1. En **Visual Studio Code**, abra el menú contextual de la carpeta **05-sdk-offline** y, a continuación, seleccione **Abrir en terminal integrado** para abrir una nueva instancia de terminal.

    > &#128221; Este comando abrirá el terminal con el directorio inicial ya establecido en la carpeta **05-sdk-offline**.

1. Agregue el paquete [Microsoft.Azure.Cosmos][nuget.org/packages/microsoft.azure.cosmos/3.22.1] desde NuGet mediante el comando siguiente:

    ```
    dotnet add package Microsoft.Azure.Cosmos --version 3.22.1
    ```

1. Compile y ejecute el proyecto mediante el comando [dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run]:

    ```
    dotnet run
    ```

1. Cierre el terminal integrado.

## Visualización de los cambios en el emulador

Ahora que ha creado una nueva base de datos en el emulador de Azure Cosmos DB, usará el **Explorador de datos** en línea para observar la nueva base de datos de api noSQL en el emulador.

1. Vuelva a la línea de comandos

1. En la página de aterrizaje **emulador de Azure Cosmos DB**, vaya al panel **Explorador**.

1. En **Explorador de datos**, actualice la **API NOSQL** para observar el nuevo nodo de base de datos **cosmosworks** dentro del árbol de navegación.

1. Vuelva a **Visual Studio Code**.

## Creación y visualización de un nuevo contenedor

La creación de un contenedor es similar al patrón usado para crear una nueva base de datos. El código que aprende aquí será relevante tanto si crea recursos en la nube como en el emulador, solo tiene que cambiar la cadena de conexión. Ampliará el archivo de script para crear un nuevo contenedor junto con la base de datos.

1. En **Visual Studio Code**, en el panel **explorador**, vaya a la carpeta **05-sdk-offline**.

1. Abra de nuevo el archivo de código **script.cs** dentro de la carpeta **05-sdk-offline**.

1. Invoque de forma asincrónica el método [CreateContainerIfNotExistsAsync][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.database.createcontainerifnotexistsasync] de la **base de datos** que pasa el nombre del nuevo contenedor (**productos**), la partición ruta de acceso de clave (**/categoryId**) y el rendimiento (**400**) que le gustaría crear en la base de datos de **cosmosworks** y almacenar el da como resultado una variable de tipo [Container][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container]:

    ```
    Container container = await database.CreateContainerIfNotExistsAsync("products", "/categoryId", 400);
    ```

1. Use el método estático **Console.WriteLine** integrado para imprimir la propiedad [Id][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.id] de la clase Container con un encabezado titulado **New Container**:

    ```
    Console.WriteLine($"New Container:\tId: {container.Id}");
    ```

1. Una vez que haya terminado, el archivo de código debería incluir:
  
    ```
    using System;
    using Microsoft.Azure.Cosmos;;
    
    string connectionString = "AccountEndpoint=https://localhost:8081/;AccountKey=C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==";
    
    CosmosClient client = new (connectionString);
    
    Database database = await client.CreateDatabaseIfNotExistsAsync("cosmicworks");
    Console.WriteLine($"New Database:\tId: {database.Id}");
    
    Container container = await database.CreateContainerIfNotExistsAsync("products", "/categoryId", 400);
    Console.WriteLine($"New Container:\tId: {container.Id}");
    ```

1. **Guarde** el archivo de código **script.cs**.

1. En **Visual Studio Code**, abra el menú contextual de la carpeta **05-sdk-offline** y, a continuación, seleccione **Abrir en terminal integrado** para abrir una nueva instancia de terminal.

1. Compile y ejecute el proyecto mediante el comando [dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run]:

    ```
    dotnet run
    ```

1. Cierre el terminal integrado.

1. Cierre **Visual Studio Code**.

1. Cambie al explorador.

1. En la página de aterrizaje **emulador de Azure Cosmos DB**, vaya al panel **Explorador**.

1. En el **Explorador de datos**, actualice la **API SQL** para observar los nuevos **productos** del nodo contenedor dentro del nodo de base de datos de **cosmosworks**.

1. Cierre la ventana o pestaña del explorador web.

## Parada del emulador de Azure Cosmos DB

Es importante detener el emulador cuando haya terminado de usarlo, ya que puede usar recursos del sistema en su entorno. Usará el icono de la bandeja del sistema para detener el emulador y todas las instancias en ejecución.

 Vaya al icono del emulador en la bandeja del sistema de Windows, abra el menú contextual y seleccione **Salir** para apagar el emulador.

> &#128221; Todas las instancias del emulador pueden tardar un minuto en salir.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/azure/cosmos-db/local-emulator]: https://docs.microsoft.com/azure/cosmos-db/local-emulator
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.id]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.id
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclient.createdatabaseifnotexistsasync]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclient.createdatabaseifnotexistsasync
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.database]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.database
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.database.createcontainerifnotexistsasync]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.database.createcontainerifnotexistsasync
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.database.id]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.database.id
[docs.microsoft.com/dotnet/core/tools/dotnet-run]: https://docs.microsoft.com/dotnet/core/tools/dotnet-run
[nuget.org/packages/microsoft.azure.cosmos/3.22.1]: https://www.nuget.org/packages/Microsoft.Azure.Cosmos/3.22.1
