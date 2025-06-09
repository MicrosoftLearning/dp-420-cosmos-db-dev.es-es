---
lab:
  title: Creación de una cuenta de Azure Cosmos DB for NoSQL
  module: Module 1 - Get started with Azure Cosmos DB for NoSQL
---

# Creación de una cuenta de Azure Cosmos DB for NoSQL

Antes de profundizar demasiado en Azure Cosmos DB, es importante controlar los aspectos básicos de la creación de los recursos que usará más. En la mayoría de los escenarios, tendrá que crear con facilidad cuentas, bases de datos, contenedores y elementos. En un escenario real, también debe tener algunas consultas básicas "a mano" para probar que ha creado todos los recursos correctamente.

En este laboratorio, creará una nueva cuenta de Azure Cosmos DB mediante NoSQL. Después, usará el Explorador de datos para crear una base de datos, un contenedor y dos elementos. Por último, consultará la base de datos de los elementos que creó.

## Creación de una cuenta de Azure Cosmos DB

Azure Cosmos DB es un servicio de base de datos NoSQL basado en la nube que admite varias API. Al aprovisionar una cuenta de Azure Cosmos DB por primera vez, seleccionará cuál de las API quiere que admita la cuenta (por ejemplo, **API de Mongo** o **API de NoSQL**).

1. Vaya a Azure Portal (``portal.azure.com``) desde una nueva ventana o pestaña del explorador web.

1. Inicie sesión en el portal con las credenciales de Microsoft asociadas a su suscripción.

1. En la categoría **Servicios de Azure**, seleccione **Crear un recurso** y, a continuación, seleccione **Azure Cosmos DB**.

    > &#128161; Como alternativa; expanda el menú **&#8801;**, seleccione **Todos los servicios**, en la categoría **Bases de datos**, seleccione **Azure Cosmos DB** y después **Crear**.

1. En el panel de **opción Seleccionar API**, seleccione la opción **Crear** en la sección **Azure Cosmos DB for NoSQL**.

1. En el panel **Crear cuenta de Azure Cosmos DB**, observe la pestaña **Aspectos básicos**.

1. En la pestaña **Aspectos básicos**, escriba los valores siguientes para cada opción:

    | **Configuración** | **Valor** |
    | --: | :-- |
    | **Tipo de carga de trabajo** | **Aprendizaje** |
    | **Suscripción** | **Use la suscripción de Azure existente.** *Todos los recursos deben pertenecer a un grupo de recursos. Cada grupo de recursos debe pertenecer a una suscripción.* |
    | **Grupo de recursos** | **Use un grupo de recursos existente o cree uno nuevo.** *Todos los recursos deben pertenecer a un grupo de recursos.* |
    | **Account Name** | **Escriba un nombre único global.** *Nombre de cuenta único global. Este nombre se usará como parte de la dirección DNS para las solicitudes.  El portal comprobará el nombre en tiempo real.* |
    | **Ubicación** | **Seleccione cualquier región disponible.** *Seleccione la región geográfica en la que se hospedará inicialmente la base de datos.* |
    | **Capacity mode (Modo de capacidad)** | **Rendimiento aprovisionado** |
    | **Aplicación de descuento por nivel Gratis** | **No aplicar** |

1. Seleccione **Revisar y crear** para ir a la pestaña **Revisar y crear** y después seleccione **Crear**.

    > &#128221; La cuenta de Azure Cosmos DB for NoSQL puede tardar entre 10 y 15 minutos en estar lista para su uso.

1. Observe el panel **Implementación**. Una vez que se complete la implementación, el panel se actualizará con un mensaje **Implementación correcta**.

1. Aún dentro del panel **Implementación**, seleccione **Ir a recursos**.

## Se usará el Explorador de datos para crear una base de datos y un contenedor

El Explorador de datos será la herramienta principal para administrar la base de datos y los contenedores de Azure Cosmos DB for NoSQL en Azure Portal. Creará una base de datos básica y un contenedor para usarlos en este laboratorio.

1. En el panel **cuenta de Azure Cosmos DB**, seleccione **Data Explorer** en el menú de recursos.

1. En el panel **Data Explorer**, seleccione **Nuevo contenedor**.

1. En la ventana emergente **Nuevo contenedor**, escriba los valores siguientes para cada configuración y, después, seleccione **Aceptar**:

    | **Configuración** | **Valor** |
    | --: | :-- |
    | **Id. de base de datos** | *cosmicworks* |
    | **Uso compartido del rendimiento entre contenedores** | *Unckecked* |
    | **Id. de contenedor** | *products* |
    | **Clave de partición** | */categoryId* |
    | **Rendimiento del contenedor (escalado automático)** | *Manual* |
    | **RU/s** | *400* |

1. De regreso en el panel **Data Explorer**, expanda el nodo de base de datos **cosmicworks** y observe el nodo contenedor **products** dentro de la jerarquía.

## Uso del Explorador de datos para crear nuevos elementos

El Explorador de datos también incluye un conjunto de características para consultar, crear y administrar elementos en un contenedor de Azure Cosmos DB for NoSQL. Creará dos elementos básicos mediante JSON sin formato en el Explorador de datos.

1. En **Data Explorer**, expanda el nodo de base de datos **cosmicworks**, expanda el nodo de contenedor **products** y, a continuación, seleccione **Elementos**.

1. Seleccione **Nuevo elemento** en la barra de comandos y, en el editor, reemplace el elemento JSON de marcador de posición por el siguiente contenido:

    ```
    {
      "categoryId": "4F34E180-384D-42FC-AC10-FEC30227577F",
      "categoryName": "Components, Pedals",
      "sku": "PD-R563",
      "name": "ML Road Pedal",
      "price": 62.09
    }
    ```

1. Selecciona **Guardar** en la barra de comandos para agregar el primer elemento JSON.

1. De nuevo en la pestaña **Elementos**, seleccione **Nuevo elemento** en la barra de comandos. En el editor, reemplace el elemento JSON de marcador de posición por el siguiente contenido:

    ```
    {
      "categoryId": "75BF1ACB-168D-469C-9AA3-1FD26BB4EA4C",
      "categoryName": "Bikes, Touring Bikes",
      "sku": "BK-T18Y-44",
      "name": "Touring-3000 Yellow, 44",
      "price": 742.35
    }
    ```

1. Selecciona **Guardar** en la barra de comandos para agregar el segundo elemento JSON.

1. En la pestaña **Elementos**, observe los dos nuevos elementos del panel **Elementos**.

## Uso del Explorador de datos para emitir una consulta básica

Por último, el Explorador de datos tiene un editor de consultas integrado que se usa para emitir consultas, observar los resultados y medir el impacto en términos de unidades de solicitud por segundo (RU/s).

1. En el panel **Data Explorer**, seleccione **Nueva consulta SQL**.

1. En la pestaña consulta, seleccione **Ejecutar consulta** para ver una consulta estándar que seleccione todos los elementos sin ningún filtro.

1. Elimine el contenido del área del editor.

1. Reemplace la consulta de marcador de posición por el siguiente contenido:

    ```
    SELECT * FROM products p WHERE p.price > 500
    ```

    > &#128221; Esta consulta seleccionará todos los elementos en los que el **precio** sea mayor que 500 USD.

1. Seleccione **Ejecutar consulta**.

1. Observe los resultados de la consulta, que deben incluir un único elemento JSON y todas sus propiedades.

1. En la pestaña **Consulta**, seleccione **Estadísticas de consulta**.

1. Aún en la pestaña **Consulta**, observe el valor del campo **Cargo de solicitud** en la sección **Estadísticas de consulta**.

    > &#128221; Normalmente, el cargo de solicitud de esta consulta simple está entre 2 y 3 RU/s cuando el tamaño del contenedor es pequeño.

1. Cierre la ventana o pestaña del explorador web.
