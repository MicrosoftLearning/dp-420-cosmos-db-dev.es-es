---
lab:
  title: "Optimización de la directiva de indexación de un contenedor de Azure Cosmos\_DB for NoSQL para operaciones comunes"
  module: Module 10 - Optimize query and operation performance in Azure Cosmos DB for NoSQL
---

# Optimización de la directiva de indexación de un contenedor de Azure Cosmos DB for NoSQL para operaciones comunes

En el caso de cargas de trabajo o cargas de trabajo con grandes objetos JSON, puede ser ventajoso optimizar la directiva de indexación solo para las propiedades de índice que sabe que desea usar en las consultas.

En este laboratorio, usaremos una aplicación de .NET de prueba para insertar un elemento JSON grande en un contenedor de Azure Cosmos DB for NoSQL mediante la directiva de indexación predeterminada y, a continuación, usaremos una directiva de indexación que se ha ajustado ligeramente.

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
    | **Capacity mode (Modo de capacidad)** | *Sin servidor* |

    > &#128221; Es posible que los entornos de laboratorio tengan restricciones que le impidan crear un nuevo grupo de recursos. Si es así, use el grupo de recursos existente creado previamente.

1. Espere a que se complete la tarea de implementación antes de continuar con esta tarea.

1. Vaya al recurso de cuenta de **Azure Cosmos DB** recién creado y vaya al panel **Data Explorer**.

1. En el panel **Data Explorer**, seleccione **Nuevo contenedor**.

1. En la ventana emergente **Nuevo contenedor**, escriba los valores siguientes para cada configuración y, después, seleccione **Aceptar**:

    | **Configuración** | **Valor** |
    | --: | :-- |
    | **Id. de base de datos** | *Crear nuevo* &vert; *``cosmicworks``* |
    | **Id. de contenedor** | *``products``* |
    | **Clave de partición** | *``/categoryId``* |

1. De nuevo en el panel **Data Explorer**, expanda el nodo de base de datos **cosmicworks** y observe el nodo contenedor **products** dentro de la jerarquía.

1. En la hoja de recursos, vaya al panel **Claves**.

1. Este panel contiene los detalles de conexión y las credenciales necesarias para conectarse a la cuenta desde el SDK. Específicamente:

    1. Observe el campo **URI**. Usará este valor de **endpoint** más adelante en este ejercicio.

    1. Observe el campo **PRIMARY KEY**. Usará este valor **key** más adelante en este ejercicio.

1. Vuelva a **Visual Studio Code**.

## Ejecución de la aplicación .NET de prueba mediante la directiva de indexación predeterminada

Este laboratorio tiene una aplicación .NET de prueba generada previamente que tomará un objeto JSON grande y creará un nuevo elemento en el contenedor de Azure Cosmos DB for NoSQL. Una vez completada la operación de escritura única, la aplicación generará el identificador único del elemento y el cargo de RU en la ventana de la consola.

1. En el panel **Explorer**, vaya a la carpeta **23-index-optimization**.

1. Abra el menú contextual de la carpeta **23-index-optimization** y, a continuación, seleccione **Abrir en terminal integrado** para abrir una nueva instancia de terminal.

    > &#128221; Este comando abrirá el terminal con el directorio inicial ya establecido en la carpeta **23-index-optimization**.

1. Compile el proyecto mediante el comando [dotnet build][docs.microsoft.com/dotnet/core/tools/dotnet-build]:

    ```
    dotnet build
    ```

    > &#128221; Es posible que vea una advertencia del compilador de que el **punto de conexión** y las variables de **clave** estén actualmente sin usar. Puede omitir esta advertencia de forma segura, ya que usará estas variables en esta tarea.

1. Cierre el terminal integrado.

1. Abra el archivo de código **script.cs**.

1. Busque la variable de **cadena** denominada **punto de conexión**. Establezca su valor en el valor **endpoint** de la cuenta de Azure Cosmos DB que creó anteriormente.
  
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

1. En **Visual Studio Code**, abra el menú contextual de la carpeta **23-index-optimization** y, a continuación, seleccione **Abrir en terminal integrado** para abrir una nueva instancia de terminal.

1. Compile y ejecute el proyecto mediante el comando **[dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run]**:

    ```
    dotnet run
    ```

1. Observe la salida del terminal. El identificador único del elemento y el cargo de solicitud de la operación (en RU) se deben imprimir en la consola.

1. Compile y ejecute el proyecto al menos dos veces más con el comando **[dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run]**. Observe el cargo de RU en la salida de la consola:

    ```
    dotnet run
    ```

1. Deje abierto el terminal integrado.

    > &#128221; Volverá a usar este terminal más adelante en este ejercicio. Es importante dejar abierto el terminal para poder comparar los cargos de RU originales y actualizados.

## Actualizar la directiva de indexación y volver a ejecutar la aplicación .NET

En este escenario de laboratorio se supone que nuestras consultas futuras se centran principalmente en las propiedades name y categoryName. Para optimizar nuestro elemento JSON grande, excluirá todos los demás campos del índice mediante la creación de una directiva de indexación que comience excluyendo todas las rutas de acceso. A continuación, la directiva incluirá selectivamente rutas de acceso específicas.

1. Vuelva al explorador web.

1. En el recurso de cuenta de **Azure Cosmos DB**, vaya al panel **Data Explorer**.

1. En **Data Explorer**, expanda el nodo de base de datos **cosmicworks**, expanda el nodo de contenedor **products** y, a continuación, seleccione **Configuración**.

1. En la pestaña **Configuración**, vaya a la sección **Directiva de indexación**.

1. Observe la directiva de indexación predeterminada:

    ```
    {
      "indexingMode": "consistent",
      "automatic": true,
      "includedPaths": [
        {
          "path": "/*"
        }
      ],
      "excludedPaths": [
        {
          "path": "/\"_etag\"/?"
        }
      ]
    }    
    ```

1. Reemplace la directiva de indexación por este objeto JSON modificado y, a continuación, **guarde** los cambios:

    ```
    {
      "indexingMode": "consistent",
      "automatic": true,
      "includedPaths": [
        {
          "path": "/name/?"
        },
        {
          "path": "/categoryName/?"
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

1. Vuelva a **Visual Studio Code**. Vuelva al terminal abierto.

1. Compile y ejecute el proyecto al menos dos veces más con el comando **[dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run]**. Observe la nueva carga de RU en la salida de la consola, que debe ser significativamente menor que la carga original. Dado que no indexa todas las propiedades del elemento, el costo de las escrituras es significativamente menor al actualizar el índice. Sin embargo, esto puede suponer un gran costo si sus lecturas necesitan consultar propiedades que no están indexadas.  

    ```
    dotnet run
    ```

    > &#128221; Si no ve un cargo de RU actualizado, es posible que tenga que esperar un par de minutos.

1. Vuelva al explorador web.

    > &#128221; Si la página **Directiva de indexación** no está abierta, vaya a **Data Explorer**, expanda el nodo de base de datos **cosmicworks**, expanda el nodo de contenedor **products**, seleccione **Configuración** y vaya a la sección **Directiva de indexación**.

1. Reemplace la directiva de indexación por este objeto JSON modificado y, a continuación, **guarde** los cambios:

    ```
    {
      "indexingMode": "none"
    }
    ```

1. Cierre la ventana o pestaña del explorador web.

1. Vuelva a **Visual Studio Code**. Vuelva al terminal abierto.

1. Compile y ejecute el proyecto al menos dos veces más con el comando **[dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run]**. Observe la nueva carga de RU en la salida de la consola, que debe ser mucho menor que la carga original.  ¿Cómo puede ser? Dado que este script mide las RU al escribir el elemento, al elegir no tener ningún índice, no hay ninguna sobrecarga para mantener ese índice. La otra cara de la moneda es que, aunque tus escrituras generarán menos RU, las lecturas serán muy costosas.

    ```
    dotnet run
    ```

    > &#128221; Si no ve un cargo de RU actualizado, es posible que tenga que esperar un par de minutos.

1. Cierre **Visual Studio Code**.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/dotnet/core/tools/dotnet-run]: https://docs.microsoft.com/dotnet/core/tools/dotnet-run
