---
lab:
  title: "Optimización de la directiva de indexación de un contenedor de Azure Cosmos\_DB for NoSQL para una consulta específica"
  module: Module 10 - Optimize query and operation performance in Azure Cosmos DB for NoSQL
---

# Optimización de la directiva de indexación de un contenedor de Azure Cosmos DB for NoSQL para una consulta

Al planear una cuenta de Azure Cosmos DB for NoSQL, conocer nuestras consultas más populares puede ayudarnos a optimizar la directiva de indexación para que las consultas sean lo más eficaces posible.

En este laboratorio, usaremos el Explorador de datos para probar consultas SQL con la directiva de indexación predeterminada y una directiva de indexación que incluya un índice compuesto.

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

1. Espere a que se complete la tarea de implementación antes de continuar con esta tarea.

1. Vaya al recurso de cuenta de **Azure Cosmos DB** recién creado y vaya al panel del **Explorador de datos**.

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

1. De nuevo en el panel **Data Explorer**, expanda el nodo de base de datos **cosmicworks** y observe el nodo contenedor **products** dentro de la jerarquía.

1. En la hoja de recursos, vaya al panel **Claves**.

1. Este panel contiene los detalles de conexión y las credenciales necesarias para conectarse a la cuenta desde el SDK. Específicamente:

    1. Observe el campo **CADENA DE CONEXIÓN PRINCIPAL**. Usará este valor de **cadena de conexión** más adelante en este ejercicio.

1. Abra **Visual Studio Code**.

## Inicialización de la cuenta de Azure Cosmos DB for NoSQL con datos de ejemplo

Usará una utilidad de línea de comandos que crea una base de datos de **cosmicworks** y un contenedor de **productos**. A continuación, la herramienta creará un conjunto de elementos que observará mediante el procesador de fuente de cambios que se ejecuta en la ventana del terminal.

1. En **Visual Studio Code**, abra el menú **Terminal** y, a continuación, seleccione **Nuevo terminal** para abrir un nuevo terminal.

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
        p.category,
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
        p.category,
        p.price
    FROM
        products p
    ORDER BY
        p.category DESC
    ```

1. Seleccione **Ejecutar consulta**.

1. Observe los resultados y las estadísticas de la consulta. El cargo por unidad de solicitud ha aumentado debido a la cláusula **ORDER BY** .

## Creación de un índice compuesto en la directiva de indexación

Ahora, tendrá que crear un índice compuesto si ordena los elementos mediante varias propiedades. En esta tarea, creará un índice compuesto para ordenar los elementos por su categoryName y, a continuación, su nombre real.

1. En **Data Explorer**, expanda el nodo de base de datos **cosmicworks**, seleccione el nodo de contenedor **products** y, a continuación, seleccione **Nueva consulta SQL**.

1. Elimine el contenido del área del editor.

1. Crea una nueva consulta SQL que ordenará primero los resultados por la **categoría** en orden descendente y, después, por el **precio** en orden ascendente:

    ```
    SELECT 
        p.name,
        p.category,
        p.price
    FROM
        products p
    ORDER BY
        p.category DESC,
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
            "path": "/category",
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

1. Cree una nueva consulta SQL que ordenará primero los resultados por el **categoryName** en orden descendente y, después, por el **precio** en orden ascendente:

    ```
    SELECT 
        p.name,
        p.category,
        p.price
    FROM
        products p
    ORDER BY
        p.category DESC,
        p.price ASC
    ```

1. Seleccione **Ejecutar consulta**.

1. Observe los resultados y las estadísticas de la consulta. Esta vez, ya que se completó la consulta, puede volver a revisar el cargo de RU.

1. Elimine el contenido del área del editor.

1. Crea una nueva consulta SQL que ordenará primero los resultados por la **categoría** en orden descendente, después por **nombre** en orden ascendente y, por último, por el **precio** en orden ascendente:

    ```
    SELECT 
        p.name,
        p.category,
        p.price
    FROM
        products p
    ORDER BY
        p.category DESC,
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
            "path": "/category",
            "order": "descending"
          },
          {
            "path": "/price",
            "order": "ascending"
          }
        ],
        [
          {
            "path": "/category",
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

1. Crea una nueva consulta SQL que ordenará primero los resultados por la **categoría** en orden descendente, después por **nombre** en orden ascendente y, por último, por el **precio** en orden ascendente:

    ```
    SELECT 
        p.name,
        p.category,
        p.price
    FROM
        products p
    ORDER BY
        p.category DESC,
        p.name ASC,
        p.price ASC
    ```

1. Seleccione **Ejecutar consulta**.

1. Observe los resultados y las estadísticas de la consulta. Esta vez, ya que se completó la consulta, puede volver a revisar el cargo de RU.

1. Cierre la ventana o pestaña del explorador web.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
