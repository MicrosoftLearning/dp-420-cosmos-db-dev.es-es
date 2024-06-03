---
lab:
  title: Ajuste del rendimiento aprovisionado mediante un script de la CLI de Azure
  module: Module 12 - Manage an Azure Cosmos DB for NoSQL solution using DevOps practices
---

# Ajuste del rendimiento aprovisionado mediante un script de la CLI de Azure

La CLI de Azure es un conjunto de comandos que puede usar para administrar varios recursos en Azure. Azure Cosmos DB tiene un grupo de comandos enriquecido que se puede usar para administrar varias facetas de una cuenta de Azure Cosmos DB independientemente de la API seleccionada.

En este laboratorio, creará una cuenta, una base de datos y un contenedor de Azure Cosmos DB mediante la CLI de Azure. A continuación, realizará ajustes en el rendimiento aprovisionado mediante la CLI de Azure.

## Inicio de sesión en la CLI de Azure

Antes de usar la CLI de Azure, primero debe comprobar la versión de la CLI e iniciar sesión con sus credenciales de Azure.

1. Inicie **Visual Studio Code**.

1. Abra el menú **Terminal** y seleccione **Nuevo terminal** para abrir una nueva instancia de terminal.

1. Vea la versión de la CLI de Azure mediante el siguiente comando:

    ```
    az --version
    ```

1. Vea los grupos de comandos de la CLI de Azure más comunes mediante el siguiente comando:

    ```
    az --help
    ```

1. Instale los certificados tls/ssl antes de iniciar sesión en Azure:

    ```
    CD "C:\Program Files (x86)\Microsoft SDKs\Azure\CLI2\"
    .\python.exe -m pip install pip-system-certs
    ```

1. Comience el procedimiento de inicio de sesión interactivo para la CLI de Azure mediante el siguiente comando:

    ```
    az login
    ```

1. La CLI de Azure abrirá automáticamente una ventana o pestaña del explorador web. Dentro de la instancia del explorador, inicie sesión en la CLI de Azure con las credenciales de Microsoft asociadas a la suscripción.

1. Cierre la ventana o pestaña del explorador web.

1. Compruebe si el proveedor de laboratorio ha creado un grupo de recursos para usted; si es así, registre su nombre, ya que lo necesitará en la sección siguiente.

    ```
    az group list --query "[].{ResourceGroupName:name}" -o table
    ```
    
    Este comando puede devolver varios nombres de grupo de recursos.

1. (Opcional) ***Si no se creó ningún grupo de recursos para usted***, elija un nombre de grupo de recursos y créelo. *Tenga en cuenta que es posible que algunos entornos de laboratorio estén bloqueados y necesitará que un administrador cree el grupo de recursos automáticamente.*

    i. Obtener el nombre de la ubicación más cercana de esta lista

    ```
    az account list-locations --query "sort_by([].{YOURLOCATION:name, DisplayName:regionalDisplayName}, &YOURLOCATION)" --output table
    ```

    ii. Cree el grupo de recursos.  *Tenga en cuenta que es posible que algunos entornos de laboratorio estén bloqueados y necesitará que un administrador cree el grupo de recursos automáticamente.*
    ```
    az group create --name YOURRESOURCEGROUPNAME --location YOURLOCATION
    ```

## Creación de una cuenta de Azure Cosmos DB mediante la CLI de Azure

El grupo de comandos **cosmosdb** contiene comandos básicos para crear y administrar cuentas de Azure Cosmos DB mediante la CLI. Dado que una cuenta de Azure Cosmos DB tiene un URI direccionable, es importante crear un nombre único global para la nueva cuenta, incluso si lo crea a través del script.

1. Vuelva a la instancia de terminal que ya está abierta en **Visual Studio Code**.

1. Consulte la mayoría de los comandos de la CLI de Azure relacionados con **Azure Cosmos DB** mediante el comando siguiente:

    ```
    az cosmosdb --help
    ```

1. Cree una variable denominada **sufijo** con el cmdlet [Get-Random][docs.microsoft.com/powershell/module/microsoft.powershell.utility/get-random] de PowerShell mediante el siguiente comando:

    ```
    $suffix=Get-Random -Maximum 1000000
    ```

    > &#128221; El cmdlet Get-Random genera un entero aleatorio entre 0 y 1000 000. Esto es útil porque nuestros servicios requieren un nombre único global.

1. Cree otro nombre de variable nuevo **accountName** con la cadena codificada de forma rígida **csms** y la sustitución de variables para insertar el valor de la variable **$suffix** mediante el siguiente comando:

    ```
    $accountName="csms$suffix"
    ```

1. Cree otro nombre de variable **resourceGroup** con el nombre del grupo de recursos que creó o visualizó anteriormente en este laboratorio mediante el siguiente comando:

    ```
    $resourceGroup="<resource-group-name>"
    ```

    > &#128221; Por ejemplo, si el grupo de recursos se denomina **dp420**, el comando será **$resourceGroup="dp420"**.

1. Use el cmdlet **echo** para escribir el valor de las variables **$accountName** y **$resourceGroup** en la salida del terminal mediante el siguiente comando:

    ```
    echo $accountName
    echo $resourceGroup
    ```

1. Vea las opciones de **az cosmosdb create** con el comando siguiente:

    ```
    az cosmosdb create --help
    ```

1. Cree una cuenta de Azure Cosmos DB con las variables predefinidas y el siguiente comando:

    ```
    az cosmosdb create --name $accountName --resource-group $resourceGroup
    ```

1. Espere a que el comando **crear** finalice la ejecución y vuelva antes de continuar con este laboratorio.

    > &#128161; El comando **crear** puede tardar entre dos y doce minutos en completarse, de media.

## Creación de recursos de Azure Cosmos DB para NoSQL mediante la CLI de Azure

El grupo de comandos **sql de cosmosdb** contiene comandos para administrar recursos específicos de la API NoSQL para Azure Cosmos DB. Siempre puede usar la marca **--help** para revisar las opciones de estos grupos de comandos.

1. Vuelva a la instancia de terminal que ya está abierta en **Visual Studio Code**.

1. Consulte los grupos de comandos de la CLI de Azure más relacionados con **Azure Cosmos DB para NoSQL** mediante el comando siguiente:

    ```
    az cosmosdb sql --help
    ```

1. Consulte los comandos de la CLI de Azure para administrar **bases de datos de Azure Cosmos DB para NoSQL** mediante el comando siguiente:

    ```
    az cosmosdb sql database --help
    ```

1. Cree una base de datos de Azure Cosmos DB con las variables predefinidas, el nombre de la base de datos **cosmosworks**, y el siguiente comando:

    ```
    az cosmosdb sql database create --name "cosmicworks" --account-name $accountName --resource-group $resourceGroup
    ```

1. Espere a que el comando **crear** finalice la ejecución y vuelva antes de continuar con este laboratorio.

1. Consulte los comandos de la CLI de Azure para administrar contenedores de **Azure Cosmos DB para NoSQL** mediante el comando siguiente:

    ```
    az cosmosdb sql container --help
    ```

1. Cree un contenedor de Azure Cosmos DB con las variables predefinidas, el nombre de la base de datos **cosmosworks**, el nombre del contenedor **productos**, y el siguiente comando:

    ```
    az cosmosdb sql container create --name "products" --throughput 400 --partition-key-path "/categoryId" --database-name "cosmicworks" --account-name $accountName --resource-group $resourceGroup
    ```

1. Espere a que el comando **crear** finalice la ejecución y vuelva antes de continuar con este laboratorio.

1. Vaya a Azure Portal (``portal.azure.com``) desde una nueva ventana o pestaña del explorador web.

1. Inicie sesión en el portal con las credenciales de Microsoft asociadas a su suscripción.

1. Seleccione **Grupos de recursos**, seleccione el grupo de recursos que creó o visualizó anteriormente en este laboratorio y, a continuación, seleccione el recurso de **cuenta de Azure Cosmos DB** que creó en este laboratorio con el prefijo **csms**.

1. En el recurso de cuenta de **Azure Cosmos DB**, vaya al panel del **Explorador de datos**.

1. En **Data Explorer**, expanda el nodo de la base de datos **cosmicworks** y, a continuación, observe el nodo contenedor **products** nuevo dentro del árbol de navegación de la **API NoSQL**.

1. Seleccione el nodo contenedor **products** dentro del árbol de navegación de la **API NoSQL** y, a continuación, seleccione **Escala y configuración**.

1. Observe los valores de la pestaña **Escala**. En concreto, observe que la opción **Manual** está seleccionada en la sección **Rendimiento** y que el rendimiento aprovisionado se establece en **400** RU/s.

1. Cierre la ventana o pestaña del explorador web.

## Ajuste del rendimiento de un contenedor existente mediante la CLI de Azure

La CLI de Azure se puede usar para migrar un contenedor entre el aprovisionamiento manual y el escalado automático del rendimiento. Si el contenedor usa el rendimiento de escalabilidad automática, la CLI se puede usar para ajustar dinámicamente el valor máximo permitido de rendimiento.

1. Vuelva a la instancia de terminal que ya está abierta en **Visual Studio Code**.

1. Consulte los comandos de la CLI de Azure para administrar el rendimiento del contenedor **Azure Cosmos DB for NoSQL** mediante el siguiente comando:

    ```
    az cosmosdb sql container throughput --help
    ```

1. Migre los **productos** rendimiento del contenedor desde el aprovisionamiento manual hasta el escalado automático mediante el siguiente comando:

    ```
    az cosmosdb sql container throughput migrate --name "products" --throughput-type autoscale --database-name "cosmicworks" --account-name $accountName --resource-group $resourceGroup
    ```

1. Espere a que el comando **migrar** finalice la ejecución y vuelva antes de continuar con este laboratorio.

1. Consulte los **productos** del contenedor para determinar el valor de rendimiento mínimo posible mediante el siguiente comando:

    ```
    az cosmosdb sql container throughput show --name "products" --query "resource.minimumThroughput" --output "tsv" --database-name "cosmicworks" --account-name $accountName --resource-group $resourceGroup
    ```

1. Actualice el rendimiento máximo de escalado automático de los **productos** del contenedor desde el valor predeterminado actual de **1000** a un nuevo valor de **5000** mediante el comando siguiente:

    ```
    az cosmosdb sql container throughput update --name "products" --max-throughput 5000 --database-name "cosmicworks" --account-name $accountName --resource-group $resourceGroup
    ```

1. Espere a que el comando **actualizar** finalice la ejecución y vuelva antes de continuar con este laboratorio.

1. Cierre **Visual Studio Code**.

1. Vaya a Azure Portal (``portal.azure.com``) desde una nueva ventana o pestaña del explorador web.

1. Inicie sesión en el portal con las credenciales de Microsoft asociadas a su suscripción.

1. Seleccione **Grupos de recursos**, seleccione el grupo de recursos que creó o visualizó anteriormente en este laboratorio y, a continuación, seleccione el recurso de **cuenta de Azure Cosmos DB** que creó en este laboratorio con el prefijo **csms**.

1. En el recurso de cuenta de **Azure Cosmos DB**, vaya al panel del **Explorador de datos**.

1. En **Data Explorer**, expanda el nodo de la base de datos **cosmicworks** y, a continuación, observe el nodo contenedor **products** nuevo dentro del árbol de navegación de la **API NoSQL**.

1. Seleccione el nodo contenedor **products** dentro del árbol de navegación de la **API NoSQL** y, a continuación, seleccione **Escala y configuración**.

1. Observe los valores de la pestaña **Escala**. En concreto, observe que la opción **escalado automático** está seleccionada en la sección **rendimiento** y que el rendimiento aprovisionado se establece en **5000** RU/s.

1. Cierre la ventana o pestaña del explorador web.

[docs.microsoft.com/powershell/module/microsoft.powershell.utility/get-random]: https://docs.microsoft.com/powershell/module/microsoft.powershell.utility/get-random
