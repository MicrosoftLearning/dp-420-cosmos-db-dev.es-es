---
lab:
  title: "Búsqueda de datos mediante Búsqueda de Azure\_AI y Azure\_Cosmos\_DB for NoSQL"
  module: Module 7 - Integrate Azure Cosmos DB for NoSQL with Azure services
---

# Búsqueda de datos mediante Búsqueda de Azure AI y Azure Cosmos DB for NoSQL

Búsqueda de Azure AI combina un motor de búsqueda como servicio con una integración completa con funcionalidades de inteligencia artificial para enriquecer la información en el índice de búsqueda.

En este laboratorio, creará un índice de Búsqueda de Azure AI que indexa datos automáticamente en un contenedor de Azure Cosmos DB for NoSQL y enriquece los datos mediante la funcionalidad Translator de Azure Cognitive Services.

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

1. De regreso en el panel **Data Explorer**, expanda el nodo de base de datos **cosmicworks** y observe el nodo contenedor **products** dentro de la jerarquía.

## Inicialización de la cuenta de Azure Cosmos DB for NoSQL con datos de ejemplo

Usará una utilidad de línea de comandos que crea una base de datos de **cosmicworks** y un contenedor de **productos**.

1. En **Visual Studio Code**, abra el menú **Terminal** y seleccione **Nuevo terminal**.

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

1. Cierre **Visual Studio Code**.

## Creación de un recurso de Búsqueda de Azure AI

Antes de continuar con este ejercicio, primero debe crear una nueva instancia de Búsqueda de Azure AI.

1. Vaya a Azure Portal (``portal.azure.com``) desde una nueva ventana o pestaña del explorador web.

1. Inicie sesión en el portal con las credenciales de Microsoft asociadas a su suscripción.

1. Seleccione **+ Crear un recurso**, busque *Búsqueda de AI*, y a continuación, cree un nuevo recurso de cuenta **Búsqueda de Azure AI** con la siguiente configuración, dejando todas las opciones restantes en sus valores predeterminados:

    | **Configuración** | **Valor** |
    | ---: | :--- |
    | **Suscripción** | *Tu suscripción a Azure existente* |
    | **Grupo de recursos** | *Seleccione un grupo de recursos ya existente o cree uno nuevo* |
    | **Nombre** | *Escriba un nombre único global*. |
    | **Ubicación** | *seleccione cualquier región disponible* |

    > &#128221; Es posible que los entornos de laboratorio tengan restricciones que le impidan crear un nuevo grupo de recursos. Si es así, usa el grupo de recursos existente creado previamente.

1. Espere a que se complete la tarea de implementación antes de continuar con esta tarea.

1. Vaya al recurso de cuenta **Búsqueda de Azure AI** recién creado.

## Creación de un indexador y un índice para datos de Azure Cosmos DB for NoSQL

Creará un indexador que indexa un subconjunto de datos en un contenedor específico de Azure Cosmos DB for NoSQL cada hora.

1. En la hoja de recursos **Búsqueda de AI**, seleccione **Importar datos**.

1. En el paso de **conexión a los datos** del asistente para la **importación de datos**, en la lista **Origen de datos**, seleccione **Azure Cosmos DB**.

1. Configure el origen de datos con la siguiente configuración, dejando todas las opciones restantes en sus valores predeterminados:

    | **Configuración** | **Valor** |
    | ---: | :--- |
    | **Nombre del origen de datos** | *``products-cosmossql-source``* |
    | **Cadena de conexión** | ***connection string** de la cuenta de Azure Cosmos DB for NoSQL creada anteriormente* |
    | **Base de datos** | *cosmicworks* |
    | **Colección** | *products* |

1. En el campo de **consulta**, escriba la consulta SQL siguiente para crear una vista materializada de un subconjunto de los datos en el contenedor:

    ```sql
    SELECT 
        p.id, 
        p.category, 
        p.name, 
        p.price,
        p._ts
    FROM 
        products p 
    WHERE 
        p._ts > @HighWaterMark 
    ORDER BY 
        p._ts
    ```

1. Active la casilla **Resultados de consulta ordenados por _ts**.

    > &#128221; Esta casilla permite que Búsqueda de Azure AI sepa que la consulta ordena los resultados por el campo **_ts**. Este tipo de ordenación permite un seguimiento del progreso incremental. Si se produce un error en el indexador, puede recuperar el mismo valor **_ts**, ya que los resultados se ordenan por marca de tiempo.

1. Seleccione **Siguiente: Agregar conocimientos cognitivos**.

1. Seleccione **Ir a: Personalizar índice de destino**.

1. En el paso de **personalización del índice de destino** del asistente, configure el índice con la siguiente configuración, dejando todas las opciones restantes en sus valores predeterminados:

    | **Configuración** | **Valor** |
    | ---: | :--- |
    | **Nombre del índice** | *``products-index``* |
    | **Clave** | *id* |

1. En la tabla de campos, configure las opciones **Recuperable**, **Filtrable**, **Ordenable**, **Clasificable** y **Buscable** para cada campo mediante la siguiente tabla:

    | **Campo** | **Retrievable** | **Filtrable** | **Ordenable** | **Clasificable** | **Se puede buscar** |
    | ---: | :---: | :---: | :---: | :---: | :---: |
    | **id** | &#10004; | &#10004; | &#10004; | | |
    | **category** | &#10004; | &#10004; | &#10004; | &#10004; | |
    | **name** | &#10004; | &#10004; | &#10004; | | &#10004; (Inglés - Microsoft) |
    | **price** | &#10004; | &#10004; | &#10004; | &#10004; | |

1. Seleccione **Siguiente: Crear indizador**.

1. En el paso de **creación de un indexador** del asistente, configure el índice con la siguiente configuración, dejando todas las opciones restantes en sus valores predeterminados:

    | **Configuración** | **Valor** |
    | ---: | :--- |
    | **Nombre** | *``products-cosmosdb-indexer``* |
    | **Programación** | *Cada hora* |

1. Seleccione **Enviar** para crear un origen de datos, un índice y un indexador.

    > &#128221; Es posible que deba descartar una encuesta emergente tras la creación del primer indexador.

1. En la hoja de recursos **Búsqueda de AI**, vaya a la pestaña **Indexadores** para observar el resultado de la primera operación de indexación.

1. Espere a que el indexador **products-cosmosdb-indexer** tenga el estado **Correcto** antes de continuar con esta tarea.

    > &#128221; Es posible que tenga que usar la opción **Actualizar** para actualizar la hoja si no se actualiza automáticamente.

1. Vaya a la pestaña **Índices** y, a continuación, seleccione el índice **products-index**.

## Validación del índice con consultas de búsqueda de ejemplo

Ahora que la vista materializada de los datos de Azure Cosmos DB for NoSQL está en el índice de búsqueda, puede realizar algunas consultas básicas que aprovechen las características de Búsqueda de Azure AI.

> &#128221; Este laboratorio no está diseñado para enseñar la sintaxis de Búsqueda de Azure AI. Estas consultas se han mantenido para presentar algunas de las características disponibles en el índice y el motor de búsqueda.

1. En la pestaña **Explorador de búsqueda**, seleccione la lista desplegable **Vista** y, a continuación, seleccione la **vista JSON**.

1. Observe que en el **editor de consultas JSON**, la sintaxis de la consulta de búsqueda JSON predeterminada que devuelve todos los resultados posibles mediante un operador **\*** (comodín).

   ```json
    {
      "search": "*",
      "count": true
    }
   ```

1. Seleccione el botón **Buscar** para realizar la búsqueda.

1. Observa que esta consulta de búsqueda devuelve todos los resultados posibles, pero también incluye un campo de metadatos que indica el recuento total de resultados incluso si no se incluyen todos en la misma página.

1. En el **editor de consultas JSON**, escriba la siguiente consulta y, luego, seleccione **Buscar**:

    ```json
    {
        "search": "touring 3000"
    }
    ```

1. Observe que esta consulta de búsqueda devuelve resultados que contienen los términos **touring** o **3000**, lo que proporciona una puntuación más alta a los resultados que contienen ambos términos. Los resultados se clasifican entonces en orden descendente por campo **@search.score**.

1. En el **editor de consultas JSON**, escriba la siguiente consulta y, luego, seleccione **Buscar**:

    ```json
    {
        "search": "blue"
        , "count": true
        , "top": 6
    }
    ```

1. Observe que esta consulta de búsqueda solo devuelve un conjunto de seis resultados a la vez, aunque haya más coincidencias en el lado servidor.

1. En el **editor de consultas JSON**, escriba la siguiente consulta y, luego, seleccione **Buscar**:

    ```json
    {
        "search": "mountain"
        , "count": true
        , "top": 25
        , "skip": 50
    }
    ```

1. Observe que esta consulta de búsqueda omite los primeros 50 resultados y devuelve un conjunto de 25 resultados. Si se trata de una vista paginada en una aplicación del lado cliente, podría deducir que sería la tercera "página" de resultados.

1. En el **editor de consultas JSON**, escriba la siguiente consulta y, luego, seleccione **Buscar**:

    ```json
    {
        "search": "touring"
        , "count": true
        , "filter": "price lt 500"
    }
    ```

1. Observe que esta consulta de búsqueda solo devuelve resultados en los que el valor del campo de precio numérico es inferior a 500.

1. En el **editor de consultas JSON**, escriba la siguiente consulta y, luego, seleccione **Buscar**:

    ```json
    {
        "search": "road"
        , "count": true
        , "top": 15
        , "facets": ["price,interval:500"]
    }
    ```

1. Observe que esta consulta de búsqueda devuelve una colección de datos de facetas que indica cuántos elementos pertenecen a cada categoría incluso si no están todos presentes en la página actual de resultados. En este ejemplo, los elementos coincidentes se dividen en categorías de precios numéricas en intervalos de 500. Normalmente se usa para rellenar filtros y ayudas a la navegación en aplicaciones del lado cliente.

1. Cierre la ventana o pestaña del explorador web.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
