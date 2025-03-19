---
lab:
  title: "07.1: Habilitación del vector de búsqueda para Azure Cosmos\_DB for NoSQL"
  module: Build copilots with Python and Azure Cosmos DB for NoSQL
---

# Habilitación del vector de búsqueda de Azure Cosmos DB for NoSQL

Azure Cosmos DB for NoSQL proporciona una eficaz funcionalidad de indexación y búsqueda de vectores diseñadas para almacenar y consultar vectores de alta dimensión de forma eficiente y precisa a cualquier escala. Para aprovechar esta funcionalidad, debes permitir que tu cuenta use la característica *Vector de búsqueda para la API de NoSQL*.

En este laboratorio, crearás una cuenta de Azure Cosmos DB for NoSQL y habilitarás la característica de vector de búsqueda en ella para preparar una base de datos para su uso como almacén de vectores.

## Preparación del entorno de desarrollo

Si aún no has clonado el repositorio de código del laboratorio de **Compilación de copilotos con Azure Cosmos DB** y configurado el entorno local, consulta las instrucciones de [Configuración del entorno de laboratorio local](00-setup-lab-environment.md) para hacerlo.

## Creación de una cuenta de Azure Cosmos DB for NoSQL

Si ya has creado una cuenta de Azure Cosmos DB for NoSQL para los laboratorios de **Compilación de copilotos de Azure Cosmos DB** en este sitio, puedes usarla para este laboratorio y pasar a la [sección siguiente](#enable-vector-search-for-nosql-api). De lo contrario, consulta las instrucciones de [Configuración de Azure Cosmos DB](../../common/instructions/00-setup-cosmos-db.md) para crear una cuenta de Azure Cosmos DB for NoSQL que usarás en todos los módulos de laboratorio y concede a tu identidad de usuario acceso para administrar los datos de la cuenta mediante la asignación al rol **Colaborador de datos integrado de Cosmos DB**.

## Habilitación del vector de búsqueda para la API de NoSQL

En esta tarea, habilitarás la característica *Vector de búsqueda para la API de NoSQL* en la cuenta de Azure Cosmos DB mediante la CLI de Azure.

1. En la barra de herramientas en [Azure Portal](https://portal.azure.com), abre Cloud Shell.

    ![El icono de Cloud Shell se destaca en la barra de herramientas de Azure Portal.](media/07-azure-portal-toolbar-cloud-shell.png)

2. En el símbolo del sistema de Cloud Shell, asegúrate de que la suscripción para el ejercicio se usa para los comandos posteriores mediante la ejecución de `az account set -s <SUBSCRIPTION_ID>`, reemplazando el token de marcador de posición `<SUBSCRIPTION_ID>` por el identificador de la suscripción que usas para este ejercicio.

3. Habilita la característica *Vector de búsqueda para la API de NoSQL* ejecutando el siguiente comando desde Azure Cloud Shell, reemplazando los tokens `<RESOURCE_GROUP_NAME>` y `<COSMOS_DB_ACCOUNT_NAME>` por el nombre del grupo de recursos y el nombre de la cuenta de Azure Cosmos DB, respectivamente.

     ```bash
     az cosmosdb update \
       --resource-group <RESOURCE_GROUP_NAME> \
       --name <COSMOS_DB_ACCOUNT_NAME> \
       --capabilities EnableNoSQLVectorSearch
     ```

4. Espera a que el comando se ejecute correctamente antes de salir de Cloud Shell.

5. Cierra Cloud Shell.

## Creación de una base de datos y un contenedor para hospedar vectores

1. Selecciona **Data Explorer** en el menú izquierdo de la cuenta de Azure Cosmos DB en [Azure Portal](https://portal.azure.com) y, a continuación, selecciona **Nuevo contenedor**.

2. En el diálogo **Nuevo contenedor**:
   1. En **Id. de base de datos**, selecciona **Crear nuevo** y escribe "CosmicWorks" en el campo Id. de base de datos.
   2. En el cuadro **Id. de contenedor**, escribe el nombre "Products".
   3. Asigna "/category_id" como **clave de partición**.

      ![Captura de pantalla de la configuración del nuevo contenedor especificada anteriormente en el diálogo.](media/07-azure-cosmos-db-new-container.png)

   4. Desplázate hasta la parte inferior del cuadro de diálogo **Nuevo contenedor**, expande **Directiva de vectores de contenedor** y selecciona **Agregar inserción de vectores**.

   5. En la sección de configuración de **Directiva de vectores de contenedor**, establece lo siguiente:

      | Configuración | Valor |
      | ------- | ----- |
      | **Path** | Escribe */embedding*. |
      | **Tipo de datos** | Selecciona *float32*. |
      | **Función de distancia** | Selecciona *coseno*. |
      | **Dimensiones** | Escribe *1536* para que coincida con el número de dimensiones producidas por el modelo `text-embedding-3-small` de OpenAI. |
      | **Tipo de índice** | Selecciona *diskANN*. |
      | **Tamaño de bytes de cuantificación** | Déjalo en blanco. |
      | **Tamaño de indexación de la lista de búsqueda** | Acepta el valor predeterminado *100*. |

      ![Captura de pantalla de la Directiva de vectores de contenedor especificada anteriormente en el diálogo Nuevo contenedor.](media/07-azure-cosmos-db-container-vector-policy.png)

   6. Selecciona **Aceptar** para crear la base de datos y el contenedor.

   7. Espera a que se cree el contenedor antes de continuar. El contenedor puede tardar varios minutos en estar listo.
