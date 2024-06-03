---
lab:
  title: "Solución de problemas de una aplicación con el SDK de Azure Cosmos\_DB for NoSQL"
  module: Module 11 - Monitor and troubleshoot an Azure Cosmos DB for NoSQL solution
---

# Solución de problemas de una aplicación con el SDK de Azure Cosmos DB for NoSQL

Azure Cosmos DB ofrece un amplio conjunto de códigos de respuesta, que nos ayudan a solucionar fácilmente los problemas que puedan surgir con nuestros diferentes tipos de operaciones. La clave está en asegurarnos de que programamos el control de errores adecuado al crear aplicaciones para Azure Cosmos DB.

En este laboratorio, crearemos un programa controlado por menús que nos permitirá insertar o eliminar uno de los dos documentos. El objetivo principal de este laboratorio es introducirnos en el uso de algunos de los códigos de respuesta más comunes y cómo utilizarlos en el código de control de errores de la aplicación.  Aunque codificaremos el control de errores para varios códigos de respuesta, solo desencadenaremos dos tipos diferentes de condiciones.  Además, el control de errores no hará nada complejo, en función del código de respuesta mostrará un mensaje en la pantalla o esperará 10 segundos y volverá a intentar la operación una vez más. 

## Preparación del entorno de desarrollo

Si aún no ha clonado el repositorio de código de laboratorio para **DP-420** al entorno en el que está trabajando en este laboratorio, siga estos pasos para hacerlo. De lo contrario, abra la carpeta clonada anteriormente en **Visual Studio Code**.

1. Inicie **Visual Studio Code**.

    > &#128221; Si aún no está familiarizado con la interfaz de Visual Studio Code, revise la [Guía de introducción para Visual Studio Code][code.visualstudio.com/docs/getstarted]

1. Abra la paleta de comandos y ejecute **Git: Clonar** para clonar el repositorio de GitHub ``https://github.com/microsoftlearning/dp-420-cosmos-db-dev`` en una carpeta local de su elección.

    > &#128161; Puede usar el método abreviado de teclado **CTRL+MAYÚS+P** para abrir la paleta de comandos.

1. Una vez clonado el repositorio, abra la carpeta local que seleccionó en **Visual Studio Code**.

## Creación de una cuenta de Azure Cosmos DB for NoSQL

Azure Cosmos DB es un servicio de base de datos NoSQL basado en la nube que admite varias API. Al aprovisionar una cuenta de Azure Cosmos DB por primera vez, seleccione cuál de las API quiere que admita la cuenta (por ejemplo, **API Mongo** o **API NoSQL**). Una vez que la cuenta de Azure Cosmos DB for NoSQL haya terminado de aprovisionar, puede recuperar el punto de conexión y la clave. Use el punto de conexión y la clave para conectarse a la cuenta de Azure Cosmos DB for NoSQL mediante programación. Use el punto de conexión y la clave en las cadenas de conexión del SDK de Azure para .NET o cualquier otro SDK.

1. Vaya a Azure Portal (``portal.azure.com``) desde una nueva ventana o pestaña del explorador web.

1. Inicie sesión en el portal con las credenciales de Microsoft asociadas a su suscripción.

1. Seleccione **+ Crear un recurso**, busque *Cosmos DB* y, a continuación, cree un nuevo recurso de cuenta de **Azure Cosmos DB for NoSQL** con la siguiente configuración, dejando todas las opciones restantes en sus valores predeterminados:

    | **Configuración** | **Valor** |
    | ---: | :--- |
    | **Suscripción** | *Su suscripción de Azure existente* |
    | **Grupo de recursos** | *Seleccione un grupo de recursos ya existente o cree uno nuevo* |
    | **Account Name** | *Escriba un nombre único global*. |
    | **Ubicación** | *seleccione cualquier región disponible* |
    | **Capacity mode (Modo de capacidad)** | *Rendimiento aprovisionado* |
    | **Aplicación de descuento por nivel Gratis** | *No aplicar* |

    > &#128221; Es posible que los entornos de laboratorio tengan restricciones que le impidan crear un nuevo grupo de recursos. Si es así, use el grupo de recursos existente creado previamente.

1. Espere a que se complete la tarea de implementación antes de continuar con esta tarea.

1. Vaya al recurso de cuenta de **Azure Cosmos DB** recién creado y vaya al panel **Claves**.

1. Este panel contiene los detalles de conexión y las credenciales necesarias para conectarse a la cuenta desde el SDK. Específicamente:

    1. Observe el campo **URI**. Usará este valor de **punto de conexión** más adelante en este ejercicio.

    1. Observe el campo **CLAVE PRINCIPAL**. Usará este valor de **clave** más adelante en este ejercicio.

1. Minimice, pero no cierre, la ventana del explorador. Volveremos a Azure Portal unos minutos después de iniciar una carga de trabajo en segundo plano en los pasos siguientes.


## Importación de la biblioteca Microsoft.Azure.Cosmos en un script de .NET

La CLI de .NET incluye un comando [add package][docs.microsoft.com/dotnet/core/tools/dotnet-add-package] para importar paquetes desde una fuente de paquetes preconfigurada. Una instalación de .NET usa NuGet como fuente de paquetes predeterminada.

1. En **Visual Studio Code**, en el panel **Explorador**, vaya a la carpeta **26-sdk-troubleshoot**.

1. Abra el menú contextual de la carpeta **26-sdk-troubleshoot** y, a continuación, seleccione **Abrir en terminal integrado** para abrir una nueva instancia de terminal.

    > &#128221; Este comando abrirá el terminal con el directorio inicial ya establecido en la carpeta **26-sdk-troubleshoot**.

1. Agregue el paquete [Microsoft.Azure.Cosmos][nuget.org/packages/microsoft.azure.cosmos/3.22.1] desde NuGet mediante el comando siguiente:

    ```
    dotnet add package Microsoft.Azure.Cosmos --version 3.22.1
    ```

## Ejecute un script para crear opciones controladas por menús para insertar y eliminar documentos.

Para poder ejecutar la aplicación, es necesario conectarla a nuestra cuenta de Azure Cosmos DB. 

1. En **Visual Studio Code**, en el panel **Explorador**, vaya a la carpeta **26-sdk-troubleshoot**.

1. Abra el archivo de código **Program.cs**.

1. Actualice la variable existente denominada **endpoint** con su valor establecido en el **punto de conexión** de la cuenta de Azure Cosmos DB que creó anteriormente.
  
    ```
    private static readonly string endpoint = "<cosmos-endpoint>";
    ```

    > &#128221; Por ejemplo, si el punto de conexión es: **https&shy;://dp420.documents.azure.com:443/**, la instrucción C# sería: **private static readonly string endpoint = "https&shy;://dp420.documents.azure.com:443/";**.

1. Actualice la variable existente denominada **key** con su valor establecido en la **clave** de la cuenta de Azure Cosmos DB que creó anteriormente.

    ```
    private static readonly string key = "<cosmos-key>";
    ```

    > &#128221; Por ejemplo, si la clave es: **fDR2ci9QgkkvERTQ==**, la instrucción C# sería: **private static readonly string key = "fDR2ci9QgkdkvERTQ==";**.

1. Guarde el archivo.

1. Compile y ejecute el proyecto mediante el comando [dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run]:

    ```
    dotnet run
    ```
    > &#128221; Este es un programa muy sencillo.  Mostrará un menú con cinco opciones como se muestra a continuación. Dos opciones para insertar un documento predefinido, dos para eliminar un documento predefinido y una opción para salir del programa.

    >```
    >1) Add Document 1 with id = '0C297972-BE1B-4A34-8AE1-F39E6AA3D828'
    >2) Add Document 2 with id = 'AAFF2225-A5DD-4318-A6EC-B056F96B94B7'
    >3) Delete Document 1 with id = '0C297972-BE1B-4A34-8AE1-F39E6AA3D828'
    >4) Delete Document 2 with id = 'AAFF2225-A5DD-4318-A6EC-B056F96B94B7'
    >5) Exit
    >Select an option:
    >```

## Hora de insertar y eliminar documentos.

1. Seleccione **1** y **ENTRAR** para insertar el primer documento. El programa insertará el primer documento y devolverá el mensaje siguiente.

    ```
    Insert Successful.
    Document for customer with id = '0C297972-BE1B-4A34-8AE1-F39E6AA3D828' Inserted.
    Press [ENTER] to continue
    ```

1. De nuevo, seleccione **1** y **ENTRAR** para insertar el primer documento. Esta vez el programa se bloqueará con una excepción. Al examinar la pila de errores, podemos encontrar el motivo del error del programa. Como podemos saber del mensaje extraído de la pila de errores, encontramos una excepción no controlada "Conflicto (409)"

    ```
    Unhandled exception. Microsoft.Azure.Cosmos.CosmosException : Response status code does not indicate success: Conflict (409);
    ```

1. Dado que estamos insertando un documento, necesitaremos revisar la lista de [códigos de estado de creación de documentos][/rest/api/cosmos-db/create-a-document#status-codes] comunes que se devuelven cuando se crea un documento. La descripción de este código es: *el id. proporcionado para el nuevo documento ha sido tomado por un documento existente*. Esto es obvio, ya que acabamos de ejecutar la opción de menú para crear el mismo documento hace unos momentos.

1. Profundizar más en la pila, podemos ver que se llamó a esta excepción desde la línea 100 y que a su vez se llamó desde la línea 64.

    ```
    at Program.CreateDocument1(Container Customer) in C:\Git\dp-420-cosmos-db-dev\26-sdk-troubleshoot\Program.cs:line 100   
   at Program.CompleteTaskOnCosmosDB(String consoleinputcharacter, Container container) in C:\Git\dp-420-cosmos-db-dev\26-sdk-troubleshoot\Program.cs:line 64
    ```

1. Al revisar la línea 100, como se esperaba, el error se debió a la operación *CreateItemAsync*. 

    ```C#
        ItemResponse<customerInfo> response = await Customer.CreateItemAsync<customerInfo>(customer, new PartitionKey(customerID));
    ```

1. Además, al revisar las líneas 100 a 103, es obvio que este código no tiene control de errores. Tendremos que corregirlo. 

    ```C#
        ItemResponse<customerInfo> response = await Customer.CreateItemAsync<customerInfo>(customer, new PartitionKey(customerID));
        Console.WriteLine("Insert Successful.");
        Console.WriteLine("Document for customer with id = '" + customerID + "' Inserted.");
    ```

1. Tendremos que decidir qué debe hacer nuestro código de control de errores. Al revisar los [códigos de estado del documento de creación][/rest/api/cosmos-db/create-a-document#status-codes], podríamos optar por crear código de control de errores para cada código de estado posible para esta operación.  En este laboratorio, solo se considerará de esta lista el código de estado 403 a 409.  Todos los demás códigos de estado devueltos solo mostrarán el mensaje de error del sistema.

    > &#128221; Tenga en cuenta que, aunque codificaremos una tarea con errores para una excepción 403, en este laboratorio no generaremos una excepción 403.

1. Vamos a agregar el control de errores para la función denominada **CompleteTaskOnCosmosDB**. Busque el bucle **while** en la función **Main** en la línea **45** y encapsule las llamadas de **CompleteTaskOnCosmosDB** con código de control de errores. Reemplazaremos la instrucción **CompleteTaskOnCosmosDB** en la línea **47** para el código siguiente.  Lo primero que hay que observar en este nuevo código es que en la **captura** capturamos una excepción de tipo de clase **CosmosException**.  Esta clase incluye la propiedad **StatusCode**, que devuelve el código de estado de finalización de la solicitud del servicio Azure Cosmos DB. La propiedad **StatusCode** es de tipo **System.Net.HttpStatusCode**, podemos usar este valor y compararlo con los nombres de campo del [código de estado HTTP][dotnet/api/system.net.httpstatuscode] de .NET.  

    ```C#
        try
        {
            await CompleteTaskOnCosmosDB(consoleinputcharacter, CustomersDB_Customer_container);
        }
        catch (CosmosException e)
        {
                    switch (e.StatusCode.ToString())
                    {
                        case ("Conflict"):
                            Console.WriteLine("Insert Failed. Response Code (409).");
                            Console.WriteLine("Can not insert a duplicate partition key, customer with the same ID already exists."); 
                            break;
                        case ("Forbidden"):
                            Console.WriteLine("Response Code (403).");
                            Console.WriteLine("The request was forbidden to complete. Some possible reasons for this exception are:");
                            Console.WriteLine("Firewall blocking requests.");
                            Console.WriteLine("Partition key exceeding storage.");
                            Console.WriteLine("Non-data operations are not allowed.");
                            break;
                        default:
                            Console.WriteLine(e.Message);
                            break;
                    }

        }

    ```

1. Guarde el archivo y, dado que se bloqueó, necesitamos volver a ejecutar el programa Menú, así que ejecute el comando:

    ```
    dotnet run
    ```
 
1. De nuevo, seleccione **1** y **ENTRAR** para insertar el primer documento. Esta vez no nos bloqueamos, pero obtenemos un mensaje más descriptivo de lo que sucedió.

    ```
    Insert Failed. 
    Response Code (409).
    Can not insert a duplicate partition key, customer with the same ID already exists.
    ```

1. Este código agregó el control de errores para las excepciones *403* y *409*, ahora agregaremos adicionalmente código para algunos tipos de comunicación comunes de excepciones. Hay tres tipos de comunicación comunes de excepciones: *429*, *503* y *408* o demasiadas solicitudes, servicio no disponible y tiempo de espera de solicitud respectivamente. En torno a la línea *66* ahora debería haber una instrucción **default**, por lo que debe agregar el código siguiente justo después de la instrucción **break;** y justo antes de la instrucción **predeterminada**.  El código comprobará si encontramos alguna de estas excepciones de comunicación y, si es así, esperará 10 segundos y, a continuación, intentará insertar el documento una vez más.  Vamos a agregar más allá del código:

    ```C#
                        case ("TooManyRequests"):
                        case ("ServiceUnavailable"):
                        case ("RequestTimeout"):
                            // Check if the issues are related to connectivity and if so, wait 10 seconds to retry.
                            await Task.Delay(10000); // Wait 10 seconds
                            try
                            {
                                Console.WriteLine("Try one more time...");
                                await CompleteTaskOnCosmosDB(consoleinputcharacter, CustomersDB_Customer_container);
                            }
                            catch (CosmosException e2)
                            {
                                Console.WriteLine("Insert Failed. " + e2.Message);
                                Console.WriteLine("Can not insert a duplicate partition key, Connectivity issues encountered.");
                                break;
                            }
                            break;
    ```

    > &#128221; Tenga en cuenta que, aunque codificaremos una tarea de qué hacer si encontramos una excepción 429, 503 o 408, en este laboratorio no generaremos un error con ese tipo de excepción.

1. Nuestra función **Main** debería tener ahora un aspecto similar al siguiente:

    ```C#
        public static async Task Main(string[] args)
        {

            CosmosClient client = new CosmosClient(connectionString,new CosmosClientOptions() { AllowBulkExecution = true, MaxRetryAttemptsOnRateLimitedRequests = 50, MaxRetryWaitTimeOnRateLimitedRequests = new TimeSpan(0,1,30)});

            Console.WriteLine("Creating Azure Cosmos DB Databases and containers");

            Database CustomersDB = await client.CreateDatabaseIfNotExistsAsync("CustomersDB");
            Container CustomersDB_Customer_container = await CustomersDB.CreateContainerIfNotExistsAsync(id: "Customer", partitionKeyPath: "/id", throughput: 400);

            Console.Clear();
            Console.WriteLine("1) Add Document 1 with id = '0C297972-BE1B-4A34-8AE1-F39E6AA3D828'");
            Console.WriteLine("2) Add Document 2 with id = 'AAFF2225-A5DD-4318-A6EC-B056F96B94B7'");
            Console.WriteLine("3) Delete Document 1 with id = '0C297972-BE1B-4A34-8AE1-F39E6AA3D828'");
            Console.WriteLine("4) Delete Document 2 with id = 'AAFF2225-A5DD-4318-A6EC-B056F96B94B7'");
            Console.WriteLine("5) Exit");
            Console.Write("\r\nSelect an option: ");
    
            string consoleinputcharacter;
        
            while((consoleinputcharacter = Console.ReadLine()) != "5") 
            {
                 try
                 {
                     await CompleteTaskOnCosmosDB(consoleinputcharacter, CustomersDB_Customer_container);
                 }
                 catch (CosmosException e)
                 {
                     switch (e.StatusCode.ToString())
                     {
                        case ("Conflict"):
                            Console.WriteLine("Insert Failed. Response Code (409).");
                            Console.WriteLine("Can not insert a duplicate partition key, customer with the same ID already exists."); 
                            break;
                        case ("Forbidden"):
                            Console.WriteLine("Response Code (403).");
                            Console.WriteLine("The request was forbidden to complete. Some possible reasons for this exception are:");
                            Console.WriteLine("Firewall blocking requests.");
                            Console.WriteLine("Partition key exceeding storage.");
                            Console.WriteLine("Non-data operations are not allowed.");
                            break;
                        case ("TooManyRequests"):
                        case ("ServiceUnavailable"):
                        case ("RequestTimeout"):
                            // Check if the issues are related to connectivity and if so, wait 10 seconds to retry.
                            await Task.Delay(10000); // Wait 10 seconds
                            try
                            {
                                Console.WriteLine("Try one more time...");
                                await CompleteTaskOnCosmosDB(consoleinputcharacter, CustomersDB_Customer_container);
                            }
                            catch (CosmosException e2)
                            {
                                Console.WriteLine("Insert Failed. " + e2.Message);
                                Console.WriteLine("Can not insert a duplicate partition key, Connectivity issues encountered.");
                                break;
                            }
                            break;
                        default:
                            Console.WriteLine(e.Message);
                            break;
                     }
                }
                

                Console.WriteLine("Choose an action:");
                Console.WriteLine("1) Add Document 1 with id = '0C297972-BE1B-4A34-8AE1-F39E6AA3D828'");
                Console.WriteLine("2) Add Document 2 with id = 'AAFF2225-A5DD-4318-A6EC-B056F96B94B7'");
                Console.WriteLine("3) Delete Document 1 with id = '0C297972-BE1B-4A34-8AE1-F39E6AA3D828'");
                Console.WriteLine("4) Delete Document 2 with id = 'AAFF2225-A5DD-4318-A6EC-B056F96B94B7'");
                Console.WriteLine("5) Exit");
                Console.Write("\r\nSelect an option: ");
            }
        }
    ```

1. Tenga en cuenta que la función **CreateDocument2** también se corregirá mediante los cambios anteriores.

1. Por último, las funciones **DeleteDocument1** y **DeleteDocument2** también necesitan reemplazar el código siguiente para el código de control de errores adecuado similar a la función **CreateDocument1**. La única diferencia con estas funciones, además de usar **DeleteItemAsync** en lugar de **CreateItemAsync**, es que los [códigos de estado de eliminación][/rest/api/cosmos-db/delete-a-document] son diferentes de los códigos de estado de inserción. Para las eliminaciones, solo tenemos en cuenta el código de estado **404**, que indica un documento que no se puede encontrar. Permite actualizar el control de errores de la llamada de función **CompleteTaskOnCosmosDB** con mayúsculas y minúsculas adicionales.  En la función **Main**, es necesario agregar el código siguiente encima del caso **default**:

    ```C#
                    case ("NotFound"):
                        Console.WriteLine("Delete Failed. Response Code (404).");
                        Console.WriteLine("Can not delete customer, customer not found.");
                        break;         
    ```

1. Guarde el archivo.

1. Una vez que haya terminado de corregir todas las funciones, pruebe todas las opciones de menú varias veces para asegurarse de que la aplicación devuelve un mensaje al encontrar una excepción y no se bloquea.  Si la aplicación se bloquea, corrija los errores y vuelva a ejecutar el comando:

    ```
    dotnet run
    ```


1. No mire, pero una vez que haya terminado, los códigos `Main` deben tener un aspecto similar al siguiente.

    ```C#
        public static async Task Main(string[] args)
        {
            CosmosClient client = new CosmosClient(connectionString,new CosmosClientOptions() { AllowBulkExecution = true, MaxRetryAttemptsOnRateLimitedRequests = 50, MaxRetryWaitTimeOnRateLimitedRequests = new TimeSpan(0,1,30)});

            Console.WriteLine("Creating Azure Cosmos DB Databases and containers");

            Database CustomersDB = await client.CreateDatabaseIfNotExistsAsync("CustomersDB");
            Container CustomersDB_Customer_container = await CustomersDB.CreateContainerIfNotExistsAsync(id: "Customer", partitionKeyPath: "/id", throughput: 400);

            Console.Clear();
            Console.WriteLine("1) Add Document 1 with id = '0C297972-BE1B-4A34-8AE1-F39E6AA3D828'");
            Console.WriteLine("2) Add Document 2 with id = 'AAFF2225-A5DD-4318-A6EC-B056F96B94B7'");
            Console.WriteLine("3) Delete Document 1 with id = '0C297972-BE1B-4A34-8AE1-F39E6AA3D828'");
            Console.WriteLine("4) Delete Document 2 with id = 'AAFF2225-A5DD-4318-A6EC-B056F96B94B7'");
            Console.WriteLine("5) Exit");
            Console.Write("\r\nSelect an option: ");
    
            string consoleinputcharacter;
        
            while((consoleinputcharacter = Console.ReadLine()) != "5") 
            {
                    try
                    {
                        await CompleteTaskOnCosmosDB(consoleinputcharacter, CustomersDB_Customer_container);
                    }
                    catch (CosmosException e)
                    {
                        switch (e.StatusCode.ToString())
                        {
                            case ("Conflict"):
                                Console.WriteLine("Insert Failed. Response Code (409).");
                                Console.WriteLine("Can not insert a duplicate partition key, customer with the same ID already exists."); 
                                break;
                            case ("Forbidden"):
                                Console.WriteLine("Response Code (403).");
                                Console.WriteLine("The request was forbidden to complete. Some possible reasons for this exception are:");
                                Console.WriteLine("Firewall blocking requests.");
                                Console.WriteLine("Partition key exceeding storage.");
                                Console.WriteLine("Non-data operations are not allowed.");
                                break;
                            case ("TooManyRequests"):
                            case ("ServiceUnavailable"):
                            case ("RequestTimeout"):
                                // Check if the issues are related to connectivity and if so, wait 10 seconds to retry.
                                await Task.Delay(10000); // Wait 10 seconds
                                try
                                {
                                    Console.WriteLine("Try one more time...");
                                    await CompleteTaskOnCosmosDB(consoleinputcharacter, CustomersDB_Customer_container);
                                }
                                catch (CosmosException e2)
                                {
                                    Console.WriteLine("Insert Failed. " + e2.Message);
                                    Console.WriteLine("Can not insert a duplicate partition key, Connectivity issues encountered.");
                                    break;
                                }
                                break;    
                            case ("NotFound"):
                                Console.WriteLine("Delete Failed. Response Code (404).");
                                Console.WriteLine("Can not delete customer, customer not found.");
                                break; 
                            default:
                                Console.WriteLine(e.Message);
                                break;
                        }

                    }

                Console.WriteLine("Choose an action:");
                Console.WriteLine("1) Add Document 1 with id = '0C297972-BE1B-4A34-8AE1-F39E6AA3D828'");
                Console.WriteLine("2) Add Document 2 with id = 'AAFF2225-A5DD-4318-A6EC-B056F96B94B7'");
                Console.WriteLine("3) Delete Document 1 with id = '0C297972-BE1B-4A34-8AE1-F39E6AA3D828'");
                Console.WriteLine("4) Delete Document 2 with id = 'AAFF2225-A5DD-4318-A6EC-B056F96B94B7'");
                Console.WriteLine("5) Exit");
                Console.Write("\r\nSelect an option: ");
            }
        }
    ```

## Conclusión

Incluso los desarrolladores más noveles saben que el control de errores adecuado debe agregarse a todo el código. Aunque el control de errores en este código es sencillo, debe haber proporcionado los conceptos básicos sobre los componentes de excepción de Azure Cosmos DB que le permitirán crear soluciones sólidas de control de errores en el código.


[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/dotnet/core/tools/dotnet-add-package]: https://docs.microsoft.com/dotnet/core/tools/dotnet-add-package
[docs.microsoft.com/dotnet/core/tools/dotnet-run]: https://docs.microsoft.com/dotnet/core/tools/dotnet-run
[nuget.org/packages/microsoft.azure.cosmos/3.22.1]: https://www.nuget.org/packages/Microsoft.Azure.Cosmos/3.22.1
[/rest/api/cosmos-db/create-a-document#status-codes]:https://docs.microsoft.com/rest/api/cosmos-db/create-a-document#status-codes
[dotnet/api/system.net.httpstatuscode]:https://docs.microsoft.com/dotnet/api/system.net.httpstatuscode?view=net-6.0
[/rest/api/cosmos-db/delete-a-document]:https://docs.microsoft.com/rest/api/cosmos-db/delete-a-document#status-codes

