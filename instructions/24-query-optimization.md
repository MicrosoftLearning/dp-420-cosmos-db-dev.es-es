---
lab:
  title: "Optimización de la directiva de indexación de un contenedor de Azure Cosmos\_DB for NoSQL para una consulta específica"
  module: Module 10 - Optimize query and operation performance in Azure Cosmos DB for NoSQL
---

# Optimización de la directiva de indexación de un contenedor de Azure Cosmos DB for NoSQL para una consulta

Al planear una cuenta de Azure Cosmos DB for NoSQL, conocer nuestras consultas más populares puede ayudarnos a optimizar la directiva de indexación para que las consultas sean lo más eficaces posible.

En este laboratorio, usaremos el Explorador de datos para probar consultas SQL con la directiva de indexación predeterminada y una directiva de indexación que incluya un índice compuesto.

## Creación de una cuenta de Azure Cosmos DB for NoSQL

Azure Cosmos DB es un servicio de base de datos NoSQL basado en la nube que admite varias API. Al aprovisionar una cuenta de Azure Cosmos DB por primera vez, seleccionará cuál de las API que quiere que admita la cuenta (por ejemplo, **API Mongo** o **API NoSQL**). Una vez que la cuenta de Azure Cosmos DB for NoSQL haya terminado de aprovisionar, puede recuperar el punto de conexión y la clave y usarlos para conectarse a la cuenta de Azure Cosmos DB for NoSQL mediante el SDK de Azure para .NET o cualquier otro SDK que prefiera.

1. Vaya a Azure Portal (``portal.azure.com``) desde una nueva ventana o pestaña del explorador web.

1. Inicie sesión en el portal con las credenciales de Microsoft asociadas a su suscripción.

1. Seleccione **+ Crear un recurso**, busque *Cosmos DB* y, a continuación, cree un nuevo recurso de cuenta de **Azure Cosmos DB for NoSQL** con la siguiente configuración, dejando todas las opciones restantes en sus valores predeterminados:

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

    1. Observe el campo **URI**. Usará este valor de **punto de conexión** más adelante en este ejercicio.

    1. Observe el campo **CLAVE PRINCIPAL**. Usará este valor de **clave** más adelante en este ejercicio.

1. Abra **Visual Studio Code**.

## Inicialización de la cuenta de Azure Cosmos DB for NoSQL con datos de ejemplo

Usará una utilidad de línea de comandos que crea una base de datos de **cosmicworks** y un contenedor de **productos**. A continuación, la herramienta creará un conjunto de elementos que observará mediante el procesador de fuente de cambios que se ejecuta en la ventana del terminal.

1. En **Visual Studio Code**, abra el menú **Terminal** y, a continuación, seleccione **Nuevo terminal** para abrir un nuevo terminal.

1. Instale la herramienta de línea de comandos [cosmicworks][nuget.org/packages/cosmicworks] para su uso global en la máquina.

    ```
    dotnet tool install cosmicworks --global --version 1.*
    ```

    > &#128161; Este comando puede tardar un par de minutos en completarse. Este comando generará el mensaje de advertencia (*Tool "cosmicworks" ya está instalado) si ya ha instalado la versión más reciente de esta herramienta en el pasado.

1. Ejecute cosmicworks para inicializar la cuenta de Azure Cosmos DB con las siguientes opciones de línea de comandos:

    | **Opción** | **Valor** |
    | ---: | :--- |
    | **--endpoint** | *El valor del punto de conexión que copió anteriormente en este laboratorio* |
    | **--key** | *El valor de clave que copió anteriormente en este laboratorio* |
    | **--datasets** | *product* |

    ```
    cosmicworks --endpoint <cosmos-endpoint> --key <cosmos-key> --datasets product
    ```

    > &#128221; Por ejemplo, si el punto de conexión es **https&shy;://dp420.documents.azure.com:443/** y la clave es **fDR2ci9QgkdkvERTQ==**, el comando sería ``cosmicworks --endpoint https://dp420.documents.azure.com:443/ --key fDR2ci9QgkdkvERTQ== --datasets product``

1. Espere a que el comando **cosmicworks** termine de poblar la cuenta con una base de datos, un contenedor y elementos.

1. Cierre el terminal integrado.

1. Cierre **Visual Studio Code** y vuelva al explorador.

## Ejecución de consultas SQL y medida de su cargo por unidad de solicitud

Antes de modificar la directiva de indexación, primero ejecutará algunas consultas SQL de ejemplo para obtener un cargo de unidad de solicitud de línea base expresado en RU.

1. En el recurso de cuenta de **Azure Cosmos DB**, vaya al panel **Data Explorer**.

1. En **Data Explorer**, expanda el nodo de base de datos **cosmicworks**, seleccione el nodo de contenedor **products** y, a continuación, seleccione **Nueva consulta SQL**.

1. Seleccione **Ejecutar consulta** para ejecutar la consulta predeterminada:

    ```
    SELECT * FROM c
    ```

1. Observe los resultados de la consulta. Seleccione **Estadísticas de consulta** para ver el cargo de unidad de solicitud en RU.

1. Elimine el contenido del área del editor.

1. Cree una nueva consulta SQL que devuelva los tres valores de todos los documentos:

    ```
    SELECT 
        p.name,
        p.categoryName,
        p.price
    FROM
        products p    
    ```

1. Seleccione **Ejecutar consulta**.

1. Observe los resultados y las estadísticas de la consulta. El cargo de unidad de solicitud es casi el mismo que la primera consulta.

1. Elimine el contenido del área del editor.

1. Cree una nueva consulta SQL que devuelva tres valores de todos los documentos ordenados por **categoryName**:

    ```
    SELECT 
        p.name,
        p.categoryName,
        p.price
    FROM
        products p
    ORDER BY
        p.categoryName DESC
    ```

1. Seleccione **Ejecutar consulta**.

1. Observe los resultados y las estadísticas de la consulta. El cargo por unidad de solicitud ha aumentado debido a la cláusula **ORDER BY**.

## Creación de un índice compuesto en la directiva de indexación

Ahora, tendrá que crear un índice compuesto si ordena los elementos mediante varias propiedades. En esta tarea, creará un índice compuesto para ordenar los elementos por su categoryName y, a continuación, su nombre real.

1. En **Data Explorer**, expanda el nodo de base de datos **cosmicworks**, seleccione el nodo de contenedor **products** y, a continuación, seleccione **Nueva consulta SQL**.

1. Elimine el contenido del área del editor.

1. Cree una nueva consulta SQL que ordenará primero los resultados por **categoryName** en orden descendente y, después, por **price** en orden ascendente:

    ```
    SELECT 
        p.name,
        p.categoryName,
        p.price
    FROM
        products p
    ORDER BY
        p.categoryName DESC,
        p.price ASC
    ```

1. Seleccione **Ejecutar consulta**.

1. La consulta debe producir un error **La consulta order by no tiene un índice compuesto correspondiente desde el que se pueda servir**.

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
          "path": "/*"
        }
      ],
      "excludedPaths": [],
      "compositeIndexes": [
        [
          {
            "path": "/categoryName",
            "order": "descending"
          },
          {
            "path": "/price",
            "order": "ascending"
          }
        ]
      ]
    }
    ```

1. En **Data Explorer**, expanda el nodo de base de datos **cosmicworks**, seleccione el nodo de contenedor **products** y, a continuación, seleccione **Nueva consulta SQL**.

1. Elimine el contenido del área del editor.

1. Cree una nueva consulta SQL que ordenará primero los resultados por **categoryName** en orden descendente y, después, por **price** en orden ascendente:

    ```
    SELECT 
        p.name,
        p.categoryName,
        p.price
    FROM
        products p
    ORDER BY
        p.categoryName DESC,
        p.price ASC
    ```

1. Seleccione **Ejecutar consulta**.

1. Observe los resultados y las estadísticas de la consulta. Esta vez, ya que se completó la consulta, puede volver a revisar el cargo de RU.

1. Elimine el contenido del área del editor.

1. Cree una nueva consulta SQL que ordenará los resultados por **categoryName** en orden descendente, después por **nombre** en orden ascendente y, por último, por **price** en orden ascendente:

    ```
    SELECT 
        p.name,
        p.categoryName,
        p.price
    FROM
        products p
    ORDER BY
        p.categoryName DESC,
        p.name ASC,
        p.price ASC
    ```

1. Seleccione **Ejecutar consulta**.

1. La consulta debe producir un error **La consulta order by no tiene un índice compuesto correspondiente desde el que se pueda servir**.

1. En **Data Explorer**, expanda el nodo de base de datos **cosmicworks**, expanda el nodo de contenedor **products** y, a continuación, seleccione **Configuración** de nuevo.

1. En la pestaña **Configuración**, vaya a la sección **Directiva de indexación**.

1. Reemplace la directiva de indexación por este objeto JSON modificado y, a continuación, **guarde** los cambios:

    ```
    {
      "indexingMode": "consistent",
      "automatic": true,
      "includedPaths": [
        {
          "path": "/*"
        }
      ],
      "excludedPaths": [],
      "compositeIndexes": [
        [
          {
            "path": "/categoryName",
            "order": "descending"
          },
          {
            "path": "/price",
            "order": "ascending"
          }
        ],
        [
          {
            "path": "/categoryName",
            "order": "descending"
          },
          {
            "path": "/name",
            "order": "ascending"
          },
          {
            "path": "/price",
            "order": "ascending"
          }
        ]
      ]
    }
    ```

1. En **Data Explorer**, expanda el nodo de base de datos **cosmicworks**, seleccione el nodo de contenedor **products** y, a continuación, seleccione **Nueva consulta SQL**.

1. Elimine el contenido del área del editor.

1. Cree una nueva consulta SQL que ordenará los resultados por **categoryName** en orden descendente, después por **nombre** en orden ascendente y, por último, por **price** en orden ascendente:

    ```
    SELECT 
        p.name,
        p.categoryName,
        p.price
    FROM
        products p
    ORDER BY
        p.categoryName DESC,
        p.name ASC,
        p.price ASC
    ```

1. Seleccione **Ejecutar consulta**.

1. Observe los resultados y las estadísticas de la consulta. Esta vez, ya que se completó la consulta, puede volver a revisar el cargo de RU.

1. Cierre la ventana o pestaña del explorador web.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
