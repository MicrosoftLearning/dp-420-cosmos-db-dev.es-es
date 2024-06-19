---
lab:
  title: "Migración de datos con Azure\_Data Factory"
  module: Module 2 - Plan and implement Azure Cosmos DB for NoSQL
---

# Migración de datos con Azure Data Factory

En Azure Data Factory, Azure Cosmos DB se admite como origen de ingesta de datos y como destino (receptor) de salida de datos.

En este laboratorio, poblaremos Azure Cosmos DB mediante una utilidad de línea de comandos útil y, a continuación, usaremos Azure Data Factory para trasladar un subconjunto de datos de un contenedor a otro.

## Creación y propagación de la cuenta de Azure Cosmos DB for NoSQL

Usará una utilidad de línea de comandos que crea una base de datos **cosmicworks** y un contenedor **products** a **4000** unidades de solicitud por segundo (RU/s). Una vez creados, ajustará el rendimiento a 400 RU/s.

Para acompañar al contenedor products, creará manualmente un contenedor **flatproducts** que será el destino de la transformación ETL y la operación de carga al final de este laboratorio.

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
    | **Limitar la cantidad total de rendimiento que se puede aprovisionar en esta cuenta** | *Desactivado* |

    > &#128221; Es posible que los entornos de laboratorio tengan restricciones que le impidan crear un nuevo grupo de recursos. Si es así, use el grupo de recursos existente creado previamente.

1. Espere a que se complete la tarea de implementación antes de continuar con esta tarea.

1. Vaya al recurso de cuenta de **Azure Cosmos DB** recién creado y vaya al panel **Claves**.

1. Este panel contiene los detalles de conexión y las credenciales necesarias para conectarse a la cuenta desde el SDK. Específicamente:

    1. Observe el campo **URI**. Usará este valor de **endpoint** más adelante en este ejercicio.

    1. Observe el campo **PRIMARY KEY**. Usará este valor de **clave** más adelante en este ejercicio.

1. Mantenga abierta la pestaña del explorador, ya que volveremos a ella más adelante.

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
    | **--endpoint** | *El valor del punto de conexión que ha comprobado anteriormente en este laboratorio* |
    | **--key** | *El valor de la clave que ha comprobado anteriormente en este laboratorio* |
    | **--datasets** | *product* |

    ```
    cosmicworks --endpoint <cosmos-endpoint> --key <cosmos-key> --datasets product
    ```

    > &#128221; Por ejemplo, si el punto de conexión es **https&shy;://dp420.documents.azure.com:443/** y la clave es **fDR2ci9QgkdkvERTQ==**, el comando sería: ``cosmicworks --endpoint https://dp420.documents.azure.com:443/ --key fDR2ci9QgkdkvERTQ== --datasets product``

1. Espere a que el comando **cosmicworks** termine de poblar la cuenta con una base de datos, un contenedor y elementos.

1. Cierre el terminal integrado.

1. Vuelva al explorador web, abra una nueva pestaña y vaya a Azure Portal (``portal.azure.com``).

1. Seleccione **Grupos de recursos**, seleccione el grupo de recursos que creó o visualizó anteriormente en este laboratorio y, a continuación, seleccione el recurso de **cuenta de Azure Cosmos DB** que creó en este laboratorio.

1. En el recurso de cuenta de **Azure Cosmos DB**, vaya al panel **Data Explorer**.

1. En **Data Explorer**, expanda el nodo de base de datos **cosmicworks**, expanda el nodo de contenedor **products** y, a continuación, seleccione **Elementos**.

1. Observe y seleccione los distintos elementos JSON en el contenedor **products**. Estos son los elementos creados por la herramienta de línea de comandos que se usa en los pasos anteriores.

1. Seleccione el nodo **Escala y configuración**. En la pestaña **Escala y configuración**, seleccione **Manual**, actualice el valor de **rendimiento requerido** de **4000 RU/s** a **400 RU/s** y, a continuación, **Guarde** los cambios**.

1. En el panel **Data Explorer**, seleccione **Nuevo contenedor**.

1. En la ventana emergente **Nuevo contenedor**, escriba los valores siguientes para cada configuración y, después, seleccione **Aceptar**:

    | **Configuración** | **Valor** |
    | --: | :-- |
    | **Id. de base de datos** | *Usar existente* &vert; *cosmicworks* |
    | **Id. de contenedor** | *`flatproducts`* |
    | **Clave de partición** | *`/category`* |
    | **Rendimiento del contenedor (escalado automático)** | *Manual* |
    | **RU/s** | *`400`* |

1. De nuevo en el panel **Data Explorer**, expanda el nodo de base de datos **cosmicworks** y observe el nodo contenedor **flatproducts** dentro de la jerarquía.

1. Vuelva al **Inicio** de Azure Portal.

## Creación de un recurso de Azure Data Factory

Ahora que los recursos de Azure Cosmos DB for NoSQL están en vigor, creará un recurso de Azure Data Factory y configurará todos los componentes y conexiones necesarios para realizar un movimiento de datos único desde un contenedor de API NoSQL a otro para extraer datos, transformarlos y cargarlos en otro contenedor de API NoSQL.

1. Seleccione **+ Crear un recurso**, busque *Data Factory* y, a continuación, cree un nuevo recurso de cuenta de **Data Factory** con la siguiente configuración, dejando todas las opciones restantes en sus valores predeterminados:

    | **Configuración** | **Valor** |
    | ---: | :--- |
    | **Suscripción** | *Su suscripción de Azure existente* |
    | **Grupo de recursos** | *Seleccione un grupo de recursos ya existente o cree uno nuevo* |
    | **Nombre** | *Escriba un nombre único global*. |
    | **Región** | *seleccione cualquier región disponible* |
    | **Versión** | *V2* |
    | **Configuración de Git** | *Configurar Git más tarde* |

    > &#128221; Es posible que los entornos de laboratorio tengan restricciones que le impidan crear un nuevo grupo de recursos. Si es así, use el grupo de recursos existente creado previamente.

1. Espere a que se complete la tarea de implementación antes de continuar con esta tarea.

1. Vaya al recurso de **Data Factory** recién creado y seleccione **Iniciar Studio**.

    > &#128161; Como alternativa, puede ir a (``adf.azure.com/home``), seleccionar el recurso de Data Factory recién creado y, a continuación, seleccionar el icono de inicio.

1. Desde la pantalla principal. Seleccione la opción **Ingerir** para iniciar el asistente rápido para realizar una operación de copia única y pasar al paso **Propiedades** del asistente.

1. A partir del paso **Propiedades** del asistente, en la sección **Tipo de tarea**, seleccione **Tarea de copia integrada**.

1. En la sección **Cadencia de tareas o programación de tareas**, seleccione **Ejecutar una vez** y, a continuación, seleccione **Siguiente** para ir al paso **Origen** del asistente.

1. En el paso **Origen** del asistente, en la lista **Tipo de origen**, seleccione **Azure Cosmos DB for NoSQL**.

1. En la sección **Conexión**, seleccione **+ Nueva conexión**.

1. En la ventana emergente **Nueva conexión (Azure Cosmos DB for NoSQL** ), configure la nueva conexión con los valores siguientes y, a continuación, seleccione **Crear**:

    | **Configuración** | **Valor** |
    | ---: | :--- |
    | **Nombre** | *`CosmosSqlConn`* |
    | **Conectar mediante entorno de ejecución de integración** | *AutoResolveIntegrationRuntime* |
    | **Método de autenticación** | *Clave de cuenta* &vert; *Cadena de conexión* |
    | **Método de selección de cuenta** | *Desde la suscripción de Azure* |
    | **Suscripción de Azure** | *Su suscripción de Azure existente* |
    | **Azure Cosmos DB account name** (Nombre de la cuenta de Azure Cosmos DB) | *El nombre de la cuenta de Azure Cosmos DB existente que eligió anteriormente en este laboratorio* |
    | **Nombre de la base de datos** | *cosmicworks* |

1. De nuevo en la sección **Almacén de datos de origen**, en la sección **Tablas de origen**, seleccione **Usar consulta**.

1. En la lista **Nombre de tabla**, seleccione **products**.

1. En el editor de **Consulta**, elimine el contenido existente y escriba la siguiente consulta:

    ```
    SELECT 
        p.name, 
        p.categoryName as category, 
        p.price 
    FROM 
        products p
    ```

1. Seleccione **Vista previa de los datos** para probar la validez de la consulta. Seleccione **Siguiente** para ir al paso **Destino** del asistente.

1. En el paso **Destino** del asistente, en la lista **Tipo de destino**, seleccione **Azure Cosmos DB for NoSQL**.

1. En la lista **Conexión**, seleccione **CosmosSqlConn**.

1. En la lista **Destino**, seleccione **flatproducts** y, a continuación, seleccione **Siguiente** para ir al paso **Configuración** del asistente.

1. En el paso **Configuración** del asistente, en el campo **Nombre de tarea**, escriba **`FlattenAndMoveData`**.

1. Deje todos los campos restantes con sus valores predeterminados en blanco y, a continuación, seleccione **Siguiente** para pasar al último paso del asistente.

1. Revise el **Resumen** de los pasos que ha seleccionado en el asistente y, a continuación, seleccione **Siguiente**.

1. Observe los distintos pasos de la implementación. Una vez finalizada la implementación, seleccione **Finalizar**.

1. Vuelva a la pestaña del explorador que tiene la **cuenta de Azure Cosmos DB** y vaya al panel **Data Explorer**.

1. En **Data Explorer**, expanda el nodo de base de datos **cosmicworks**, seleccione el nodo de contenedor **flatproducts** y, a continuación, seleccione **Nueva consulta SQL**.

1. Elimine el contenido del área del editor.

1. Cree una nueva consulta SQL que devuelva todos los documentos en los que el **nombre** equivalga a **HL Headset**:

    ```
    SELECT 
        p.name, 
        p.category, 
        p.price 
    FROM
        flatproducts p
    WHERE
        p.name = 'HL Headset'
    ```

1. Seleccione **Ejecutar consulta**.

1. Observe los resultados de la consulta.

1. Cierre la ventana o pestaña del explorador web.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[nuget.org/packages/cosmicworks]: https://www.nuget.org/packages/cosmicworks
