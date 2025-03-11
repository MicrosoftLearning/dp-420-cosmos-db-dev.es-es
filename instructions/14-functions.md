---
lab:
  title: Procesamiento de datos de Azure Cosmos DB for NoSQL mediante Azure Functions
  module: Module 7 - Integrate Azure Cosmos DB for NoSQL with Azure services
---

# Procesamiento de datos de Azure Cosmos DB for NoSQL mediante Azure Functions

El desencadenador de Azure Cosmos DB para Azure Functions se implementa mediante un procesador de fuente de cambios. Puede crear funciones que respondan a las operaciones de creación y actualización en el contenedor de Azure Cosmos DB for NoSQL con este conocimiento. Si ha implementado manualmente un procesador de fuente de cambios, la configuración de Azure Functions es similar.

En este laboratorio, crearás una aplicación de funciones y todos sus recursos necesarios, que supervisa la información de registro de la base de datos y de los resultados para cada operación detectada dentro de él.

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
    | **Capacity mode (Modo de capacidad)** | *Sin servidor* |

    > &#128221; Es posible que los entornos de laboratorio tengan restricciones que le impidan crear un nuevo grupo de recursos. Si es así, use el grupo de recursos existente creado previamente.

1. Espere a que se complete la tarea de implementación antes de continuar con esta tarea.

1. Vaya al recurso de cuenta de **Azure Cosmos DB** recién creado y vaya al panel **Claves**.

1. Este panel contiene los detalles de conexión y las credenciales necesarias para conectarse a la cuenta desde el SDK. Específicamente:

    1. Observe el campo **URI**. Usará este valor de **endpoint** más adelante en este ejercicio.

    1. Observe el campo **PRIMARY KEY**. Usará este valor de **clave** más adelante en este ejercicio.

1. En el menú de recursos, seleccione **Explorador de datos**.

1. En el panel **Data Explorer**, expanda **Nuevo contenedor** y, a continuación, seleccione **Nueva base de datos**.

1. En la ventana emergente **Nueva base de datos**, escriba los valores siguientes para cada configuración y, después, seleccione **Aceptar**:

    | **Configuración** | **Valor** |
    | --: | :-- |
    | **Id. de base de datos** | *``cosmicworks``* |

1. De nuevo en el panel **Data Explorer**, observe el nodo de base de datos **cosmicworks** dentro de la jerarquía.

1. En el panel **Data Explorer**, seleccione **Nuevo contenedor**.

1. En la ventana emergente **Nuevo contenedor**, escriba los valores siguientes para cada configuración y, después, seleccione **Aceptar**:

    | **Configuración** | **Valor** |
    | --: | :-- |
    | **Id. de base de datos** | *Usar existente* &vert; *cosmicworks* |
    | **Id. de contenedor** | *``products``* |
    | **Clave de partición** | *``/categoryId``* |

1. De nuevo en el panel **Data Explorer**, expanda el nodo de base de datos **cosmicworks** y observe el nodo contenedor **productos** dentro de la jerarquía.

1. En el panel **Data Explorer**, vuelva a seleccionar **Nuevo contenedor**.

1. En la ventana emergente **Nuevo contenedor**, escriba los valores siguientes para cada configuración y, después, seleccione **Aceptar**:

    | **Configuración** | **Valor** |
    | --: | :-- |
    | **Id. de base de datos** | *Usar existente* &vert; *cosmicworks* |
    | **Id. de contenedor** | *``productslease``* |
    | **Clave de partición** | *``/id``* |

1. De nuevo en el panel **Data Explorer**, expanda el nodo de base de datos **cosmosworks** y observe el nodo de contenedor **productslease** dentro de la jerarquía.

1. Vuelva al **Inicio** de Azure Portal.

## Creación de Application Insight

Antes de crear el *Azure Funtion Application*, deberá habilitar una *Azure Application Insight* para que pueda supervisar la diversión de la aplicación. Application Insight necesitará primero un *área de trabajo de Log Analytics*.

1. En el cuadro de búsqueda, busque **áreas de trabajo de Log Analytics**.

1. Seleccione esta opción para **+Crear** una nueva área de trabajo de *Log Analytics*.

1. En el cuadro de diálogo **área de trabajo de Log Analytics**, escriba los valores siguientes para cada configuración y, a continuación, seleccione **Revisar y crear** y, a continuación, seleccione **Crear**:

    | **Configuración** | **Valor** |
    | ---: | :--- |
    | **Suscripción** | *Su suscripción de Azure existente* |
    | **Grupo de recursos** | *Seleccione un grupo de recursos ya existente o cree uno nuevo* |
    | **Nombre** | *``lab14laworkspace``* |
    | **Ubicación** | *seleccione cualquier región disponible* |

1. Una vez creada el *área de trabajo de Log Analytics*, en el cuadro de búsqueda, busque **Application Insights**.

1. Seleccione esta opción para **+Crear** una nueva *Application Insight*.

1. En el cuadro de diálogo **Application Insights**, escriba los valores siguientes para cada configuración y, a continuación, seleccione **Revisar y crear** y, a continuación, seleccione **Crear**:

    | **Configuración** | **Valor** |
    | ---: | :--- |
    | **Suscripción (ambas entradas)** | *Su suscripción de Azure existente* |
    | **Grupo de recursos** | *Seleccione un grupo de recursos ya existente o cree uno nuevo* |
    | **Nombre** | *``lab14appinsight``* |
    | **Ubicación** | *seleccione cualquier región disponible* |
    | **Área de trabajo de Log Analytics** | *lab14laworkspace* |

Ahora debería poder supervisar la función de la aplicación.

## Creación de una aplicación de funciones de Azure y una función desencadenada por Azure Cosmos DB

Para empezar a escribir código, deberá crear el recurso de Azure Functions y sus recursos dependientes (Application Insights, Storage) mediante el asistente para la creación.

1. Selecciona **+ Crear un recurso**, busca *Funciones* y, a continuación, crea un nuevo recurso de cuenta de **Aplicación de funciones**. Selecciona **Consumo** como opción de hospedaje, configura la aplicación con la siguiente configuración y deja todas las opciones restantes en sus valores predeterminados:

    | **Configuración** | **Valor** |
    | ---: | :--- |
    | **Suscripción** | *Su suscripción de Azure existente* |
    | **Grupo de recursos** | *Seleccione un grupo de recursos ya existente o cree uno nuevo* |
    | **Nombre** | *Escriba un nombre único global*. |
    | **Publicar** | *Código* |
    | **Pila en tiempo de ejecución** | *.NET* |
    | **Versión** | *8 (LTS) en curso* |
    | **Región** | *seleccione cualquier región disponible* |
    | **Cuenta de almacenamiento** | *Creación de una cuenta de almacenamiento nueva* |

    > &#128221; Es posible que los entornos de laboratorio tengan restricciones que le impidan crear un nuevo grupo de recursos. Si es así, use el grupo de recursos existente creado previamente.

1. Espere a que se complete la tarea de implementación antes de continuar con esta tarea.

1. Ve al recurso de cuenta de **Azure Functions** recién creado y, en la página de información general, selecciona **Crear función**.

1. En el emergente **Crear función**, cree una función con la siguiente configuración, dejando todas las opciones restantes en sus valores predeterminados:

    | **Configuración** | **Valor** |
    | ---: | :--- |
    | **Seleccione una plantilla** | *Desencadenador de Azure Cosmos DB* |
    | **Nombre de la función** | *``ItemsListener``* |
    | **Conexión de la cuenta de Cosmos DB** | *Seleccionar nueva* &vert; *Seleccionar cuenta de Azure Cosmos DB* &vert; *Seleccione la cuenta de Azure Cosmos DB que creó anteriormente* |
    | **Nombre de la base de datos** | *``cosmicworks``* |
    | **Nombre de colección** | *``products``* |
    | **Nombre de la colección de concesiones** | *``productslease``* |
    | **Crear colección de concesiones si no existe** | *No* |

## Implementación del código de función en .NET

La función que creó anteriormente es un script de C# que se edita en el portal. Ahora usará el portal para escribir una función corta para generar el identificador único de cualquier elemento insertado o actualizado en el contenedor.

1. En el panel **ItemsListener**&vert;**Code + Test**, ve al editor del script **run.csx** y elimina su contenido.

1. En el área del editor, haga referencia a la biblioteca **Microsoft.Azure.DocumentDB.Core**:

    ```
    #r "Microsoft.Azure.DocumentDB.Core"
    ```

1. Agregue los bloques de uso para los espacios de nombres **System**, **System.Collections.Generic**, y [Microsoft.Azure.Documents][docs.microsoft.com/dotnet/api/microsoft.azure.documents]:

    ```
    using System;
    using System.Collections.Generic;
    using Microsoft.Azure.Documents;
    ```

1. Cree un nuevo método estático denominado **Run** que tenga dos parámetros:

    1. Parámetro denominado **entrada** de tipo **IReadOnlyList\<\>** con un tipo genérico de [Document][docs.microsoft.com/dotnet/api/microsoft.azure.documents.document].

    1. Parámetro denominado **registro** de tipo **ILogger**.

    ```
    public static void Run(IReadOnlyList<Document> input, ILogger log)
    {
    }
    ```

1. En el método **Run**, invoque el método**LogInformation** del **registro** variable que pasa una cadena que calcula el recuento de elementos del lote actual:

    ```
    log.LogInformation($"# Modified Items:\t{input?.Count ?? 0}"); 
    ```

1. Aún dentro del método **Run**, cree un bucle foreach que itera en la variable de **entrada** mediante la variable **elemento** para representar una instancia de tipo **Document**:

    ```
    foreach(Document item in input)
    {
    }
    ```

1. Dentro del bucle foreach del método **Run**, invoque el método **LogInformation** de la variable de **registro** pasando una cadena que imprime la propiedad [Id][docs.microsoft.com/dotnet/api/microsoft.azure.documents.resource.id] de la variable **elemento**:

    ```
    log.LogInformation($"Detected Operation:\t{item.Id}");
    ```

1. Una vez que haya terminado, el archivo de código debería incluir:
  
    ```
    #r "Microsoft.Azure.DocumentDB.Core"
    
    using System;
    using System.Collections.Generic;
    using Microsoft.Azure.Documents;
    
    public static void Run(IReadOnlyList<Document> input, ILogger log)
    {
        log.LogInformation($"# Modified Items:\t{input?.Count ?? 0}");
    
        foreach(Document item in input)
        {
            log.LogInformation($"Detected Operation:\t{item.Id}");
        }
    }
    ```

1. Expande la sección **Registros** en la parte inferior de la página, expande los **registros de App Insights** y selecciona **Registros del sistema de archivos** para conectarte a los registros de streaming de la función actual.

    > &#128161; Puede tardar un par de segundos en conectarse al servicio de registro de streaming. Verá un mensaje en la salida del registro una vez que esté conectado.

1. **Guardar** el código de función actual.

1. Observe el resultado de la compilación de código de C#. Debería esperar ver un mensaje **Compilación correcta** al final de la salida del registro.

    > &#128221; Es posible que vea mensajes de advertencia en la salida del registro. Estas advertencias no afectarán a este laboratorio.

1. **Maximizar** la sección de registro para expandir la ventana de salida para rellenar el espacio máximo disponible.

    > &#128221; Usará otra herramienta para generar elementos en el contenedor de Azure Cosmos DB for NoSQL. Una vez que genere los elementos, volverá a esta ventana del explorador para observar la salida. No cierre la ventana del explorador prematuramente.

## Inicialización de la cuenta de Azure Cosmos DB for NoSQL con datos de ejemplo

Usará una utilidad de línea de comandos que crea una base de datos de **cosmicworks** y un contenedor de **productos**. A continuación, la herramienta creará un conjunto de elementos que observará mediante el procesador de fuente de cambios que se ejecuta en la ventana del terminal.

1. Inicie **Visual Studio Code**.

    > &#128221; Si aún no está familiarizado con la interfaz de Visual Studio Code, revise la [Guía de introducción para Visual Studio Code][code.visualstudio.com/docs/getstarted]

1. En **Visual Studio Code**, abra el menú **Terminal** y, a continuación, seleccione **Nuevo terminal** para abrir una nueva instancia de terminal.

1. Instale la herramienta de línea de comandos [cosmicworks][nuget.org/packages/cosmicworks] para su uso global en la máquina.

    ```
    dotnet tool install cosmicworks --global --version 1.*
    ```

    > &#128161; Este comando puede tardar un par de minutos en completarse. Este comando generará el mensaje de advertencia (*Tool "cosmicworks" ya está instalado) si ya ha instalado la versión más reciente de esta herramienta en el pasado.

1. Ejecute cosmicworks para inicializar la cuenta de Azure Cosmos DB con las siguientes opciones de línea de comandos:

    | **Opción** | **Valor** |
    | ---: | :--- |
    | **--endpoint** | *El valor del punto de conexión que copió anteriormente en este laboratorio* |
    | **--key** | *El valor de clave que copió anteriormente en este laboratorio* |
    | **--datasets** | *product* |

    ```
    cosmicworks --endpoint <cosmos-endpoint> --key <cosmos-key> --datasets product
    ```

    > &#128221; Por ejemplo, si el punto de conexión es **https&shy;://dp420.documents.azure.com:443/** y la clave es **fDR2ci9QgkdkvERTQ==**, el comando sería: ``cosmicworks --endpoint https://dp420.documents.azure.com:443/ --key fDR2ci9QgkdkvERTQ== --datasets product``

1. Espere a que el comando **cosmicworks** termine de poblar la cuenta con una base de datos, un contenedor y elementos.

1. Cierre el terminal integrado.

1. Cierre **Visual Studio Code**.

1. Vuelva a la ventana o pestaña del explorador abierto actualmente con la sección registro de Azure Functions expandida.

1. Observe la salida del registro de la función. El terminal genera un mensaje **Operación detectada** para cada cambio que se le envió mediante la fuente de cambios. Las operaciones se procesan por lotes en grupos de aproximadamente 100 operaciones.

1. Cierre la ventana o pestaña del explorador web.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/dotnet/api/microsoft.azure.documents]: https://docs.microsoft.com/dotnet/api/microsoft.azure.documents
[docs.microsoft.com/dotnet/api/microsoft.azure.documents.document]: https://docs.microsoft.com/dotnet/api/microsoft.azure.documents.document
[docs.microsoft.com/dotnet/api/microsoft.azure.documents.resource.id]: https://docs.microsoft.com/dotnet/api/microsoft.azure.documents.resource.id
