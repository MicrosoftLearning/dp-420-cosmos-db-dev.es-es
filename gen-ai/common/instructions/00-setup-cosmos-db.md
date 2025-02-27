---
title: "Configuración de Azure\_Cosmos\_DB"
lab:
  title: "Configuración de Azure\_Cosmos\_DB"
  module: Setup
layout: default
nav_order: 3
parent: Common setup instructions
---

# Configuración de Azure Cosmos DB

En este ejercicio, crearás una cuenta de Azure Cosmos DB for NoSQL que usarás en todos los módulos de laboratorio y concederás a tu identidad de usuario acceso para administrar los datos de la cuenta mediante la asignación del rol **Colaborador de datos integrado de Cosmos DB**. Esto te permitirá usar la autenticación de Azure para acceder a la base de datos desde el código de laboratorio y evitarás tener que almacenar y administrar claves.

## Creación de una cuenta de Azure Cosmos DB for NoSQL

Azure Cosmos DB es un servicio de base de datos NoSQL basado en la nube que admite varias API. Al aprovisionar una cuenta de Azure Cosmos DB por primera vez, seleccionarás cuál de las API quieres que admita la cuenta. Una vez que la cuenta de Azure Cosmos DB for NoSQL se haya terminado de aprovisionar, puedes recuperar el punto de conexión y la clave y usarlos para conectarte a la cuenta de Azure Cosmos DB for NoSQL mediante el SDK de Azure para Python o cualquier otro SDK que prefieras.

1. Ve a Azure Portal (``portal.azure.com``) desde una nueva ventana o pestaña del explorador web.

1. Inicia sesión en el portal con las credenciales de Microsoft asociadas a tu suscripción.

1. Selecciona **+ Crear un recurso**, busca *Cosmos DB* y, a continuación, crea un nuevo recurso de cuenta de **Azure Cosmos DB for NoSQL** con la siguiente configuración, dejando todas las opciones restantes en sus valores predeterminados:

    | **Configuración** | **Valor** |
    | ---: | :--- |
    | **Suscripción** | *Su suscripción a Azure existente* |
    | **Grupo de recursos** | *Selecciona un grupo de recursos ya existente o crea un nuevo* |
    | **Nombre de cuenta** | *Escribe un nombre único global*. |
    | **Ubicación** | *Selecciona cualquier región disponible* |
    | **Modo de capacidad** | *Sin servidor* |
    | **Aplicación de descuento por nivel Gratis** | *No aplicar* |

    > &#128221; Es posible que los entornos de laboratorio tengan restricciones que te impidan crear un nuevo grupo de recursos. Si es así, usa el grupo de recursos existente creado previamente.

1. Espera a que se complete la tarea de implementación antes de continuar con esta tarea.

1. Ve al recurso de cuenta de **Azure Cosmos DB** recién creado y, después, ve al panel **Claves**.

1. Este panel contiene los detalles de conexión y las credenciales necesarias para conectarte a la cuenta desde el SDK. Específicamente:

    1. Copia el campo **URI** y guárdalo en un editor de texto para más adelante. Usarás este valor de **endpoint** más adelante en este ejercicio.

1. Mantén la pestaña del explorador abierta para el siguiente paso.

## Proporcionar a tu identidad de usuario el rol RBAC de colaborador de datos integrado de Cosmos DB

Como tarea final de este ejercicio, concederás acceso a la identidad de usuario de Microsoft Entra ID para administrar los datos de la cuenta de Azure Cosmos DB for NoSQL mediante su asignación al rol RBAC de **colaborador de datos integrado de Cosmos DB**. Esto te permitirá usar la autenticación de Azure para acceder a la base de datos desde el código y evitarás tener que almacenar y administrar claves.

> &#128221; El uso del control de acceso basado en rol (RBAC) de Microsoft Entra ID para autenticarte en servicios de Azure, como Azure Cosmos DB, presenta varias ventajas principales sobre los métodos basados en claves. RBAC de Entra ID mejora la seguridad a través de controles de acceso precisos adaptados a los roles de usuario, lo que reduce eficazmente los riesgos de acceso no autorizado. También simplifica la administración de usuarios, lo que permite a los administradores asignar y modificar dinámicamente permisos sin la molestia de distribuir y mantener claves criptográficas. Además, este enfoque mejora el cumplimiento y la auditoría al alinearse con las directivas de la organización y facilitar la supervisión y revisión integrales del acceso. Al optimizar la administración de acceso seguro, RBAC de Entra ID crea una solución más eficaz y escalable para aprovechar los servicios de Azure.

1. En la barra de herramientas en [Azure Portal](https://portal.azure.com), abre Cloud Shell.

    ![El icono de Cloud Shell se destaca en la barra de herramientas de Azure Portal.](media/azure-portal-toolbar-cloud-shell.png)

1. En el símbolo del sistema de Cloud Shell, asegúrate de que la suscripción para el ejercicio se usa para los comandos posteriores mediante la ejecución de `az account set -s <SUBSCRIPTION_ID>`, reemplazando el token de marcador de posición `<SUBSCRIPTION_ID>` por el identificador de la suscripción que usas para este ejercicio.

1. Copia la salida del comando anterior para su uso como token `<PRINCIPAL_OBJECT_ID>` en el comando `az cosmosdb sql role assignment create` siguiente.

1. A continuación, recuperarás el identificador de definición del rol **Colaborador de datos integrado de Cosmos DB**. Ejecuta el comando siguiente, asegurándote de reemplazar los tokens `<RESOURCE_GROUP_NAME>` y `<COSMOS_DB_ACCOUNT_NAME>`.

    ```bash
    az cosmosdb sql role definition list --resource-group "<RESOURCE_GROUP_NAME>" --account-name "<COSMOS_DB_ACCOUNT_NAME>"
    ```

    Revisa la salida y busca la definición de roles denominada **Colaborador de datos integrado en Cosmos DB**. La salida contiene el identificador único de la definición de roles en la propiedad `name`. Registra este valor, ya que es necesario usarlo en el paso de asignación más adelante, en el paso siguiente.

1. Ya estás listo para asignarte a la definición del rol **Colaborador de datos integrado de Cosmos DB**. Escribe el siguiente comando en el símbolo del sistema, asegurándote de reemplazar los tokens `<RESOURCE_GROUP_NAME>` y `<COSMOS_DB_ACCOUNT_NAME>`.

    > &#128221; En el comando siguiente, `role-definition-id` se establece en `00000000-0000-0000-0000-000000000002`, que es el valor predeterminado para la definición del rol **Colaborador de datos integrado de Cosmos DB**. Si el valor que recuperaste del comando `az cosmosdb sql role definition list` difiere, reemplaza el valor del comando siguiente antes de la ejecución. El comando `az ad signed-in-user show` recupera el id. de objeto del usuario de Entra ID conectado.

    ```bash
    az cosmosdb sql role assignment create --resource-group "<RESOURCE_GROUP_NAME>" --account-name "<COSMOS_DB_ACCOUNT_NAME>" --role-definition-id "00000000-0000-0000-0000-000000000002" --principal-id $(az ad signed-in-user show --query id -o tsv) --scope "/"
    ```

1. Cuando el comando termine de ejecutarse, podrás ejecutar código localmente para insertar la interacción con los datos almacenados en la base de datos de NoSQL de Cosmos DB.

1. Cierra Cloud Shell.
