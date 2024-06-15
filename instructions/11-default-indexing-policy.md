---
lab:
  title: "Revisión de la directiva de indexación predeterminada para un contenedor de Azure Cosmos\_DB for NoSQL con el portal"
  module: Module 6 - Define and implement an indexing strategy for Azure Cosmos DB for NoSQL
---

# Revisión de la directiva de indexación predeterminada para un contenedor de Azure Cosmos DB for NoSQL con el portal

Cada contenedor de Azure Cosmos DB tiene una directiva de indexación que dirige al servicio sobre cómo indexar elementos dentro del contenedor. De forma predeterminada, esta directiva de indexación indexa cada propiedad de cada elemento. La directiva de indexación predeterminada facilita la introducción a Azure Cosmos DB rápidamente, ya que no tiene que pensar en la indexación, el rendimiento y la administración al principio de un proyecto.

En este laboratorio, observará y manipulará la directiva de índice predeterminada para algunos contenedores mediante el Explorador de datos.

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

    1. Observe el campo **PRIMARY KEY**. Usará este valor de **clave** más adelante en este ejercicio.


## Inicialización de la cuenta de Azure Cosmos DB for NoSQL con datos

La herramienta de línea de comandos [cosmicworks][nuget.org/packages/cosmicworks] implementa datos de ejemplo en cualquier cuenta de Azure Cosmos DB for NoSQL. La herramienta es de código abierto y está disponible a través de NuGet. Instalará esta herramienta en Azure Cloud Shell y la usará para inicializar la base de datos.

1. Inicie **Visual Studio Code**.

1. En **Visual Studio Code**, abra el menú **Terminal** y, a continuación, seleccione **Nuevo terminal** para abrir una nueva instancia de terminal.

    > &#128221; Si aún no está familiarizado con la interfaz de Visual Studio Code, revise la [Guía de introducción para Visual Studio Code][code.visualstudio.com/docs/getstarted]

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

## Visualización y manipulación de la directiva de indexación predeterminada

Cuando un contenedor se crea mediante código, portal o herramienta; la directiva de indexación se establece en un valor predeterminado inteligente si no lo especifica de otro modo. Observará que la directiva de indexación predeterminada y realizar un cambio en la directiva.

1. Vuelva al explorador web.

1. En el recurso de cuenta de **Azure Cosmos DB**, vaya al panel del **Explorador de datos**.

1. En **Data Explorer**, expanda el nodo de la base de datos **cosmicworks** y, a continuación, observe el nodo contenedor **products** nuevo dentro del árbol de navegación de la **API NOSQL**.

1. Seleccione el nodo contenedor **products** dentro del árbol de navegación de la **API NOSQL** y, a continuación, seleccione **Nueva consulta SQL**.

1. Elimine el contenido del área del editor.

1. Cree una nueva consulta SQL que devuelva todos los documentos en los que el **nombre** equivalga a **HL Headset**:

    ```
    SELECT * FROM p WHERE p.name = 'HL Headset'
    ```

1. Seleccione **Ejecutar consulta**.

1. Observe los resultados de la consulta.

1. En la pestaña **Consulta**, seleccione **Estadísticas de consulta**.

1. Observe el valor del campo **Cargo de solicitud** en la sección **Estadísticas de consulta**.

    > &#128221; Todas las rutas de acceso se indexan actualmente, por lo que esta consulta debe ser relativamente eficaz.

1. Dentro del nodo contenedor **products** en el árbol de navegación de la **API NOSQL**, seleccione **Escala y configuración**.

1. Observe la directiva de indexación predeterminada dentro de la sección **Directiva de indexación**:

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

    > &#128221; Esta directiva predeterminada indexará todas las rutas de acceso posibles a excepción de **_etag**.

1. En el editor, reemplace el contenido de la directiva de indexación para indexar solo la ruta **/price**:

    ```
    {
      "indexingMode": "consistent",
      "automatic": true,
      "includedPaths": [
        {
          "path": "/price/?"
        }
      ],
      "excludedPaths": [
        {
          "path": "/*"
        }
      ]
    }
    ```

1. Seleccione **Guardar** para conservar los cambios.

1. Seleccione **Nueva consulta SQL**.

1. Elimine el contenido del área del editor.

1. Cree una nueva consulta SQL que devuelva todos los documentos en los que el **nombre** equivalga a **HL Headset**:

    ```
    SELECT * FROM p WHERE p.name = 'HL Headset'
    ```

1. Seleccione **Ejecutar consulta**.

1. Observe los resultados de la consulta.

1. En la pestaña **Consulta**, seleccione **Estadísticas de consulta**.

1. Observe el valor del campo **Cargo de solicitud** en la sección **Estadísticas de consulta**.

    > &#128221; Ahora que la propiedad **name** no está indexada, el cargo de solicitud ha aumentado.

1. Elimine el contenido del área del editor.

1. Cree una nueva consulta SQL que devuelva todos los documentos en los que el **precio** sea mayor que **3000 USD**:

    ```
    SELECT * FROM p WHERE p.price > 3000
    ```

1. Seleccione **Ejecutar consulta**.

1. Observe los resultados de la consulta.

1. En la pestaña **Consulta**, seleccione **Estadísticas de consulta**.

1. Observe el valor del campo **Cargo de solicitud** en la sección **Estadísticas de consulta**.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[nuget.org/packages/cosmicworks]: https://www.nuget.org/packages/cosmicworks/
