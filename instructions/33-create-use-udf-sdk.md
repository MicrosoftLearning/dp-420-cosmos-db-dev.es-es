---
lab:
  title: Implementación y uso de funciones definidas por el usuario con el SDK
  module: Module 13 - Create server-side programming constructs in Azure Cosmos DB for NoSQL
---

# Implementación y uso de funciones definidas por el usuario con el SDK

El SDK de .NET para Azure Cosmos DB for NoSQL se puede usar para administrar e invocar construcciones de programación del lado servidor directamente desde un contenedor. Al preparar un nuevo contenedor, puede tener sentido usar el SDK de .NET para publicar UDF directamente en un contenedor en lugar de realizar las tareas manualmente mediante el Explorador de datos.

En este laboratorio, creará una nueva UDF mediante el SDK de .NET y, a continuación, usará el Explorador de datos para validar que la UDF funciona correctamente.

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

    1. Observe el campo **URI**. Usará este valor de **punto de conexión** más adelante en este ejercicio.

    1. Observe el campo **CLAVE PRINCIPAL**. Usará este valor de **clave** más adelante en este ejercicio.

    1. Observe el campo **CADENA DE CONEXIÓN PRINCIPAL**. Usará este valor de **cadena de conexión** más adelante en este ejercicio.

1. Sin cerrar la ventana del explorador, abra **Visual Studio Code**.

## Inicialización de la cuenta de Azure Cosmos DB for NoSQL con datos

La herramienta de línea de comandos [cosmicworks][nuget.org/packages/cosmicworks] implementa datos de ejemplo en cualquier cuenta de Azure Cosmos DB for NoSQL. La herramienta es de código abierto y está disponible a través de NuGet. Instalará esta herramienta en Azure Cloud Shell y la usará para inicializar la base de datos.

1. En **Visual Studio Code**, abra el menú **Terminal** y, a continuación, seleccione **Nuevo terminal** para abrir una nueva instancia de terminal.

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

1. Cierre el terminal integrado.

## Crear una función definida por el usuario (UDF) usando el SDK de .NET

La clase [Container][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container] del SDK de .NET incluye una propiedad [Scripts][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.scripts] que se usa para realizar operaciones CRUD con procedimientos almacenados, UDF y desencadenadores directamente desde el SDK. Usará esta propiedad para crear una nueva UDF y después pasar esa UDF a un contenedor de Azure Cosmos DB for NoSQL. El UDF que crearemos usando el SDK computará el precio del producto con el impuesto, lo que nos permitirá ejecutar consultas SQL sobre los productos usando su precio con el impuesto.

1. En **Visual Studio Code**, en el panel **Explorador**, vaya a la carpeta **33-create-use-udf-sdk**.

1. Abra el archivo de código **script.cs**.

1. Agregue un bloque de uso para el espacio de nombres [Microsoft.Azure.Cosmos.Scripts][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts]:

    ```
    using Microsoft.Azure.Cosmos.Scripts;
    ```

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

1. Cree una nueva variable de tipo [UserDefinedFunctionProperties][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts.userdefinedfunctionproperties] llamada props usando el constructor vacío predeterminado:

    ```
    UserDefinedFunctionProperties props = new ();
    ```

1. Establezca la propiedad [Id][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts.userdefinedfunctionproperties.id] de la variable **props** con el valor **tax**:

    ```
    props.Id = "tax";
    ```

1. Establezca la propiedad [Body][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts.userdefinedfunctionproperties.body] de la variable **props** con un valor de **props.Body = "function tax(i) { return i * 1.25; }";**:

    ```
    props.Body = "function tax(i) { return i * 1.25; }";
    ```

1. Llame de forma asincrónica al método [Scripts.CreateUserDefinedFunctionAsync][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.scripts] de la variable **container** pasando la variable **props** como parámetro y guardando el resultado en una variable llamada **udf** de tipo [UserDefinedFunctionResponse][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts.userdefinedfunctionresponse]:

    ```
    UserDefinedFunctionResponse udf = await container.Scripts.CreateUserDefinedFunctionAsync(props);
    ```

1. Use el método estático incorporado **Console.WriteLine** para imprimir la propiedad [Resource.Id][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts.userdefinedfunctionresponse.resource] de la clase UserDefinedFunctionResponse con un encabezado titulado **UDF creada**:

    ```
    Console.WriteLine($"Created UDF [{udf.Resource?.Id}]");
    ```

1. Una vez que haya terminado, el archivo de código debería incluir:
  
    ```
    using System;
    using Microsoft.Azure.Cosmos;
    using Microsoft.Azure.Cosmos.Scripts;

    string endpoint = "<cosmos-endpoint>";

    string key = "<cosmos-key>";

    CosmosClient client = new CosmosClient(endpoint, key);

    Database database = await client.CreateDatabaseIfNotExistsAsync("cosmicworks");

    Container container = await database.CreateContainerIfNotExistsAsync("products", "/category/name");

    UserDefinedFunctionProperties props = new ();
    props.Id = "tax";
    props.Body = "function tax(i) { return i * 1.25; }";
    
    UserDefinedFunctionResponse udf = await container.Scripts.CreateUserDefinedFunctionAsync(props);
    
    Console.WriteLine($"Created UDF [{udf.Resource?.Id}]");
    ```

1. **Guarde** el archivo **script.cs**.

1. En **Visual Studio Code**, abra el menú contextual de la carpeta **33-create-use-udf-sdk** y después seleccione **Abrir en terminal integrado** para abrir una nueva instancia de terminal.

1. Compile y ejecute el proyecto mediante el comando [dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run]:

    ```
    dotnet run
    ```

1. El script generará ahora el nombre de la UDF recién creada:

    ```
    Created UDF [tax]
    ```

1. Cierre el terminal integrado.

1. Cierre **Visual Studio Code**.

## Prueba de la UDF mediante el Explorador de datos

Ahora que se ha creado una nueva UDF en el contenedor de Azure Cosmos DB, usará el Explorador de datos para validar que la UDF funciona según lo previsto.

1. Vuelva al explorador web.

1. En el recurso de cuenta de **Azure Cosmos DB**, vaya al panel del **Explorador de datos**.

1. En **Data Explorer**, expanda el nodo de la base de datos **cosmicworks** y, a continuación, observe el nodo contenedor **products** nuevo dentro del árbol de navegación de la **API NOSQL**.

1. Seleccione el nodo contenedor **products** dentro del árbol de navegación de la **API NOSQL** y, a continuación, seleccione **Nueva consulta SQL**.

1. En la pestaña consulta, seleccione **Ejecutar consulta** para ver una consulta estándar que seleccione todos los elementos sin ningún filtro.

1. Elimine el contenido del área del editor.

1. Cree una nueva consulta SQL que devuelva todos los documentos con dos valores de precio proyectados. El primer valor es el valor de precio sin formato del contenedor y el segundo valor es el valor de precio calculado por la UDF:

    ```
    SELECT p.id, p.price, udf.tax(p.price) AS priceWithTax FROM products p
    ```

1. Seleccione **Ejecutar consulta**.

1. Observe los documentos y compare sus campos **price** y **priceWithTax**.

    > &#128221; El campo **priceWithTax** debería tener un valor un 25 % mayor que el campo **price**.

1. Cierre la ventana o pestaña del explorador web.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.scripts]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.scripts
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts.userdefinedfunctionproperties]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts.userdefinedfunctionproperties
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts.userdefinedfunctionproperties.body]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts.userdefinedfunctionproperties.body
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts.userdefinedfunctionproperties.id]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts.userdefinedfunctionproperties.id
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts.userdefinedfunctionresponse]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts.userdefinedfunctionresponse
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts.userdefinedfunctionresponse.resource]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts.userdefinedfunctionresponse.resource
[docs.microsoft.com/dotnet/core/tools/dotnet-run]: https://docs.microsoft.com/dotnet/core/tools/dotnet-run
[nuget.org/packages/cosmicworks]: https://www.nuget.org/packages/cosmicworks/
