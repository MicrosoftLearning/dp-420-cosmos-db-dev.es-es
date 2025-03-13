---
lab:
  title: "Creación de un contenedor de Azure Cosmos\_DB for NoSQL mediante plantillas de Azure Resource Manager"
  module: Module 12 - Manage an Azure Cosmos DB for NoSQL solution using DevOps practices
---

# Creación de un contenedor de Azure Cosmos DB for NoSQL mediante plantillas de Azure Resource Manager

Las plantillas de Azure Resource Manager son archivos JSON que definen de forma declarativa la infraestructura que quiere desplegar en Azure. Las plantillas de Azure Resource Manager son una solución común de infraestructura como código para implementar servicios en Azure. Bicep lleva el concepto un poco más lejos y define un lenguaje específico del dominio más fácil de leer que se puede usar para crear plantillas JSON.

En este laboratorio, creará una cuenta, una base de datos y un contenedor de Azure Cosmos DB mediante una plantilla de Azure Resource Manager. Primero creará la plantilla a partir de JSON sin formato y, a continuación, creará la plantilla mediante el lenguaje específico del dominio Bicep.

## Preparación del entorno de desarrollo

Si aún no ha clonado el repositorio de código de laboratorio para **DP-420** al entorno en el que está trabajando en este laboratorio, siga estos pasos para hacerlo. De lo contrario, abra la carpeta clonada anteriormente en **Visual Studio Code**.

1. Inicie **Visual Studio Code**.

    > &#128221; Si aún no está familiarizado con la interfaz de Visual Studio Code, revise la [Guía de introducción para Visual Studio Code][code.visualstudio.com/docs/getstarted]

1. Abra la paleta de comandos y ejecute **Git: Clonar** para clonar el repositorio de GitHub ``https://github.com/microsoftlearning/dp-420-cosmos-db-dev`` en una carpeta local de su elección.

    > &#128161; Puede usar el método abreviado de teclado **CTRL+MAYÚS+P** para abrir la paleta de comandos.

1. Una vez clonado el repositorio, abra la carpeta local que seleccionó en **Visual Studio Code**.

## Crear recursos de Azure Cosmos DB for NoSQL con plantillas de Azure Resource Manager

El proveedor de recursos **Microsoft.DocumentDB** en Azure Resource Manager permite implementar cuentas, bases de datos y contenedores mediante archivos JSON. Aunque los archivos pueden ser complejos, siguen un formato predecible y se pueden escribir con la ayuda de una extensión de Visual Studio Code.

> &#128161; Si está bloqueado y no puede identificar un error de sintaxis con la plantilla, use esta [plantilla de Azure Resource Manager de solución][github.com/arm-template-guide] como guía.

1. En **Visual Studio Code**, en el panel **Explorador**, vaya a la carpeta **31-create-container-arm-template**.

1. Abra el archivo **deploy.json**.

1. Observe la plantilla de Azure Resource Manager vacía:

    ```
    {
        "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "resources": [
        ]
    }
    ```

1. Dentro de la matriz de **recursos**, agregue un nuevo objeto JSON para crear una nueva cuenta de Azure Cosmos DB:

    ```
    {
        "type": "Microsoft.DocumentDB/databaseAccounts",
        "apiVersion": "2021-05-15",
        "name": "[concat('csmsarm', uniqueString(resourceGroup().id))]",
        "location": "[resourceGroup().location]",
        "properties": {
            "databaseAccountOfferType": "Standard",
            "locations": [
                {
                    "locationName": "westus"
                }
            ]
        }
    }
    ```

    El objeto está configurado con los siguientes valores:

    | **Configuración** | **Valor** |
    | ---: | :--- |
    | **Tipo de recurso** | *Microsoft.DocumentDB/databaseAccounts* |
    | **Versión de API** | *2021-05-15* |
    | **Account name** | *csmsarm* &amp; *cadena única generada a partir del nombre de cuenta*  |
    | **Ubicación** | *Ubicación actual del grupo de recursos* |
    | **Tipo de oferta de cuenta** | *Estándar* |
    | **Ubicaciones** | *Solo Oeste de EE. UU.* |

1. Guarde el archivo **deploy.json**.

1. Abra el menú contextual de la carpeta **31-create-container-arm-template** y, a continuación, seleccione **Abrir en terminal integrado** para abrir una nueva instancia de terminal.

    > &#128221; Este comando abrirá el terminal con el directorio inicial ya establecido en la carpeta **31-create-container-arm-template**.

1. Instale los certificados tls/ssl antes de iniciar sesión en Azure:

    ```
    $CurrentDirectory=$pwd
    CD "C:\Program Files\Microsoft SDKs\Azure\CLI2\"
    .\python.exe -m pip install pip-system-certs
    CD $CurrentDirectory
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

    i. Obtén el nombre de la ubicación más cercana de esta lista

    ```
    az account list-locations --query "sort_by([].{YOURLOCATION:name, DisplayName:regionalDisplayName}, &YOURLOCATION)" --output table
    ```

    ii. Cree el grupo de recursos.  *Tenga en cuenta que es posible que algunos entornos de laboratorio estén bloqueados y necesitará que un administrador cree el grupo de recursos automáticamente.*
    ```
    az group create --name YOURRESOURCEGROUPNAME --location YOURLOCATION
    ```

1. Cree un nuevo nombre de variable **resourceGroup** con el nombre del grupo de recursos que creó o visualizó anteriormente en este laboratorio mediante el siguiente comando:

    ```
    $resourceGroup="<resource-group-name>"
    ```

    > &#128221; Por ejemplo, si el grupo de recursos se denomina **dp420**, el comando será **$resourceGroup="dp420"**.

1. Use el cmdlet **echo** para escribir el valor de la variable **$resourceGroup** en la salida del terminal mediante el siguiente comando:

    ```
    echo $resourceGroup
    ```

1. Implemente la plantilla de Azure Resource Manager mediante el comando [az deployment group create][docs.microsoft.com/cli/azure/deployment/group]:

    ```
    az deployment group create --name "arm-deploy-account" --resource-group $resourceGroup --template-file .\deploy.json
    ```

1. Deje abierto el terminal integrado y vuelva al editor para el archivo **deploy.json**.

1. Dentro de la matriz de **recursos**, agregue otro nuevo objeto JSON para crear una nueva base de datos de Azure Cosmos DB for NoSQL:

    ```
    ,
    {
        "type": "Microsoft.DocumentDB/databaseAccounts/sqlDatabases",
        "apiVersion": "2021-05-15",
        "name": "[concat('csmsarm', uniqueString(resourceGroup().id), '/cosmicworks')]",
        "dependsOn": [
            "[resourceId('Microsoft.DocumentDB/databaseAccounts', concat('csmsarm', uniqueString(resourceGroup().id)))]"
        ],
        "properties": {
            "resource": {
                "id": "cosmicworks"
            }
        }
    }
    ```

    El objeto está configurado con los siguientes valores:

    | **Configuración** | **Valor** |
    | ---: | :--- |
    | **Tipo de recurso** | *Microsoft.DocumentDB/databaseAccounts/sqlDatabases* |
    | **Versión de API** | *2021-05-15* |
    | **Account name** | *csmsarm* &amp; *cadena única generada a partir del nombre de cuenta* &amp; */cosmicworks*  |
    | **Id. del recurso** | *cosmicworks* |
    | **Dependencias** | *databaseAccount creado anteriormente en la plantilla* |

1. Guarde el archivo **deploy.json**.

1. Vuelva al terminal integrado.

1. Implemente la plantilla de Azure Resource Manager mediante el comando **az deployment group create**:

    ```
    az deployment group create --name "arm-deploy-database" --resource-group $resourceGroup --template-file .\deploy.json
    ```

1. Deje abierto el terminal integrado y vuelva al editor para el archivo **deploy.json**.

1. Dentro de la matriz de **recursos**, agregue otro nuevo objeto JSON para crear un nuevo contenedor de Azure Cosmos DB for NoSQL:

    ```
    ,
    {
        "type": "Microsoft.DocumentDB/databaseAccounts/sqlDatabases/containers",
        "apiVersion": "2021-05-15",
        "name": "[concat('csmsarm', uniqueString(resourceGroup().id), '/cosmicworks/products')]",
        "dependsOn": [
            "[resourceId('Microsoft.DocumentDB/databaseAccounts', concat('csmsarm', uniqueString(resourceGroup().id)))]",
            "[resourceId('Microsoft.DocumentDB/databaseAccounts/sqlDatabases', concat('csmsarm', uniqueString(resourceGroup().id)), 'cosmicworks')]"
        ],
        "properties": {
            "options": {
                "throughput": 400
            },
            "resource": {
                "id": "products",
                "partitionKey": {
                    "paths": [
                        "/categoryId"
                    ]
                }
            }
        }
    }
    ```

    El objeto está configurado con los siguientes valores:

    | **Configuración** | **Valor** |
    | ---: | :--- |
    | **Tipo de recurso** | *Microsoft.DocumentDB/databaseAccounts/sqlDatabases/containers* |
    | **Versión de API** | *2021-05-15* |
    | **Account name** | *csmsarm* &amp; *cadena única generada a partir del nombre de cuenta* &amp; */cosmicworks/products*  |
    | **Id. del recurso** | *products* |
    | **Rendimiento** | *400* |
    | **Clave de partición** | */categoryId* |
    | **Dependencias** | *Cuenta y base de datos creadas anteriormente en la plantilla* |

1. Guarde el archivo **deploy.json**.

1. Vuelva al terminal integrado.

1. Implemente la plantilla final de Azure Resource Manager mediante el comando **az deployment group create**:

    ```
    az deployment group create --name "arm-deploy-container" --resource-group $resourceGroup --template-file .\deploy.json
    ```

1. Cierre el terminal integrado.

## Observar los recursos de Azure Cosmos DB implementados

Una vez implementados los recursos de Azure Cosmos DB for NoSQL, puede ir a los recursos de Azure Portal. Con el Explorador de datos, validará que la cuenta, la base de datos y el contenedor se implementaron y configuraron correctamente.

1. Vaya a Azure Portal (``portal.azure.com``) desde una nueva ventana o pestaña del explorador web.

1. Inicie sesión en el portal con las credenciales de Microsoft asociadas a su suscripción.

1. Seleccione **Grupos de recursos**, seleccione el grupo de recursos que creó o visualizó anteriormente en este laboratorio y, a continuación, seleccione el recurso de **cuenta de Azure Cosmos DB** que creó en este laboratorio con el prefijo **csmsarm**.

1. En el recurso de cuenta de **Azure Cosmos DB**, vaya al panel del **Explorador de datos**.

1. En **Data Explorer**, expanda el nodo de la base de datos **cosmicworks** y, a continuación, observe el nodo contenedor **products** nuevo dentro del árbol de navegación de la **API NoSQL**.

1. Seleccione el nodo contenedor **products** dentro del árbol de navegación de la **API NoSQL** y, a continuación, seleccione **Escala y configuración**.

1. Observe los valores de la sección **Escala**. En concreto, observe que la opción **Manual** está seleccionada en la sección **Rendimiento** y que el rendimiento aprovisionado se establece en **400** RU/s.

1. Observe los valores de la sección **Configuración**. En concreto, observe que el valor de **Clave de partición** está establecido en **/categoryId**.

1. Cierre la ventana o pestaña del explorador web.

## Crear recursos de Azure Cosmos DB for NoSQL con plantillas de Bicep

Bicep es un lenguaje eficaz específico de dominio que simplifica y facilita la implementación de recursos de Azure en comparación con las plantillas de Azure Resource Manager. Implementará el mismo recurso exacto mediante Bicep y un nombre diferente para ilustrar las diferencias\[\].

> &#128161; Si está bloqueado y no puede identificar un error de sintaxis con la plantilla, use esta [plantilla de Bicep de solución][github.com/bicep-template-guide] como guía.

1. En **Visual Studio Code**, en el panel **Explorador**, vaya a la carpeta **31-create-container-arm-template**.

1. Abra el archivo **deploy.bicep** vacío.

1. Dentro del archivo, agregue un nuevo objeto para crear una nueva cuenta de Azure Cosmos DB:

    ```
    param location string = resourceGroup().location
    
    resource Account 'Microsoft.DocumentDB/databaseAccounts@2021-05-15' = {
      name: 'csmsbicep${uniqueString(resourceGroup().id)}'
      location: location
      properties: {
        databaseAccountOfferType: 'Standard'
        locations: [
          { 
            locationName: 'westus' 
          }
        ]
      }
    }
    ```

    El objeto está configurado con los siguientes valores:

    | **Configuración** | **Valor** |
    | ---: | :--- |
    | **Alias** | *Cuenta* |
    | **Nombre** | *csmsarm* &amp; *cadena única generada a partir del nombre de cuenta* |
    | **Tipo de recurso** | *Microsoft.DocumentDB/databaseAccounts/sqlDatabases* |
    | **Versión de API** | *2021-05-15* |
    | **Ubicación** | *Ubicación actual del grupo de recursos* |
    | **Tipo de oferta de cuenta** | *Estándar* |
    | **Ubicaciones** | *Solo Oeste de EE. UU.* |

1. Guarde el archivo **deploy.bicep**.

1. Abra el menú contextual de la carpeta **31-create-container-arm-template** y, a continuación, seleccione **Abrir en terminal integrado** para abrir una nueva instancia de terminal.

1. Cree un nuevo nombre de variable **resourceGroup** con el nombre del grupo de recursos que creó o visualizó anteriormente en este laboratorio mediante el siguiente comando:

    ```
    $resourceGroup="<resource-group-name>"
    ```

    > &#128221; Por ejemplo, si el grupo de recursos se denomina **dp420**, el comando será **$resourceGroup="dp420"**.

1. Implemente la plantilla de Bicep mediante el comando **az deployment group create**:

    ```
    az deployment group create --name "bicep-deploy-account" --resource-group $resourceGroup --template-file .\deploy.bicep
    ```

1. Deje abierto el terminal integrado y vuelva al editor para el archivo **deploy.bicep**.

1. En el archivo, agregue otro nuevo objeto para crear una nueva base de datos de Azure Cosmos DB:

    ```
    resource Database 'Microsoft.DocumentDB/databaseAccounts/sqlDatabases@2021-05-15' = {
      parent: Account
      name: 'cosmicworks'
      properties: {
        resource: {
            id: 'cosmicworks'
        }
      }
    }
    ```

    El objeto está configurado con los siguientes valores:

    | **Configuración** | **Valor** |
    | ---: | :--- |
    | **Parent** | *Cuenta creada anteriormente en la plantilla* |
    | **Alias** | *Base de datos* |
    | **Nombre** | *cosmicworks*  |
    | **Tipo de recurso** | *Microsoft.DocumentDB/databaseAccounts/sqlDatabases* |
    | **Versión de API** | *2021-05-15* |
    | **Id. del recurso** | *cosmicworks* |

1. Guarde el archivo **deploy.bicep**.

1. Vuelva al terminal integrado.

1. Implemente la plantilla de Bicep mediante el comando **az deployment group create**:

    ```
    az deployment group create --name "bicep-deploy-database" --resource-group $resourceGroup --template-file .\deploy.bicep
    ```

1. Deje abierto el terminal integrado y vuelva al editor para el archivo **deploy.bicep**.

1. En el archivo, agregue otro nuevo objeto para crear un nuevo contenedor de Azure Cosmos DB:

    ```
    resource Container 'Microsoft.DocumentDB/databaseAccounts/sqlDatabases/containers@2021-05-15' = {
      parent: Database
      name: 'products'
      properties: {
        options: {
          throughput: 400
        }
        resource: {
          id: 'products'
          partitionKey: {
            paths: [
              '/categoryId'
            ]
          }
        }
      }
    }
    ```

    El objeto está configurado con los siguientes valores:

    | **Configuración** | **Valor** |
    | ---: | :--- |
    | **Parent** | *Base de datos creada anteriormente en la plantilla* |
    | **Alias** | *Contenedor* |
    | **Nombre** | *products*  |
    | **Id. del recurso** | *products* |
    | **Rendimiento** | *400* |
    | **Ruta de acceso de la clave de partición** | */categoryId* |

1. Guarde el archivo **deploy.bicep**.

1. Vuelva al terminal integrado.

1. Implemente la plantilla final de Bicep mediante el comando **az deployment group create**:

    ```
    az deployment group create --name "bicep-deploy-container" --resource-group $resourceGroup --template-file .\deploy.bicep
    ```

1. Cierre el terminal integrado.

1. Cierre **Visual Studio Code**.

## Observar los resultados de la implementación de plantillas de Bicep

Las implementaciones de Bicep se pueden validar con muchas de las mismas técnicas que se usan en las implementaciones de Azure Resource Manager. No solo validará que la cuenta, la base de datos y el contenedor se implementaron correctamente, también verá el historial de implementación en las seis implementaciones.

1. Vaya a Azure Portal (``portal.azure.com``) desde una nueva ventana o pestaña del explorador web.

1. Inicie sesión en el portal con las credenciales de Microsoft asociadas a su suscripción.

1. Seleccione **Grupos de recursos** y, luego, el grupo de recursos que creó o visualizó anteriormente en este laboratorio.

1. En el grupo de recursos, vaya al panel **Implementaciones**.

1. Observe las seis implementaciones de las plantillas de Azure Resource Manager y los archivos de Bicep.

1. Aún dentro del grupo de recursos, vaya al panel **Información general**.

1. Aún dentro del grupo de recursos, seleccione el recurso de **cuenta de Azure Cosmos DB** que creó en este laboratorio con el prefijo **csmsbicep**.

1. En el recurso de cuenta de **Azure Cosmos DB**, vaya al panel del **Explorador de datos**.

1. En **Data Explorer**, expanda el nodo de la base de datos **cosmicworks** y, a continuación, observe el nodo contenedor **products** nuevo dentro del árbol de navegación de la **API NoSQL**.

1. Seleccione el nodo contenedor **products** dentro del árbol de navegación de la **API NoSQL** y, a continuación, seleccione **Escala y configuración**.

1. Observe los valores de la sección **Escala**. En concreto, observe que la opción **Manual** está seleccionada en la sección **Rendimiento** y que el rendimiento aprovisionado se establece en **400** RU/s.

1. Observe los valores de la sección **Configuración**. En concreto, observe que el valor de **Clave de partición** está establecido en **/categoryId**.

1. Cierre la ventana o pestaña del explorador web.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/cli/azure/deployment/group]: https://docs.microsoft.com/cli/azure/deployment/group
[github.com/arm-template-guide]: https://raw.githubusercontent.com/Sidney-Andrews/acdbsad/solutions/31-create-container-arm-template/deploy.json
[github.com/bicep-template-guide]: https://raw.githubusercontent.com/Sidney-Andrews/acdbsad/solutions/31-create-container-arm-template/deploy.bicep
