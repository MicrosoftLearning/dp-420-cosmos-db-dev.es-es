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
    | **Limitar la cantidad total de rendimiento que se puede aprovisionar en esta cuenta** | *Desactivado* |

    > &#128221; Es posible que los entornos de laboratorio tengan restricciones que le impidan crear un nuevo grupo de recursos. Si es así, usa el grupo de recursos existente creado previamente.

1. Espera a que se complete la tarea de implementación antes de continuar con esta tarea.

1. Ve al recurso de cuenta de **Azure Cosmos DB** recién creado y, después, ve al panel **Claves**.

1. Este panel contiene los detalles de conexión y las credenciales necesarias para conectarte a la cuenta desde el SDK. Específicamente:

    1. Observe el campo **CADENA DE CONEXIÓN PRINCIPAL**. Usará este valor de **cadena de conexión** más adelante en este ejercicio.

1. Mantenga abierta la pestaña del explorador, ya que volveremos a ella más adelante.

1. Inicie **Visual Studio Code**.

    > &#128221; Si aún no está familiarizado con la interfaz de Visual Studio Code, revise la [Guía de introducción para Visual Studio Code][code.visualstudio.com/docs/getstarted]

1. En **Visual Studio Code**, abra el menú **Terminal** y, a continuación, seleccione **Nuevo terminal** para abrir una nueva instancia de terminal.

1. Instale la herramienta de línea de comandos [cosmicworks][nuget.org/packages/cosmicworks] para su uso global en la máquina.

    ```powershell
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

1. Vuelva al explorador web, abra una nueva pestaña y vaya a Azure Portal (``portal.azure.com``).

1. Seleccione **Grupos de recursos**, seleccione el grupo de recursos que creó o visualizó anteriormente en este laboratorio y, a continuación, seleccione el recurso de **cuenta de Azure Cosmos DB** que creó en este laboratorio.

1. En el recurso de cuenta de **Azure Cosmos DB**, vaya al panel **Data Explorer**.

1. En **Data Explorer**, expanda el nodo de base de datos **cosmicworks**, expanda el nodo de contenedor **products** y, a continuación, seleccione **Elementos**.

1. Observe y seleccione los distintos elementos JSON en el contenedor **products**. Estos son los elementos creados por la herramienta de línea de comandos que se usa en los pasos anteriores.

1. Selecciona el nodo **Scale**. En la pestaña **Scale**, selecciona **Manual**, actualiza el valor de **rendimiento requerido** de **4000 RU/s** a **400 RU/s** y, a continuación, **Guarda** los cambios**.

1. En el panel **Data Explorer**, seleccione **Nuevo contenedor**.

1. En la ventana emergente **Nuevo contenedor**, escriba los valores siguientes para cada configuración y, después, seleccione **Aceptar**:

    | **Configuración** | **Valor** |
    | --: | :-- |
    | **Id. de base de datos** | *Usar existente* &vert; *cosmicworks* |
    | **Id. de contenedor** | *`flatproducts`* |
    | **Clave de partición** | *`/category`* |

1. De nuevo en el panel **Data Explorer**, expanda el nodo de base de datos **cosmicworks** y observe el nodo contenedor **flatproducts** dentro de la jerarquía.

1. Vuelva al **Inicio** de Azure Portal.

## Creación de un recurso de Azure Data Factory

Ahora que los recursos de Azure Cosmos DB for NoSQL están en vigor, creará un recurso de Azure Data Factory y configurará todos los componentes y conexiones necesarios para realizar un movimiento de datos único desde un contenedor de API NoSQL a otro para extraer datos, transformarlos y cargarlos en otro contenedor de API NoSQL.

1. Seleccione **+ Crear un recurso**, busque *Data Factory* y, a continuación, cree un nuevo recurso de cuenta de **Data Factory** con la siguiente configuración, dejando todas las opciones restantes en sus valores predeterminados:

    | **Configuración** | **Valor** |
    | ---: | :--- |
    | **Suscripción** | *Tu suscripción a Azure existente* |
    | **Grupo de recursos** | *Seleccione un grupo de recursos ya existente o cree uno nuevo* |
    | **Nombre** | *Escriba un nombre único global*. |
    | **Región** | *seleccione cualquier región disponible* |
    | **Versión** | *V2* |

    > &#128221; Es posible que los entornos de laboratorio tengan restricciones que te impidan crear un nuevo grupo de recursos. Si es así, usa el grupo de recursos existente creado previamente.

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
        p.category.name as category, 
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
